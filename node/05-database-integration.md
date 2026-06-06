# Database Integration in Node.js

## Table of Contents
1. [MongoDB & Mongoose](#mongodb--mongoose)
2. [SQL Databases](#sql-databases)
3. [Connection Pooling](#connection-pooling)
4. [Query Optimization](#query-optimization)
5. [Transactions](#transactions)

---

## MongoDB & Mongoose

### Installation
```bash
npm install mongoose
```

### Basic Connection
```javascript
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/mydb', {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log('Connected to MongoDB'))
.catch(error => console.error('Connection error:', error));
```

### Schema Definition
```javascript
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  age: {
    type: Number,
    min: 0,
    max: 120
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

const User = mongoose.model('User', userSchema);
```

### CRUD Operations
```javascript
// Create
const user = new User({
  name: 'John',
  email: 'john@example.com',
  age: 30
});
await user.save();

// Read one
const user = await User.findById(id);
const user = await User.findOne({ email: 'john@example.com' });

// Read many
const users = await User.find({ age: { $gt: 18 } });
const users = await User.find().limit(10).skip(20);

// Update
await User.findByIdAndUpdate(id, { name: 'Jane' }, { new: true });
await User.updateMany({ age: { $gt: 30 } }, { verified: true });

// Delete
await User.findByIdAndDelete(id);
await User.deleteMany({ age: { $lt: 18 } });
```

### Schema Methods
```javascript
userSchema.methods.getAge = function() {
  return new Date().getFullYear() - this.age;
};

userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email });
};

// Usage
const user = await User.findByEmail('john@example.com');
console.log(user.getAge());
```

### Middleware (Hooks)
```javascript
userSchema.pre('save', async function(next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

userSchema.post('save', function(doc, next) {
  console.log('User saved:', doc._id);
  next();
});
```

### Population (Joins)
```javascript
const postSchema = new mongoose.Schema({
  title: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
});

const Post = mongoose.model('Post', postSchema);

// Populate author
const post = await Post.findById(postId).populate('author');
console.log(post.author.name);

// Multiple populations
const post = await Post.findById(postId)
  .populate('author')
  .populate('comments.author');
```

### Indexing
```javascript
userSchema.index({ email: 1 }); // Single index
userSchema.index({ email: 1, name: 1 }); // Compound index
userSchema.index({ email: 1 }, { unique: true }); // Unique index

// TTL index (auto-delete after 1 hour)
userSchema.index({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

---

## SQL Databases

### Using MySQL with mysql2
```bash
npm install mysql2
```

### Basic Connection
```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'mydb',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

// Query
const connection = await pool.getConnection();
const [rows] = await connection.query('SELECT * FROM users WHERE id = ?', [1]);
connection.release();
```

### Using Sequelize ORM
```bash
npm install sequelize mysql2
```

#### Model Definition
```javascript
const { DataTypes } = require('sequelize');
const sequelize = require('./database');

const User = sequelize.define('User', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true
  },
  age: {
    type: DataTypes.INTEGER
  }
}, {
  timestamps: true
});

module.exports = User;
```

#### Sequelize CRUD
```javascript
// Create
const user = await User.create({
  name: 'John',
  email: 'john@example.com',
  age: 30
});

// Read
const user = await User.findByPk(1);
const users = await User.findAll({ where: { age: { [Op.gt]: 18 } } });

// Update
await user.update({ name: 'Jane' });
// or
await User.update({ verified: true }, { where: { age: { [Op.gt]: 30 } } });

// Delete
await user.destroy();
```

#### Associations
```javascript
const User = sequelize.define('User', { /* ... */ });
const Post = sequelize.define('Post', { /* ... */ });

// One-to-many
User.hasMany(Post, { foreignKey: 'userId' });
Post.belongsTo(User);

// Many-to-many
User.belongsToMany(Role, { through: 'UserRole' });
Role.belongsToMany(User, { through: 'UserRole' });

// Query with association
const user = await User.findByPk(1, {
  include: Post
});
```

---

## Connection Pooling

### Benefits
- Reuse connections
- Reduce overhead
- Improve performance
- Handle concurrent requests

### Pool Configuration
```javascript
const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'mydb',
  
  // Pool settings
  waitForConnections: true,
  connectionLimit: 10,      // Max connections
  queueLimit: 0,           // Unlimited queue
  
  // Connection settings
  enableKeepAlive: true,
  keepAliveInitialDelayMs: 0,
  
  // Timeouts
  connectTimeout: 10000,
  acquireTimeoutMillis: 30000
});
```

### Connection Lifecycle
```javascript
// Get connection
const connection = await pool.getConnection();

try {
  // Use connection
  const [rows] = await connection.query('SELECT * FROM users');
  console.log(rows);
} finally {
  // Always release
  connection.release();
}
```

---

## Query Optimization

### Indexing Strategy
```javascript
// Create indexes
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_created ON users(createdAt);
CREATE INDEX idx_email_status ON users(email, status);

// Analyze query
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';
```

### Query Examples
```javascript
// Good: Uses index
const user = await User.findOne({ email: 'john@example.com' });

// Bad: Full table scan
const users = await User.find({ age: { $gt: 18 } });

// Good: Limit results
const users = await User.find().limit(10);

// Good: Select specific fields
const users = await User.find({}, { name: 1, email: 1 });

// Good: Use lean() for read-only
const users = await User.find().lean();
```

### Aggregation Pipeline
```javascript
const results = await User.aggregate([
  { $match: { age: { $gte: 18 } } },
  { $group: { _id: '$city', count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 10 }
]);
```

### Query Caching
```javascript
const redis = require('redis');
const client = redis.createClient();

async function getUser(id) {
  // Check cache
  const cached = await client.get(`user:${id}`);
  if (cached) {
    return JSON.parse(cached);
  }

  // Query database
  const user = await User.findById(id);

  // Cache result
  await client.setex(`user:${id}`, 3600, JSON.stringify(user));

  return user;
}
```

---

## Transactions

### MongoDB Transactions
```javascript
const session = await mongoose.startSession();
session.startTransaction();

try {
  await User.updateOne({ _id: userId }, { balance: newBalance }, { session });
  await Transaction.create([{ /* ... */ }], { session });
  
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

### SQL Transactions
```javascript
const connection = await pool.getConnection();

try {
  await connection.beginTransaction();
  
  await connection.query('UPDATE accounts SET balance = balance - ? WHERE id = ?', [amount, fromId]);
  await connection.query('UPDATE accounts SET balance = balance + ? WHERE id = ?', [amount, toId]);
  
  await connection.commitTransaction();
} catch (error) {
  await connection.rollbackTransaction();
  throw error;
} finally {
  connection.release();
}
```

### Sequelize Transactions
```javascript
const t = await sequelize.transaction();

try {
  await User.update({ balance: 100 }, { where: { id: 1 }, transaction: t });
  await User.update({ balance: 200 }, { where: { id: 2 }, transaction: t });
  
  await t.commit();
} catch (error) {
  await t.rollback();
  throw error;
}
```

---

## Best Practices

1. ✅ Always use connection pooling
2. ✅ Create appropriate indexes
3. ✅ Use transactions for multi-step operations
4. ✅ Implement query caching
5. ✅ Monitor slow queries
6. ✅ Use pagination for large result sets
7. ✅ Validate input before queries
8. ✅ Use prepared statements to prevent SQL injection
9. ✅ Handle connection errors gracefully
10. ✅ Close connections properly
