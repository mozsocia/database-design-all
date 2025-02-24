
Prevention Techniques:

1. Isolation Levels:
- READ UNCOMMITTED: No protection
- READ COMMITTED: Prevents dirty reads
- REPEATABLE READ: Prevents dirty and non-repeatable reads
- SERIALIZABLE: Prevents all phenomena

2. Locking Mechanisms:
- Shared (Read) Locks
- Exclusive (Write) Locks
- Range Locks
- Predicate Locks

3. Multi-Version Concurrency Control (MVCC):
- Maintains multiple versions of data
- Allows readers to see a consistent snapshot
- Used by PostgreSQL, Oracle, and others
