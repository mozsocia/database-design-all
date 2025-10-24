### 🛑 How to setup two workers and a coordinator for postgresql in docker


Below is a **complete, ready-to-run example** that starts **one coordinator** (the “frontend” that accepts client connections) and **two Citus workers** inside Docker Compose.  
It uses the official `citus` image (PostgreSQL 15 + Citus 12 at the time of writing) and configures everything with environment variables and a tiny entry-point script.

---

## 1. Directory layout

```
citus-cluster/
├─ docker-compose.yml
└─ init/
   └─ 00-setup-citus.sh
```

---

## 2. `docker-compose.yml`

```yaml
version: "3.9"

services:
  coordinator:
    image: citusdata/citus:15
    container_name: citus_coordinator
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_USER: citus
      PGUSER: citus
    volumes:
      - coordinator_data:/var/lib/postgresql/data
    command: >
      -c citus.shard_replication_factor=1
      -c citus.max_prepared_transactions=200
    depends_on:
      - worker1
      - worker2
    entrypoint: ["/bin/bash", "-c"]
    command: |
      until pg_isready -h worker1 -p 5432 && pg_isready -h worker2 -p 5432; do
        echo "Waiting for workers..."; sleep 2;
      done;
      docker-entrypoint.sh postgres

  worker1:
    image: citusdata/citus:15
    container_name: citus_worker1
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_USER: citus
    volumes:
      - worker1_data:/var/lib/postgresql/data
    command: >
      -c citus.shard_replication_factor=1
      -c citus.enable_manual_changes=true

  worker2:
    image: citusdata/citus:15
    container_name: citus_worker2
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_USER: citus
    volumes:
      - worker2_data:/var/lib/postgresql/data
    command: >
      -c citus.shard_replication_factor=1
      -c citus.enable_manual_changes=true

volumes:
  coordinator_data:
  worker1_data:
  worker2_data:
```

**What it does**

| Service       | Role                | Important flags |
|---------------|---------------------|-----------------|
| `coordinator` | Accepts client connections, runs `citus` extension | waits for workers, then starts PG |
| `worker1/2`   | Store shards        | `enable_manual_changes` allows `add_node` |

---

## 3. `init/00-setup-citus.sh`

```bash
#!/bin/bash
set -e

# Wait for PostgreSQL to be ready
until pg_isready -q; do sleep 2; done

# Create the citus extension (only on coordinator)
psql -v ON_ERROR_STOP=1 -U "$POSTGRES_USER" -d postgres <<-EOSQL
  CREATE EXTENSION IF NOT EXISTS citus;
EOSQL

# Register the two workers
psql -v ON_ERROR_STOP=1 -U "$POSTGRES_USER" -d postgres <<-EOSQL
  SELECT citus_add_node('worker1', 5432);
  SELECT citus_add_node('worker2', 5432);
EOSQL

echo "Citus cluster ready – coordinator + 2 workers"
```

Make it executable:

```bash
chmod +x init/00-setup-citus.sh
```

---

## 4. Wire the script into the coordinator

Add a bind-mount in `docker-compose.yml` under the **coordinator** service:

```yaml
    volumes:
      - coordinator_data:/var/lib/postgresql/data
      - ./init/00-setup-citus.sh:/docker-entrypoint-initdb.d/00-setup-citus.sh
```

Docker will run any `*.sh` file in `/docker-entrypoint-initdb.d/` **once** when the data directory is empty (i.e. on first start).

---

## 5. Start the cluster

```bash
docker compose up -d
```

Wait a few seconds for the init script to finish. You can watch logs:

```bash
docker compose logs -f coordinator
```

You should see:

```
Citus cluster ready – coordinator + 2 workers
```

---

## 6. Verify

```bash
# Connect to the coordinator
docker exec -it citus_coordinator psql -U citus

citus=# SELECT * FROM citus_get_active_worker_nodes();
 nodeid | nodename | nodeport | groupid
--------+----------+----------+---------
      1 | worker1  |     5432 |       1
      2 | worker2  |     5432 |       2
(2 rows)
```

Create a distributed table:

```sql
CREATE TABLE test_dist(id serial primary key, val int);
SELECT create_distributed_table('test_dist', 'id');
```

Insert data – it will be sharded automatically across the two workers.

---

## 7. Quick cheat-sheet

| Command | Purpose |
|---------|---------|
| `docker compose up -d` | Start all containers |
| `docker compose down -v` | **Stop & delete volumes** (fresh start) |
| `docker exec -it citus_coordinator psql -U citus` | Shell into coordinator |
| `SELECT citus_add_node('worker3',5432);` | Add another worker later |
| `SELECT citus_remove_node('worker2',5432);` | Remove a worker |

---

## 8. Production notes (optional but recommended)

1. **Persistent passwords** – use Docker secrets or a `.env` file.  
2. **Replication factor** – set `citus.shard_replication_factor=2` on coordinator **after** you have ≥2 workers per shard.  
3. **Superuser** – avoid `POSTGRES_USER=citus` in prod; create a dedicated `citus_admin`.  
4. **Network** – put the services on a custom bridge network for isolation.  
5. **Backups** – `pg_dumpall -U citus -h <coordinator>` or use `wal-e`/`barman`.

---

That’s it! You now have a **fully functional Citus cluster** (1 coordinator + 2 workers) running in Docker with a single `docker compose up`.