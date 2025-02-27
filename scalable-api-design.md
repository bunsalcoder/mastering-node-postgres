# Building Scalable and Maintainable Systems

## 1. API Design

### RESTful vs GraphQL APIs
- **RESTful APIs**: Follows a structured approach with endpoints for each resource (e.g., `/users`, `/posts`).
- **GraphQL APIs**: Allows fetching only required data in a single request, reducing over-fetching and under-fetching.

### Example: RESTful API with Express
```javascript
const express = require('express');
const app = express();
app.use(express.json());

app.get('/users/:id', (req, res) => {
    res.json({ id: req.params.id, name: "Pisey" });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

### Example: GraphQL API with Apollo Server
```javascript
const { ApolloServer, gql } = require('apollo-server');

const typeDefs = gql`
  type User {
    id: ID!
    name: String!
  }
  type Query {
    user(id: ID!): User
  }
`;
const resolvers = {
  Query: {
    user: (_, { id }) => ({ id, name: "Pisey" }),
  },
};
const server = new ApolloServer({ typeDefs, resolvers });
server.listen().then(({ url }) => console.log(`Server running at ${url}`));
```

---

## 2. Authentication & Authorization

### JWT-based Authentication with Passport.js
```javascript
const passport = require('passport');
const JwtStrategy = require('passport-jwt').Strategy;
const ExtractJwt = require('passport-jwt').ExtractJwt;
const opts = {
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: 'your_secret_key',
};

passport.use(new JwtStrategy(opts, (jwt_payload, done) => {
  if (jwt_payload.id) return done(null, jwt_payload);
  return done(null, false);
}));
```

---

## 3. Error Handling

### Using Middleware for Error Handling
```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Internal Server Error' });
});
```

---

## 4. Caching with Redis

### Installing and Connecting Redis
```sh
npm install redis
```
```javascript
const redis = require('redis');
const client = redis.createClient();
client.on('error', (err) => console.error('Redis error:', err));

// Caching middleware
const cacheMiddleware = (req, res, next) => {
  const key = `user:${req.params.id}`;
  client.get(key, (err, data) => {
    if (data) return res.json(JSON.parse(data));
    next();
  });
};
```

---

## 5. Testing

### Writing Unit Tests with Jest
```sh
npm install --save-dev jest
```
```javascript
test('adds 1 + 2 to equal 3', () => {
  expect(1 + 2).toBe(3);
});
```

### Integration Testing with Supertest
```sh
npm install --save-dev supertest
```
```javascript
const request = require('supertest');
const app = require('../app');

test('GET /users', async () => {
  const res = await request(app).get('/users/1');
  expect(res.statusCode).toBe(200);
  expect(res.body).toHaveProperty('name');
});
```

