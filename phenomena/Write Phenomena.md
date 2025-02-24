Write Phenomena:

1. Lost Update
- Two transactions read and update the same data, causing one update to be lost
- Example:
  * Transaction 1 reads stock count = 100
  * Transaction 2 reads stock count = 100
  * Transaction 1 updates stock to 99
  * Transaction 2 updates stock to 99
  * Final result should be 98, but it's 99

2. Write Skew
- Two transactions read overlapping data sets and make disjoint updates
- Example:
  * Hospital rule: At least one doctor must be on call
  * Two doctors are on call
  * Transaction 1 reads that two doctors are on call, decides to take Doctor 1 off
  * Transaction 2 reads that two doctors are on call, decides to take Doctor 2 off
  * Both commit, leaving no doctors on call

3. Read Skew (Inconsistent Read)
- A transaction reads related data that's being modified by another transaction
- Example:
  * Account A has $100, Account B has $100
  * Transaction 1 transfers $50 from A to B
  * Transaction 2 reads A ($50), then B ($100)
  * Transaction 2 sees total $150 instead of correct $200
