# Express.js & Web Servers

## Table of Contents
1. [Express Basics](#express-basics)
2. [Routing](#routing)
3. [Middleware](#middleware)
4. [Request & Response](#request--response)
5. [Error Handling](#error-handling)
6. [REST API Best Practices](#rest-api-best-practices)

---

## Express Basics

### Installation & Setup
```bash
npm install express
```

### Hello World Application
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Using ES Modules
```javascript
import express from 'express';

const app = express();
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### App Configuration
```javascript
const app = express();

// Set port
app.set('port', 3000);

// Set view engine
app.set('view engine', 'ejs');

// Set views directory
app.set('views', './views');

// Enable trust proxy
app.set('trust proxy', true);
```

---

## Routing

### Basic Routes
```javascript
// GET request
app.get('/users', (req, res) => {
  res.json({ message: 'Get all users' });
});

// POST request
app.post('/users', (req, res) => {
  res.json({ message: 'Create user' });
});

// PUT request
app.put('/users/:id', (req, res) => {
  res.json({ message: `Update user ${req.params.id}` });
});

// DELETE request
app.delete('/users/:id', (req, res) => {
  res.json({ message: `Delete user ${req.params.id}` });
});
```

### Route Parameters
```javascript
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ userId });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});
```

### Query Strings
```javascript
// GET /users?page=1&limit=10
app.get('/users', (req, res) => {
  const page = req.query.page || 1;
  const limit = req.query.limit || 10;
  res.json({ page, limit });
});
```

### Router Objects
```javascript
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ message: 'Get all products' });
});

router.post('/', (req, res) => {
  res.json({ message: 'Create product' });
});

module.exports = router;
```

### Using Routers
```javascript
const express = require('express');
const productRoutes = require('./routes/products');

const app = express();

app.use('/api/products', productRoutes);
```

---

## Middleware

Middleware functions execute during request-response cycle.

### Built-in Middleware
```javascript
const express = require('express');
const app = express();

// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static('public'));
```

### Custom Middleware
```javascript
// Request logging middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});

// Error-handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal Server Error' });
});
```

### Route-Specific Middleware
```javascript
const authenticate = (req, res, next) => {
  if (req.headers.authorization) {
    next();
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
};

app.get('/protected', authenticate, (req, res) => {
  res.json({ message: 'Protected route' });
});
```

### Multiple Middleware
```javascript
const auth = (req, res, next) => {
  // Check authentication
  next();
};

const authorize = (req, res, next) => {
  // Check authorization
  next();
};

app.delete('/users/:id', auth, authorize, (req, res) => {
  res.json({ message: 'User deleted' });
});
```

### Third-party Middleware
```javascript
const cors = require('cors');
const helmet = require('helmet');

app.use(cors());
app.use(helmet());
```

---

## Request & Response

### Request Object
```javascript
app.post('/data', (req, res) => {
  // URL parameters
  req.params;

  // Query strings
  req.query;

  // Request body
  req.body;

  // Headers
  req.headers;

  // HTTP method
  req.method;

  // URL path
  req.path;

  // Original URL
  req.originalUrl;

  // Remote IP
  req.ip;

  // Host
  req.hostname;
});
```

### Response Methods
```javascript
app.get('/users', (req, res) => {
  // Send plain text
  res.send('Hello');

  // Send JSON
  res.json({ key: 'value' });

  // Send file
  res.sendFile('/path/to/file');

  // Render template
  res.render('template', { data: 'value' });

  // Set status and send
  res.status(201).json({ id: 1 });

  // Set header
  res.set('Content-Type', 'application/json');

  // Redirect
  res.redirect('/new-url');

  // Send file for download
  res.download('/path/file.pdf');
});
```

### Response Headers
```javascript
app.get('/data', (req, res) => {
  res.set({
    'Content-Type': 'application/json',
    'X-Custom-Header': 'value'
  });
  
  res.json({ data: 'value' });
});
```

---

## Error Handling

### Try-Catch in Routes
```javascript
app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Global Error Handler
```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  res.status(err.status || 500).json({
    error: {
      message: err.message,
      status: err.status || 500
    }
  });
});
```

### Custom Error Class
```javascript
class AppError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
  }
}

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      throw new AppError('User not found', 404);
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

---

## REST API Best Practices

### Proper Status Codes
```javascript
// 200 OK - Successful GET
app.get('/users', (req, res) => {
  res.json(users);
});

// 201 Created - Successful POST
app.post('/users', (req, res) => {
  res.status(201).json(newUser);
});

// 204 No Content - Successful DELETE
app.delete('/users/:id', (req, res) => {
  res.status(204).send();
});

// 400 Bad Request
app.post('/users', (req, res) => {
  if (!req.body.name) {
    return res.status(400).json({ error: 'Name required' });
  }
});

// 401 Unauthorized
app.get('/admin', authenticate, (req, res) => {
  // Only authenticated users
});

// 403 Forbidden
app.get('/admin', authenticate, authorize, (req, res) => {
  // Only authorized users
});

// 404 Not Found
app.get('/users/:id', (req, res) => {
  if (!user) {
    return res.status(404).json({ error: 'Not found' });
  }
});

// 500 Internal Server Error
app.get('/data', (req, res) => {
  // Handled by error middleware
});
```

### Consistent Response Format
```javascript
// Success response
{
  "status": "success",
  "data": { /* actual data */ }
}

// Error response
{
  "status": "error",
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input"
  }
}
```

### API Versioning
```javascript
const v1 = express.Router();
const v2 = express.Router();

v1.get('/users', (req, res) => {
  res.json({ version: 1, users: [] });
});

v2.get('/users', (req, res) => {
  res.json({ version: 2, users: [] });
});

app.use('/api/v1', v1);
app.use('/api/v2', v2);
```

### Pagination
```javascript
app.get('/users', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const users = allUsers.slice(skip, skip + limit);
  
  res.json({
    data: users,
    pagination: {
      page,
      limit,
      total: allUsers.length
    }
  });
});
```

---

## Complete Example: User API

```javascript
const express = require('express');
const app = express();

app.use(express.json());

let users = [];
let nextId = 1;

// GET all users
app.get('/api/users', (req, res) => {
  res.json({ data: users });
});

// GET user by ID
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json({ data: user });
});

// CREATE user
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email required' });
  }

  const user = { id: nextId++, name, email };
  users.push(user);
  
  res.status(201).json({ data: user });
});

// UPDATE user
app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  user.name = req.body.name || user.name;
  user.email = req.body.email || user.email;

  res.json({ data: user });
});

// DELETE user
app.delete('/api/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  const [deleted] = users.splice(index, 1);
  res.json({ data: deleted });
});

// Error handler
app.use((err, req, res, next) => {
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000, () => console.log('Server on port 3000'));
```
