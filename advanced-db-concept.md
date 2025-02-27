# Advanced Database Concepts

## 1. Indexing and Query Optimization

### Understanding Indexes
Indexes improve query performance by allowing the database to locate rows faster. PostgreSQL supports various types of indexes:

- **B-tree Index** (default, used for most cases)
- **Hash Index** (for equality comparisons)
- **GIN Index** (for full-text search)
- **BRIN Index** (for large tables with ordered data)

### Creating an Index
```sql
CREATE INDEX idx_users_email ON users(email);
```

### Using `EXPLAIN ANALYZE`
To analyze how queries are executed and optimized:
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```
This helps in identifying slow queries and optimizing indexes.

---

## 2. Normalization vs Denormalization

### Normalization
Normalization organizes data to reduce redundancy and improve consistency. Key forms:

- **1NF (First Normal Form)**: Ensure atomicity of data.
- **2NF (Second Normal Form)**: Remove partial dependencies.
- **3NF (Third Normal Form)**: Remove transitive dependencies.

Example: Instead of storing user addresses inside the `users` table, create a separate `addresses` table linked by `user_id`.

### Denormalization
Denormalization improves performance by reducing joins. It is useful for read-heavy applications.

Example: Storing user’s latest order directly in the `users` table instead of joining with an `orders` table.

---

## 3. Database Design Patterns

### Entity-Attribute-Value (EAV)
Useful when storing dynamic attributes.
```sql
CREATE TABLE product_attributes (
    id SERIAL PRIMARY KEY,
    product_id INT REFERENCES products(id),
    attribute_name TEXT,
    attribute_value TEXT
);
```

### Sharding
Dividing a large dataset into smaller, manageable pieces across different database instances to improve scalability.
Example: Sharding users by region to distribute the load.

---

## 4. Data Integrity

### Foreign Keys
Enforce relationships between tables.
```sql
ALTER TABLE orders ADD CONSTRAINT fk_orders_users FOREIGN KEY (user_id) REFERENCES users(id);
```

### Constraints
Prevent invalid data.
```sql
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE(email);
```

### Triggers
Automatically execute actions before or after a database event.
```sql
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.modified_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER before_user_update
BEFORE UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION update_modified_column();
```

---

These advanced database concepts help in designing efficient, scalable, and high-performing database systems.

Relevant to `advanced-query-optimization.md`

