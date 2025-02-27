# Knex.js - Advanced SQL Query Builder for Node.js

Knex.js is a flexible and powerful SQL query builder for Node.js that works with several SQL databases, including PostgreSQL. This README is designed to help you master Knex.js from intermediate to advanced levels, providing clear explanations and examples for key concepts such as migrations, advanced queries, transactions, performance optimizations, and more.

## Table of Contents

1. [Installation and Setup](#installation-and-setup)
2. [Basic CRUD Operations](#basic-crud-operations)
3. [Advanced Queries](#advanced-queries)
4. [Migrations and Seeders](#migrations-and-seeders)
5. [Transactions](#transactions)
6. [Join Strategies](#join-strategies)
7. [Optimizing Queries](#optimizing-queries)
8. [Handling Complex SQL Operations](#handling-complex-sql-operations)

## 1. Installation and Setup

Knex.js works with PostgreSQL and other SQL databases. Here's how to get started.

### Install Dependencies

First, install Knex and the PostgreSQL client for Node.js:

```bash
npm install knex pg
```

### Setup Configuration

Create a knexfile.js configuration file for your PostgreSQL connection:

```js
// knexfile.js
module.exports = {
  client: 'pg',
  connection: {
    host: 'localhost',
    user: 'your-username',
    password: 'your-password',
    database: 'your-database'
  }
};
```
In your application file, initialize Knex:

```js
const knex = require('knex');
const db = knex(require('./knexfile'));
```

## 2. Basic CRUD Operations

### SELECT (Read)

To retrieve all users:

```js
db('users').select('*').then(users => {
  console.log(users);
});
```

To retrieve users with a specific condition:

```js
db('users').where('age', '>', 18).select('name', 'email').then(users => {
  console.log(users);
});
```

### INSERT (Create)

To insert a new record into the users table:

```js
db('users').insert({ name: 'Neary', email: 'neary@example.com' })
  .then(() => console.log('User added!'))
  .catch(err => console.log(err));
```

### UPDATE (Update)

To update a user's information:

```js
db('users').where('id', 1).update({ name: 'Neary' })
  .then(() => console.log('User updated!'))
  .catch(err => console.log(err));
```

### DELETE (Delete)

To delete a user:

```js
db('users').where('id', 1).del()
  .then(() => console.log('User deleted!'))
  .catch(err => console.log(err));
```

## 3. Advanced Queries

Knex supports complex queries like joins, aggregations, and subqueries.

### Joins

To retrieve users with their associated posts:

```js
db('users')
  .join('posts', 'users.id', '=', 'posts.user_id')
  .select('users.name', 'posts.title')
  .then(results => console.log(results));
```

### Subqueries

To filter users based on posts they have created:

```js
db('users')
  .whereIn('id', db('posts').select('user_id').where('title', 'like', '%Knex%'))
  .then(users => console.log(users));
```

### Aggregation (Count, Sum, Average)

To count the number of posts by each user:

```js
db('posts')
  .count('id as post_count')
  .groupBy('user_id')
  .then(results => console.log(results));
```

## 4. Migrations and Seeders

### Migrations

Migrations help manage your database schema and track changes over time. To create a migration:

```bash
npx knex migrate:make create_users_table
```

The migration file might look like this:

```js
exports.up = function(knex) {
  return knex.schema.createTable('users', table => {
    table.increments('id').primary();
    table.string('name');
    table.string('email').unique();
  });
};

exports.down = function(knex) {
  return knex.schema.dropTable('users');
};
```

Run migrations with:

```bash
npx knex migrate:latest
```

### Seeders

Seeders are used to populate the database with sample data. Create a seeder:

```bash
npx knex seed:make users
```

The seeder file will look like this:

```js
exports.seed = function(knex) {
  return knex('users').del()
    .then(function () {
      return knex('users').insert([
        { name: 'Neary', email: 'neary@example.com' },
        { name: 'Romdoul', email: 'romdoul@example.com' }
      ]);
    });
};
```

Run seeders with:

```bash
npx knex seed:run
```

## 5. Transactions

Knex allows you to handle database transactions, ensuring data consistency during multiple operations.

### Example Transaction

```js
db.transaction(trx => {
  trx('users').insert({ name: 'Neary', email: 'neary@example.com' })
    .then(() => trx('posts').insert({ title: 'New Post', user_id: 1 }))
    .then(trx.commit)
    .catch(trx.rollback);
})
  .then(() => console.log('Transaction completed'))
  .catch(err => console.log('Transaction failed', err));
```

### Nesting Transactions

You can also nest transactions:

```js
db.transaction(trx => {
  return trx('users').insert({ name: 'Neary', email: 'neary@example.com' })
    .then(() => {
      return trx('posts').insert({ title: 'Another Post', user_id: 1 });
    });
})
  .then(() => console.log('Nested transaction completed'))
  .catch(err => console.log('Nested transaction failed', err));
```

## 6. Join Strategies

Knex allows you to optimize joins based on your data and query needs.

### Inner Join

```js
db('users')
  .innerJoin('posts', 'users.id', 'posts.user_id')
  .select('users.name', 'posts.title')
  .then(result => console.log(result));
```

### Left Join (Handling NULL values)

```js
db('users')
  .leftJoin('posts', 'users.id', 'posts.user_id')
  .select('users.name', 'posts.title')
  .then(result => console.log(result));
```

### Multiple Joins

```js
db('users')
  .leftJoin('posts', 'users.id', 'posts.user_id')
  .leftJoin('comments', 'posts.id', 'comments.post_id')
  .select('users.name', 'posts.title', 'comments.content')
  .then(result => console.log(result));
```

## 7. Optimizing Queries

When working with large datasets, it's important to optimize queries for performance.

### Indexing

Create indexes on frequently queried columns:

```js
db.schema.table('users', function (table) {
  table.index('email');
});
```

### Query Optimization

Use `EXPLAIN ANALYZE` to view query execution plans and optimize them accordingly.

### Pagination

To paginate results:

```js
db('users').limit(10).offset(20).then(rows => console.log(rows));
```

## 8. Handling Complex SQL Operations

Knex allows you to write and execute complex SQL queries easily.

### Window Functions

You can use window functions for advanced operations like ranking or partitioning results:

```js
db('posts')
  .select('id', 'title', db.raw('ROW_NUMBER() OVER (ORDER BY created_at DESC) AS row_num'))
  .then(rows => console.log(rows));
```

### CTE (Common Table Expressions)

Knex supports Common Table Expressions (CTEs) for breaking down complex queries:

```js
db.raw(`
  WITH user_posts AS (
    SELECT user_id, COUNT(*) AS post_count FROM posts GROUP BY user_id
  )
  SELECT users.name, user_posts.post_count FROM users
  JOIN user_posts ON users.id = user_posts.user_id
`)
  .then(rows => console.log(rows));
```

## Conclusion

Mastering Knex.js means mastering SQL queries, migrations, transactions, and optimizing complex operations. This README provided a structured path to help you enhance your skills from intermediate to advanced levels. With Knex, you can build powerful, efficient, and maintainable Node.js applications.

For additional information, please refer to the [official Knex documentation](https://knexjs.org/).