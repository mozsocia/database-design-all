# How PostgreSQL Scaling with Citus Works for Beginners

At its core, Citus helps you distribute your database across multiple servers instead of trying to run everything on one increasingly powerful machine. Here's what's happening:

## The Basic Concept

Traditionally, PostgreSQL runs on a single server. As your data grows, you'd need to upgrade that server with more CPU, RAM, and storage (vertical scaling), which has limits.

Citus changes this by letting you:
1. Split your database across multiple servers (horizontal scaling)
2. Keep using standard PostgreSQL commands and features

## How It Works

When you use Citus:

1. **Your database cluster has two types of nodes:**
   - A **coordinator node** that users and applications connect to
   - Multiple **worker nodes** that actually store and process the data

2. **Your tables get distributed in one of two ways:**
   - **Distributed tables**: Large tables get split into shards based on a distribution column (like user_id)
   - **Reference tables**: Small lookup tables get copied to all worker nodes

3. **When you run a query:**
   - The coordinator receives it
   - Determines which worker nodes have the relevant data
   - Sends portions of the query to those workers
   - Combines the results and returns them to you

## The Benefits

- **Scalability**: Add more worker nodes as your data grows
- **Performance**: Queries run in parallel across multiple servers
- **Familiarity**: Keep using standard PostgreSQL syntax

## Real-world Example

Imagine a user table with millions of records:
- Without Citus: All records live on one server, queries slow down as data grows
- With Citus: Data is distributed across multiple servers by user_id, queries about specific users go directly to the right server

This approach lets you scale to hundreds of terabytes with much better performance than a single PostgreSQL server could handle.



### 🔴 How to create coordinator node and workers node

# Setting Up Citus Coordinator and Worker Nodes

Here's how to set up a Citus cluster with coordinator and worker nodes:

## 1. Install Prerequisites on All Machines

On all servers (coordinator and workers):

```bash
# Add Citus repository
curl https://install.citusdata.com/community/deb.sh | sudo bash

# Install PostgreSQL and Citus packages
sudo apt-get install postgresql-14-citus-11.0
sudo apt-get install postgresql-14-citus-11.0-postgis  # If you need PostGIS
```

## 2. Initialize PostgreSQL on All Machines

On all servers:

```bash
# Initialize the database
sudo pg_ctlcluster 14 main start
```

## 3. Configure Coordinator Node

On the coordinator server:

```bash
# Edit postgresql.conf
sudo nano /etc/postgresql/14/main/postgresql.conf
```

Add these settings:
```
listen_addresses = '*'  
shared_preload_libraries = 'citus'
```

Now edit pg_hba.conf to allow worker connections:
```bash
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

Add lines for your worker IPs:
```
host    all             all             worker1_ip/32        trust
host    all             all             worker2_ip/32        trust
```

Restart PostgreSQL:
```bash
sudo systemctl restart postgresql
```

## 4. Configure Worker Nodes

On each worker server, make similar configuration changes:

```bash
# Edit postgresql.conf
sudo nano /etc/postgresql/14/main/postgresql.conf
```

Add:
```
listen_addresses = '*'
shared_preload_libraries = 'citus'
```

Edit pg_hba.conf to allow coordinator connection:
```bash
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

Add:
```
host    all             all             coordinator_ip/32    trust
```

Restart PostgreSQL:
```bash
sudo systemctl restart postgresql
```

## 5. Create the Citus Extension

On all nodes (coordinator and workers), connect to PostgreSQL and create the extension:

```bash
sudo -u postgres psql
```

```sql
CREATE EXTENSION citus;
```

## 6. Register Workers with Coordinator

On the coordinator only:

```bash
sudo -u postgres psql
```

```sql
SELECT * FROM citus_add_node('worker1_hostname_or_ip', 5432);
SELECT * FROM citus_add_node('worker2_hostname_or_ip', 5432);
```

## 7. Verify Configuration

On the coordinator:

```sql
SELECT * FROM citus_get_active_worker_nodes();
```

This should list all your worker nodes.

## Production Considerations

For production environments:
- Use proper authentication instead of "trust"
- Configure appropriate memory settings in postgresql.conf
- Set up proper network security between nodes
- Consider automated failover solutions

Now your Citus cluster is ready to start distributing tables and handling queries across multiple nodes!

### 🔴 How to distribute table and reference table please give details as beginner

# Distributing Tables in Citus for Beginners

Once your Citus cluster is set up, you need to tell it how to distribute your data. Here's how to create distributed and reference tables:

## Understanding Distribution Types

1. **Distributed Tables**: Large tables split across worker nodes
2. **Reference Tables**: Small lookup tables replicated to all workers

## Step 1: Connect to Your Database

First, connect to PostgreSQL on your coordinator node:

```bash
sudo -u postgres psql
```

## Step 2: Create Regular PostgreSQL Tables

Start by creating normal PostgreSQL tables:

```sql
-- Example user table
CREATE TABLE users (
    user_id bigserial PRIMARY KEY,
    username text NOT NULL,
    email text NOT NULL,
    created_at timestamp DEFAULT now()
);

-- Example products table (for reference table example)
CREATE TABLE products (
    product_id serial PRIMARY KEY,
    name text NOT NULL,
    price decimal(10,2) NOT NULL
);
```

## Step 3: Distribute Your Large Tables

For large tables that need to be distributed across nodes:

```sql
-- Distribute the users table using user_id as the distribution column
SELECT create_distributed_table('users', 'user_id');
```

What happens:
- Citus creates shards (data chunks) for this table
- Each shard is placed on a worker node
- The data is distributed based on the value in 'user_id' column
- By default, Citus creates 32 shards

The distribution column should be:
- Usually your primary key or a natural partition key
- Used frequently in WHERE clauses
- Ideally evenly distributed (no "hot spots")

## Step 4: Create Reference Tables

For smaller tables that need to be available on all nodes:

```sql
-- Create a reference table for products
SELECT create_reference_table('products');
```

What happens:
- The entire table is copied to every worker node
- Good for lookup tables that all queries might need access to
- Typically used for tables under ~10MB in size

## Step 5: Insert Data Normally

After distribution, you can insert data normally:

```sql
-- Insert into distributed table
INSERT INTO users (username, email) VALUES ('john_doe', 'john@example.com');

-- Insert into reference table
INSERT INTO products (name, price) VALUES ('Basic Widget', 9.99);
```

## Step 6: Verify Your Distribution

Check how your table is distributed:

```sql
-- See shards for your distributed table
SELECT * FROM pg_dist_shard WHERE logicalrelid = 'users'::regclass;

-- See placement of shards on workers
SELECT * FROM pg_dist_placement JOIN pg_dist_shard 
  ON pg_dist_placement.shardid = pg_dist_shard.shardid
  WHERE logicalrelid = 'users'::regclass;
```

## Important Distribution Tips

1. **Co-locate related tables** - Tables that join frequently should use the same distribution column:
   ```sql
   CREATE TABLE orders (
       order_id bigserial PRIMARY KEY,
       user_id bigint NOT NULL,
       order_date timestamp DEFAULT now()
   );
   
   -- Distribute using the same column as the users table
   SELECT create_distributed_table('orders', 'user_id');
   ```

2. **Choose distribution columns wisely**:
   - They should divide data relatively evenly
   - They should be used in most query WHERE clauses
   - They shouldn't change after insertion

3. **For small tables**, always use reference tables so they're available everywhere

This distribution strategy lets your database grow horizontally as your data volume increases!
