# PostgreSQL Partition Types for Beginners

PostgreSQL offers three main partition types to divide large tables into smaller, more manageable pieces. Here's a simple explanation of each:

## 1. Range Partitioning
```sql
CREATE TABLE measurements (
    logdate date not null,
    reading int
) PARTITION BY RANGE (logdate);

CREATE TABLE measurements_2023 PARTITION OF measurements
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
```
**What it does:** Divides data based on a range of values (like dates, IDs, or prices). Each partition contains values within a specific range.

**When to use:** Best for time-series data or sequential values where you want to query specific periods.

## 2. List Partitioning
```sql
CREATE TABLE sales (
    sale_date date,
    region text,
    amount decimal
) PARTITION BY LIST (region);

CREATE TABLE sales_americas PARTITION OF sales
    FOR VALUES IN ('North America', 'South America');
```
**What it does:** Divides data based on specific discrete values in a column.

**When to use:** When your data naturally groups into distinct categories (regions, departments, status types).

## 3. Hash Partitioning
```sql
CREATE TABLE orders (
    order_id int,
    customer_id int,
    amount decimal
) PARTITION BY HASH (customer_id);

CREATE TABLE orders_p0 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```
**What it does:** Distributes data evenly across partitions using a hash function.

**When to use:** When you need balanced partitions but don't have a natural way to divide the data.

Would you like me to explain how these partition types work in more detail?
