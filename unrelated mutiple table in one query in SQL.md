**Q: But I do not want results in one variable, I want different table info in different key, but in one query**

--

If you want to fetch data from different tables in a single query but keep the results separate (e.g., in different keys), you can use a **JSON object** to structure the output. This approach allows you to return the data in a single query while keeping the results for each table distinct.

Here’s how you can achieve this in Node.js:

---

### SQL Query
You can use **subqueries** or **Common Table Expressions (CTEs)** to fetch data from multiple tables and structure the results as a JSON object.

```sql
SELECT
    (SELECT JSON_ARRAYAGG(JSON_OBJECT('id', id, 'name', name)) FROM users) AS users,
    (SELECT JSON_ARRAYAGG(JSON_OBJECT('id', id, 'name', name)) FROM products) AS products;
```

---

### Node.js Code
```javascript
const mysql = require('mysql2/promise');

async function fetchDataSeparateKeys() {
    // Create a connection to the database
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
                (SELECT JSON_ARRAYAGG(JSON_OBJECT('id', id, 'name', name)) FROM users) AS users,
                (SELECT JSON_ARRAYAGG(JSON_OBJECT('id', id, 'name', name)) FROM products) AS products;
        `;

        // Execute the query
        const [results] = await connection.execute(query);

        // Parse the JSON results
        const data = {
            users: JSON.parse(results[0].users),
            products: JSON.parse(results[0].products)
        };

        // Log the results
        console.log('Users:', data.users);
        console.log('Products:', data.products);

        return data;
    } catch (error) {
        console.error('Error executing query:', error);
    } finally {
        // Close the connection
        await connection.end();
    }
}

// Call the function
fetchDataSeparateKeys();
```

---

### Explanation
1. **JSON_ARRAYAGG**: Aggregates rows into a JSON array.
2. **JSON_OBJECT**: Converts each row into a JSON object.
3. **Subqueries**: Each subquery fetches data from a different table and structures it as a JSON array.
4. **Node.js**: The results are parsed into JavaScript objects for easy access.

---

### Output
The result will be an object with separate keys for each table:
```javascript
{
    users: [
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' }
    ],
    products: [
        { id: 1, name: 'Laptop' },
        { id: 2, name: 'Smartphone' }
    ]
}
```

---

### Performance Considerations
1. **Indexes**: Ensure the columns used in the subqueries (e.g., `id`, `name`) are indexed.
2. **Limits**: If the tables are large, consider adding `LIMIT` clauses to the subqueries to avoid fetching too much data at once.
3. **Database Optimization**: Optimize your database configuration for better query performance.

---

### Alternative: Using UNION with a Type Column
If you prefer to avoid JSON functions, you can use a `UNION` query with a `type` column to differentiate between tables, then process the results in Node.js:

```sql
SELECT 'user' AS type, id, name FROM users
UNION ALL
SELECT 'product' AS type, id, name FROM products;
```

In Node.js, you can separate the results into different keys:

```javascript
async function fetchDataWithUnion() {
    const connection = await mysql.createConnection({
        host: 'localhost',
        user: 'root',
        password: 'password',
        database: 'your_database'
    });

    try {
        // Define the SQL query
        const query = `
            SELECT 'user' AS type, id, name FROM users
            UNION ALL
            SELECT 'product' AS type, id, name FROM products;
        `;

        // Execute the query
        const [results] = await connection.execute(query);

        // Separate results into different keys
        const data = {
            users: results.filter(row => row.type === 'user'),
            products: results.filter(row => row.type === 'product')
        };

        // Log the results
        console.log('Users:', data.users);
        console.log('Products:', data.products);

        return data;
    } catch (error) {
        console.error('Error executing query:', error);
    } finally {
        await connection.end();
    }
}

fetchDataWithUnion();
```

This approach avoids JSON functions but requires additional processing in Node.js to separate the results. Choose the method that best fits your use case!

**Q: You have given two option "subqueries or Common Table Expressions (CTEs) " and "Using UNION with a Type Column" which one is more performant and use only one query so use less resources and connection**

--

When comparing **subqueries/CTEs** and **UNION with a Type Column** in terms of performance and resource usage, the **UNION with a Type Column** approach is generally more performant and efficient for the following reasons:

---

### Why **UNION with a Type Column** is More Performant:
1. **Single Pass Over Data**:
   - The `UNION` approach combines the results of two `SELECT` queries into a single result set in one pass. This is often more efficient than running separate subqueries, which may require multiple passes over the data.

2. **Less Overhead**:
   - Subqueries or CTEs can introduce additional overhead, especially if they involve complex aggregations or transformations. `UNION` is a simpler operation and is optimized by most databases.

3. **Index Utilization**:
   - If the columns used in the `UNION` query (e.g., `id`, `name`) are indexed, the database can efficiently retrieve the data. Subqueries may not always leverage indexes as effectively.

4. **Single Query**:
   - Both approaches use a single query, but `UNION` is often more straightforward and easier for the database to optimize.

5. **Scalability**:
   - `UNION` scales better for larger datasets because it avoids the overhead of JSON aggregation and parsing, which can be resource-intensive.

---

### Example: **UNION with a Type Column**
Here’s how you can implement the **UNION with a Type Column** approach in Node.js:

#### SQL Query
```sql
SELECT 'user' AS type, id, name FROM users
UNION ALL
SELECT 'product' AS type, id, name FROM products;
```

#### Node.js Code
```javascript
const mysql = require('mysql2/promise');

async function fetchDataWithUnion() {
    const connection = await mysql.createConnection({
        host: 'localhost',
        user: 'root',
        password: 'password',
        database: 'your_database'
    });

    try {
        // Define the SQL query
        const query = `
            SELECT 'user' AS type, id, name FROM users
            UNION ALL
            SELECT 'product' AS type, id, name FROM products;
        `;

        // Execute the query
        const [results] = await connection.execute(query);

        // Separate results into different keys
        const data = {
            users: results.filter(row => row.type === 'user'),
            products: results.filter(row => row.type === 'product')
        };

        // Log the results
        console.log('Users:', data.users);
        console.log('Products:', data.products);

        return data;
    } catch (error) {
        console.error('Error executing query:', error);
    } finally {
        await connection.end();
    }
}

fetchDataWithUnion();
```

---

### Output
The result will be an object with separate keys for each table:
```javascript
{
    users: [
        { type: 'user', id: 1, name: 'John Doe' },
        { type: 'user', id: 2, name: 'Jane Smith' }
    ],
    products: [
        { type: 'product', id: 1, name: 'Laptop' },
        { type: 'product', id: 2, name: 'Smartphone' }
    ]
}
```

---

### When to Use Subqueries/CTEs:
Subqueries or CTEs are useful when:
1. You need to perform complex transformations or aggregations on the data before combining it.
2. You want to structure the output as JSON directly in the query.
3. The data from each table is independent and doesn’t need to be combined into a single result set.

---

### Conclusion
For your use case, where you want to fetch data from unrelated tables in a single query and keep the results separate, the **UNION with a Type Column** approach is more performant and resource-efficient. It uses a single query, minimizes overhead, and scales better for larger datasets.
