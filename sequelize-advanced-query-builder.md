# Sequelize – ORM for Node.js

Sequelize is a powerful ORM (Object-Relational Mapping) library for Node.js that simplifies database interactions by abstracting raw SQL queries into JavaScript objects. It supports PostgreSQL and provides a structured way to manage database operations.

## Table of Contents

1. [Installation and Setup](#1-installation-and-setup)
2. [Sequelize Fundamentals](#2-sequelize-fundamentals)
3. [CRUD Operations](#3-crud-operations)
4. [Associations](#4-associations)
5. [Advanced Concepts](#5-advanced-concepts)
   - [Eager vs Lazy Loading](#eager-vs-lazy-loading)
   - [Transactions](#transactions)
   - [Custom Queries](#custom-queries)
   - [Indexes and Performance](#indexes-and-performance)
   - [Data Validation & Hooks](#data-validation--hooks)

---

## 1. Installation and Setup

To use Sequelize with PostgreSQL, install the required dependencies:

```sh
npm install sequelize pg pg-hstore
```

### Initializing Sequelize

```javascript
const { Sequelize } = require("sequelize");
const sequelize = new Sequelize("database", "username", "password", {
  host: "localhost",
  dialect: "postgres",
});

(async () => {
  try {
    await sequelize.authenticate();
    console.log("Connection has been established successfully.");
  } catch (error) {
    console.error("Unable to connect to the database:", error);
  }
})();
```

---

## 2. Sequelize Fundamentals

### Defining a Model

```javascript
const { DataTypes } = require("sequelize");

const User = sequelize.define("User", {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
  },
});
```

---

## 3. CRUD Operations

### Create a Record

```javascript
const newUser = await User.create({ name: "Samnang", email: "samnang@example.com" });
console.log(newUser.toJSON());
```

### Read Records

```javascript
const users = await User.findAll();
console.log(users);
```

### Update a Record

```javascript
await User.update({ name: "Samnang" }, { where: { id: 1 } });
```

### Delete a Record

```javascript
await User.destroy({ where: { id: 1 } });
```

---

## 4. Associations

### One-to-Many Relationship

```javascript
const Post = sequelize.define("Post", {
  title: DataTypes.STRING,
  content: DataTypes.TEXT,
});

User.hasMany(Post);
Post.belongsTo(User);
```

### Many-to-Many Relationship

```javascript
const UserRole = sequelize.define("UserRole", {});
User.belongsToMany(Role, { through: UserRole });
Role.belongsToMany(User, { through: UserRole });
```

---

## 5. Advanced Concepts

### Eager vs Lazy Loading

#### Lazy Loading:
```javascript
const user = await User.findByPk(1);
const posts = await user.getPosts();
```

#### Eager Loading:
```javascript
const userWithPosts = await User.findOne({
  where: { id: 1 },
  include: Post,
});
```

### Transactions

```javascript
const transaction = await sequelize.transaction();
try {
  await User.create({ name: "Samnang" }, { transaction });
  await transaction.commit();
} catch (error) {
  await transaction.rollback();
}
```

### Custom Queries

```javascript
const users = await sequelize.query("SELECT * FROM Users", { type: Sequelize.QueryTypes.SELECT });
console.log(users);
```

### Indexes and Performance

```javascript
const Product = sequelize.define("Product", {
  name: { type: DataTypes.STRING, unique: true },
});
```

### Data Validation & Hooks

#### Adding Validation:
```javascript
const User = sequelize.define("User", {
  email: {
    type: DataTypes.STRING,
    validate: {
      isEmail: true,
    },
  },
});
```

#### Using Hooks:
```javascript
User.beforeCreate((user) => {
  user.name = user.name.toUpperCase();
});
```

---

For more details, refer to the [official Sequelize documentation](https://sequelize.org/).

