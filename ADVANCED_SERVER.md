Great! Here's how you can extend your `json-server` setup to include:

* ✅ **Custom authentication (login + JWT)**
* ✅ **JWT token verification middleware**
* ✅ **Rate limiting (per IP)**

---

## 📁 Project Structure

```
your-project/
├── db.json
├── routes.json
├── server.js
├── auth.js
├── middleware/
│   ├── verifyToken.js
│   └── rateLimiter.js
├── package.json
```

---

## ✅ Step 1: Install Dependencies

```bash
npm install json-server jsonwebtoken express-rate-limit
```

---

## ✅ Step 2: Create a User in `db.json`

```json
// db.json
{
  "users": [
    {
      "id": 1,
      "username": "admin",
      "password": "123456" // In production, use hashed passwords
    }
  ],
  "posts": []
}
```

---

## ✅ Step 3: Create JWT Auth Logic (`auth.js`)

```js
// auth.js
const jsonServer = require('json-server');
const jwt = require('jsonwebtoken');
const fs = require('fs');

const SECRET_KEY = 'your-secret-key';
const expiresIn = '1h';

// Generate token
function createToken(payload) {
  return jwt.sign(payload, SECRET_KEY, { expiresIn });
}

// Verify token
function verifyToken(token) {
  return jwt.verify(token, SECRET_KEY);
}

// Authenticate user
function isAuthenticated({ username, password }) {
  const users = JSON.parse(fs.readFileSync('./db.json', 'UTF-8')).users;
  return users.find(user => user.username === username && user.password === password);
}

module.exports = {
  createToken,
  verifyToken,
  isAuthenticated
};
```

---

## ✅ Step 4: JWT Middleware (`middleware/verifyToken.js`)

```js
// middleware/verifyToken.js
const { verifyToken } = require('../auth');

function verifyJWT(req, res, next) {
  if (
    req.method === 'GET' || // Allow public GET
    req.url.startsWith('/login') || 
    req.url.startsWith('/register')
  ) return next();

  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ message: 'Missing or invalid token' });
  }

  try {
    const token = authHeader.split(' ')[1];
    const decoded = verifyToken(token);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ message: 'Token expired or invalid' });
  }
}

module.exports = verifyJWT;
```

---

## ✅ Step 5: Rate Limiting Middleware (`middleware/rateLimiter.js`)

```js
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per window
  message: 'Too many requests from this IP, please try again later'
});

module.exports = limiter;
```

---

## ✅ Step 6: Final `server.js`

```js
// server.js
const jsonServer = require('json-server');
const express = require('express');
const bodyParser = require('body-parser');
const { createToken, isAuthenticated } = require('./auth');
const verifyJWT = require('./middleware/verifyToken');
const rateLimiter = require('./middleware/rateLimiter');

const server = express();
const router = jsonServer.router('db.json');
const middlewares = jsonServer.defaults();

server.use(bodyParser.urlencoded({ extended: true }));
server.use(bodyParser.json());

// Use rate limiter globally
server.use(rateLimiter);

// Login endpoint
server.post('/login', (req, res) => {
  const { username, password } = req.body;

  const user = isAuthenticated({ username, password });
  if (!user) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  const token = createToken({ id: user.id, username: user.username });
  res.status(200).json({ token });
});

// Middleware to protect routes
server.use(verifyJWT);

// Custom route remapping
server.use(jsonServer.rewriter({
  "/api/*": "/$1"
}));

server.use(middlewares);
server.use(router);

// Start server
server.listen(3000, () => {
  console.log('🚀 JSON Server running at http://localhost:3000');
});
```

---

## ✅ Step 7: Test Auth & Rate Limit

* **Login to get token**
  `POST http://localhost:3000/login`

  ```json
  {
    "username": "admin",
    "password": "123456"
  }
  ```

* **Use token to access protected routes**
  Add `Authorization` header:

  ```
  Authorization: Bearer YOUR_TOKEN
  ```

* **Rate limit** will block if you exceed 100 requests in 15 mins per IP.

---

## 🧪 Extra Ideas

* 🔐 Use `bcryptjs` to hash passwords
* 🔄 Add `/register` route to allow new user creation
* ⚠️ Add role-based access control (admin/user)
* 💾 Store tokens in localStorage for frontend apps

---
