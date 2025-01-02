
**Q: ok now how to get related data from multiple table in one query and i want to like user.order will have all order of that user, Without JSON Functions, i want to get all columns for both table**

--

To fetch related data from multiple tables in one query and structure it such that each user has their associated orders (e.g., `user.orders`), you can use **JOINs** to combine the data and then process the results in Node.js to nest the orders under each user. This approach avoids JSON functions and retrieves all columns from both tables.

---

### Example Scenario
Assume you have two tables:
- `users`: Stores user information.
- `orders`: Stores order information, with a foreign key `user_id` linking to the `users` table.

You want to fetch all users and their associated orders in a nested structure.

---

### SQL Query
Use a **LEFT JOIN** to fetch all users and their orders (if any):

```sql
SELECT 
    users.*, 
    orders.* 
FROM 
    users 
LEFT JOIN 
    orders 
ON 
    users.id = orders.user_id;
```

---

### Node.js Code
Process the query results to nest orders under each user:

```javascript
const mysql = require('mysql2/promise');

async function fetchUsersWithOrders() {
    const connection = await mysql.createConnection({
        host: 'localhost',
        user: 'root',
        password: 'password',
        database: 'your_database'
    });

    try {
        // Define the SQL query
        const query = `
            SELECT 
                users.*, 
                orders.* 
            FROM 
                users 
            LEFT JOIN 
                orders 
            ON 
                users.id = orders.user_id;
        `;

        // Execute the query
        const [results] = await connection.execute(query);

        // Process the results to nest orders under each user
        const usersMap = new Map();

        for (const row of results) {
            const userId = row.id;

            // If the user is not already in the map, add them
            if (!usersMap.has(userId)) {
                usersMap.set(userId, {
                    id: row.id,
                    name: row.name,
                    email: row.email, // Add other user columns as needed
                    orders: [] // Initialize an empty array for orders
                });
            }

            // If the row contains order data, add it to the user's orders
            if (row.user_id) {
                usersMap.get(userId).orders.push({
                    id: row.user_id, // Order ID
                    product: row.product, // Add other order columns as needed
                    amount: row.amount,
                    created_at: row.created_at
                });
            }
        }

        // Convert the map to an array of users
        const users = Array.from(usersMap.values());

        // Log the results
        console.log('Users with Orders:', users);

        return users;
    } catch (error) {
        console.error('Error executing query:', error);
    } finally {
        await connection.end();
    }
}

fetchUsersWithOrders();
```

---

### Explanation
1. **SQL Query**:
   - The `LEFT JOIN` ensures that all users are included, even if they have no orders.
   - All columns from both tables are selected.

2. **Processing in Node.js**:
   - A `Map` is used to group orders by user ID.
   - For each row, the user is added to the map if they don’t already exist.
   - If the row contains order data, the order is added to the user’s `orders` array.

3. **Output**:
   - The result is an array of users, where each user has a nested `orders` array containing their associated orders.

---

### Example Output
```javascript
[
    {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
        orders: [
            {
                id: 101,
                product: 'Laptop',
                amount: 1200,
                created_at: '2023-10-01 12:00:00'
            },
            {
                id: 102,
                product: 'Smartphone',
                amount: 800,
                created_at: '2023-10-02 14:00:00'
            }
        ]
    },
    {
        id: 2,
        name: 'Jane Smith',
        email: 'jane@example.com',
        orders: [] // No orders for this user
    }
]
```

---

### Performance Considerations
1. **Indexes**:
   - Ensure `users.id` and `orders.user_id` are indexed to optimize the `JOIN`.

2. **Limiting Data**:
   - If the dataset is large, consider adding `LIMIT` or pagination to the query.

3. **Database Optimization**:
   - Optimize your database configuration for better query performance.

---

### Why Not Use JSON Functions?
Using JSON functions (e.g., `JSON_OBJECT`, `JSON_ARRAYAGG`) can simplify the query but may introduce overhead, especially for large datasets. The approach above avoids JSON functions and processes the data in Node.js, which is more flexible and performant for this use case.



