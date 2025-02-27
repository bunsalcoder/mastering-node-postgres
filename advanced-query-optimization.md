# Advanced Query Optimization in PostgreSQL with Node.js

## 1. PostgreSQL Indexing Strategies
Indexes help speed up queries by allowing the database to quickly find rows. PostgreSQL supports multiple types of indexes:

### 1.1 B-Tree Index (Default)
- Best for **equality (`=`) and range queries (`<`, `>`, `BETWEEN`)**.
- Automatically created for **primary keys and unique constraints**.

#### Creating a B-Tree Index
```sql
CREATE INDEX idx_users_email ON users (email);
```
**Knex:**
```js
knex.schema.alterTable('users', function (table) {
  table.index('email', 'idx_users_email');
});
```
**Sequelize:**
```js
await queryInterface.addIndex('users', ['email'], {
  name: 'idx_users_email',
  using: 'BTREE',
});
```

### 1.2 Hash Index
- Faster than B-Tree for **exact matches (`=` only)**.
- Not created automatically.

#### Creating a Hash Index
```sql
CREATE INDEX idx_users_email_hash ON users USING hash (email);
```

### 1.3 GIN (Generalized Inverted Index)
- Best for **full-text search** and **JSONB columns**.

#### Indexing a JSONB Column
```sql
CREATE INDEX idx_users_data ON users USING gin (data);
```

### 1.4 BRIN (Block Range Index)
- Best for **very large tables** where data is **naturally sorted**.

#### Creating a BRIN Index
```sql
CREATE INDEX idx_users_created_at ON users USING brin (created_at);
```

## 2. Using `EXPLAIN ANALYZE` to Analyze Query Performance
PostgreSQL provides `EXPLAIN ANALYZE` to **visualize how a query executes**.

#### Basic Example
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```
#### Understanding the Output
```
Index Scan using idx_users_email on users  (cost=0.15..8.27 rows=1 width=64) (actual time=0.045..0.047 rows=1 loops=1)
```
- `Index Scan`: PostgreSQL **used an index** instead of scanning the entire table.
- `actual time=0.045..0.047`: Actual **execution time** (in milliseconds).

## 3. Optimizing Database Connections Using Connection Pooling
Database connections are expensive, and **connection pooling** reduces overhead by reusing connections.

### Setting Up Connection Pooling in Knex
```js
const knex = require('knex')({
  client: 'pg',
  connection: {
    host: 'localhost',
    user: 'postgres',
    password: 'password',
    database: 'mydb',
  },
  pool: { min: 2, max: 10 },
});
```

### Setting Up Connection Pooling in Sequelize
```js
const { Sequelize } = require('sequelize');
const sequelize = new Sequelize('mydb', 'postgres', 'password', {
  host: 'localhost',
  dialect: 'postgres',
  pool: {
    max: 10,
    min: 2,
    acquire: 30000,
    idle: 10000,
  },
});
```

## Conclusion
✅ **Use the right index** for the right query (B-Tree, Hash, GIN, BRIN).  
✅ **Analyze queries with `EXPLAIN ANALYZE`** to detect performance issues.  
✅ **Use connection pooling** to optimize database performance.  

---

🔹 Next Step: Apply these concepts in a real-world project and test performance improvements! 🚀
