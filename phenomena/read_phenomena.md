I'll explain all types of database phenomena that can occur during concurrent transactions in detail:

Read Phenomena:

### 1. Dirty Read
- A transaction reads data written by a concurrent uncommitted transaction
- Example:
  * Transaction 1 updates a user's balance from $100 to $200
  * Transaction 2 reads the balance as $200 before Transaction 1 commits
  * Transaction 1 rolls back, reverting balance to $100
  * Transaction 2 is now working with invalid data ($200)
 ![dirty_read drawio](https://github.com/user-attachments/assets/a50857a3-d8ad-4a2e-b471-3c0257969424)

  

### 2. Non-Repeatable Read
- A transaction reads the same data twice but gets different values
- Example:
  * Transaction 1 reads a product price as $50
  * Transaction 2 updates the price to $60 and commits
  * Transaction 1 reads the price again and gets $60
  * The same query yielded different results within the same transaction
 
  ![non-repeatable_read drawio](https://github.com/user-attachments/assets/5a0fd506-0ff9-4459-85b3-4a01c01c8980)


### 3. Phantom Read
- A transaction re-executes a query that returns a set of rows and finds that the set has changed
- Example:
  * Transaction 1 reads all products priced over $100 (finds 5 products)
  * Transaction 2 inserts a new product priced at $150 and commits
  * Transaction 1 repeats the query and now finds 6 products

![phantom_read drawio](https://github.com/user-attachments/assets/f00a03f0-c98a-452d-9ce4-55f28ef62a53)















