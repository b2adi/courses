# Node.js Interview Questions & Coding Challenges

## Most Asked Interview Questions (Descending Order of Importance)

---

## ⭐⭐⭐ Critical Concepts

### 1. What is Node.js and how does it work?

**Answer:**
Node.js is a JavaScript runtime built on Chrome's V8 engine that allows JavaScript to run outside browsers. It uses an **event-driven, non-blocking I/O model**.

**Key characteristics:**
- Single-threaded but highly scalable
- Event loop processes asynchronous operations
- libuv library handles async operations
- Perfect for I/O-intensive applications

**How it works:**
1. Code execution
2. Event loop checks for events
3. Callbacks executed when events complete
4. Non-blocking operations allow other code to execute

```javascript
// Example
const fs = require('fs');

console.log('Start');

// Non-blocking operation
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  console.log('File read');
});

console.log('End');

// Output:
// Start
// End
// File read
```

---

### 2. Explain the Event Loop

**Answer:**
The event loop is the heart of Node.js. It continuously checks for events and executes their callbacks.

**Phases:**
1. **timers** - Execute callbacks scheduled by setTimeout/setInterval
2. **pending callbacks** - Execute deferred I/O callbacks
3. **idle, prepare** - Internal use
4. **poll** - Retrieve new I/O events
5. **check** - Execute setImmediate callbacks
6. **close callbacks** - Close event callbacks

```javascript
console.log('1: Start');

setTimeout(() => console.log('2: setTimeout'), 0);
setImmediate(() => console.log('3: setImmediate'));
Promise.resolve().then(() => console.log('4: Promise'));

console.log('5: End');

// Output:
// 1: Start
// 5: End
// 4: Promise
// 2: setTimeout
// 3: setImmediate
```

**Why?** Microtasks (Promises, queueMicrotask) execute before macrotasks (setTimeout, setImmediate).

---

### 3. Callbacks vs Promises vs Async/Await

**Answer:**

#### Callbacks
```javascript
// ❌ Callback Hell
function getData(callback) {
  fetchUser((err, user) => {
    if (err) callback(err);
    else {
      fetchPosts(user.id, (err, posts) => {
        if (err) callback(err);
        else callback(null, { user, posts });
      });
    }
  });
}
```

#### Promises
```javascript
// ✅ Better - avoids nesting
function getData() {
  return fetchUser()
    .then(user => fetchPosts(user.id).then(posts => ({ user, posts })))
    .catch(error => console.error(error));
}
```

#### Async/Await
```javascript
// ✅ Best - looks synchronous
async function getData() {
  try {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error(error);
  }
}
```

**Key differences:**

| Feature | Callbacks | Promises | Async/Await |
|---------|-----------|----------|------------|
| Readability | Poor | Good | Excellent |
| Error handling | Try-catch | .catch() | Try-catch |
| Composition | Difficult | Chainable | Clean |
| Learning curve | Easy | Medium | Medium |

---

### 4. What are Streams?

**Answer:**
Streams process data in chunks rather than loading everything into memory. Perfect for large files.

```javascript
const fs = require('fs');

// ❌ Bad - loads entire file
const data = fs.readFileSync('large.txt');

// ✅ Good - streams in chunks
fs.createReadStream('large.txt')
  .pipe(process.stdout);
```

**Types of streams:**
- **Readable** - Read data
- **Writable** - Write data
- **Transform** - Modify data passing through
- **Duplex** - Both readable and writable

```javascript
// Backpressure handling
const readStream = fs.createReadStream('large.txt');
const writeStream = fs.createWriteStream('copy.txt');

readStream.on('data', (chunk) => {
  const canContinue = writeStream.write(chunk);
  
  if (!canContinue) {
    readStream.pause();
    writeStream.once('drain', () => readStream.resume());
  }
});
```

---

### 5. Explain Middleware in Express

**Answer:**
Middleware are functions that have access to request, response, and next function.

```javascript
const express = require('express');
const app = express();

// Logging middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next(); // Pass to next middleware
});

// Authentication middleware
const authenticate = (req, res, next) => {
  if (req.headers.authorization) {
    next();
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
};

// Protected route
app.get('/admin', authenticate, (req, res) => {
  res.json({ message: 'Admin data' });
});

// Error middleware (must be last)
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});
```

**Order matters:** Middleware executes in registration order.

---

### 6. How do you handle errors in Node.js?

**Answer:**

```javascript
// Try-Catch
async function getData() {
  try {
    const data = await fetch('/api/data');
    return data;
  } catch (error) {
    console.error('Error:', error);
  }
}

// Promise .catch()
fetch('/api/data')
  .then(res => res.json())
  .catch(error => console.error(error));

// Global handlers
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
});

// Express error handler
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    error: err.message
  });
});
```

---

## ⭐⭐ Advanced Concepts

### 7. What's the difference between module.exports and exports?

**Answer:**
```javascript
// ❌ Won't work - exports is reference to module.exports
exports = { name: 'test' }; // Lost reference!

// ✅ Works
module.exports = { name: 'test' };

// ✅ Works
exports.name = 'test'; // Adding to object
```

**module.exports** is the actual export, **exports** is just a reference.

---

### 8. Explain CommonJS vs ES Modules

**Answer:**

```javascript
// CommonJS (Node.js default)
const fs = require('fs');
module.exports = { getData: () => {} };

// ES Modules (Modern)
import fs from 'fs';
export const getData = () => {};

// To use ES Modules:
// package.json: { "type": "module" }
// or .mjs file extension
```

**Differences:**
- CommonJS is synchronous
- ES Modules are asynchronous
- ES Modules support tree-shaking
- CommonJS is Node.js default

---

### 9. What's the purpose of package-lock.json?

**Answer:**
**package-lock.json** locks dependency versions to ensure consistent installs.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "lockfileVersion": 2,
  "requires": true,
  "packages": {
    "": {
      "name": "my-app",
      "version": "1.0.0",
      "dependencies": {
        "express": "4.18.2"
      }
    }
  }
}
```

**Benefits:**
- Same versions across environments
- Prevents dependency conflicts
- Faster installs (no resolution needed)
- Security - tracks known vulnerabilities

---

### 10. How do you prevent callback hell?

**Answer:**

```javascript
// ❌ Callback Hell
function process(callback) {
  readFile('1.txt', (err, data1) => {
    readFile('2.txt', (err, data2) => {
      readFile('3.txt', (err, data3) => {
        writeFile('output.txt', data1 + data2 + data3, callback);
      });
    });
  });
}

// ✅ Solution 1: Named functions
function readFile1() { return readFile('1.txt'); }
function readFile2() { return readFile('2.txt'); }
function readFile3() { return readFile('3.txt'); }
function writeOutput(data) { /* ... */ }

readFile1()
  .then(readFile2)
  .then(readFile3)
  .then(writeOutput);

// ✅ Solution 2: Async/Await
async function process() {
  const data1 = await readFile('1.txt');
  const data2 = await readFile('2.txt');
  const data3 = await readFile('3.txt');
  await writeFile('output.txt', data1 + data2 + data3);
}

// ✅ Solution 3: Promise.all
Promise.all([
  readFile('1.txt'),
  readFile('2.txt'),
  readFile('3.txt')
]).then(([d1, d2, d3]) => writeFile('output.txt', d1 + d2 + d3));
```

---

## 🔥 Coding Challenges (Descending Order of Importance)

---

### Challenge 1: Implement Promise.all()

**Difficulty:** ⭐⭐⭐

Implement a function that behaves like Promise.all()

**Solution:**
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Input is not iterable'));
    }

    if (promises.length === 0) {
      return resolve([]);
    }

    const results = new Array(promises.length);
    let resolvedCount = 0;

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          resolvedCount++;

          if (resolvedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}

// Test
promiseAll([
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3)
]).then(console.log); // [1, 2, 3]
```

---

### Challenge 2: Implement Retry Logic

**Difficulty:** ⭐⭐⭐

Create a function that retries failed operations with exponential backoff

**Solution:**
```javascript
async function retryWithBackoff(
  operation,
  maxRetries = 3,
  initialDelay = 1000
) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxRetries - 1) {
        throw error;
      }

      const delay = initialDelay * Math.pow(2, attempt);
      console.log(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms...`);
      
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
async function fetchData() {
  return await fetch('/api/data');
}

retryWithBackoff(fetchData, 3, 1000)
  .then(data => console.log(data))
  .catch(error => console.error('Failed after retries:', error));
```

---

### Challenge 3: Implement a Debounce Function

**Difficulty:** ⭐⭐

Create a debounce function that delays execution until after calls stop

**Solution:**
```javascript
function debounce(func, delay) {
  let timeoutId;

  return function(...args) {
    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// Usage
const handleSearch = debounce(async (query) => {
  const results = await fetch(`/api/search?q=${query}`);
  console.log(await results.json());
}, 300);

// Only fires once after user stops typing
input.addEventListener('input', (e) => handleSearch(e.target.value));
```

---

### Challenge 4: Implement Throttle Function

**Difficulty:** ⭐⭐

Create a throttle function that limits function execution frequency

**Solution:**
```javascript
function throttle(func, limit) {
  let inThrottle;

  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Usage
const handleScroll = throttle(() => {
  console.log('Scroll event');
}, 1000);

window.addEventListener('scroll', handleScroll);
```

---

### Challenge 5: Create a Custom EventEmitter

**Difficulty:** ⭐⭐⭐

Implement a basic EventEmitter class with on, emit, and off methods

**Solution:**
```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }

  emit(eventName, ...args) {
    if (!this.events[eventName]) return;

    this.events[eventName].forEach(callback => {
      callback(...args);
    });
  }

  off(eventName, callback) {
    if (!this.events[eventName]) return;

    this.events[eventName] = this.events[eventName].filter(
      cb => cb !== callback
    );
  }

  once(eventName, callback) {
    const onceWrapper = (...args) => {
      callback(...args);
      this.off(eventName, onceWrapper);
    };

    this.on(eventName, onceWrapper);
  }
}

// Usage
const emitter = new EventEmitter();

emitter.on('greet', (name) => console.log(`Hello, ${name}!`));
emitter.emit('greet', 'Node.js'); // Hello, Node.js!
```

---

### Challenge 6: Create a Simple File Logger

**Difficulty:** ⭐⭐

Create a logger that writes to file without blocking

**Solution:**
```javascript
const fs = require('fs');
const { EventEmitter } = require('events');

class FileLogger extends EventEmitter {
  constructor(filePath) {
    super();
    this.filePath = filePath;
    this.writeStream = fs.createWriteStream(filePath, { flags: 'a' });
  }

  log(level, message) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] ${level}: ${message}\n`;

    this.writeStream.write(logEntry, (err) => {
      if (err) {
        this.emit('error', err);
      } else {
        this.emit('logged', { level, message });
      }
    });
  }

  info(message) { this.log('INFO', message); }
  error(message) { this.log('ERROR', message); }
  warn(message) { this.log('WARN', message); }

  close() {
    this.writeStream.end();
  }
}

// Usage
const logger = new FileLogger('app.log');
logger.info('Application started');
logger.error('An error occurred');
logger.on('logged', ({ level, message }) => console.log(`Logged: ${level}`));
```

---

### Challenge 7: Implement Memoization

**Difficulty:** ⭐⭐

Create a memoization function for caching expensive computations

**Solution:**
```javascript
function memoize(func) {
  const cache = new Map();

  return function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log('From cache');
      return cache.get(key);
    }

    const result = func.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Usage
const expensiveCalculation = memoize((n) => {
  console.log('Computing...');
  let result = 0;
  for (let i = 0; i < n; i++) {
    result += i;
  }
  return result;
});

console.log(expensiveCalculation(1000)); // Computing...
console.log(expensiveCalculation(1000)); // From cache
```

---

### Challenge 8: Implement Queue Data Structure

**Difficulty:** ⭐⭐

Create a Queue class (FIFO) for managing async tasks

**Solution:**
```javascript
class Queue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }

  push(task) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, resolve, reject });
      this.process();
    });
  }

  async process() {
    while (this.running < this.concurrency && this.queue.length > 0) {
      this.running++;
      const { task, resolve, reject } = this.queue.shift();

      try {
        const result = await task();
        resolve(result);
      } catch (error) {
        reject(error);
      } finally {
        this.running--;
        this.process();
      }
    }
  }
}

// Usage
const queue = new Queue(2); // Max 2 concurrent tasks

queue.push(async () => {
  await new Promise(r => setTimeout(r, 1000));
  console.log('Task 1');
});

queue.push(async () => {
  await new Promise(r => setTimeout(r, 500));
  console.log('Task 2');
});
```

---

### Challenge 9: Parse URL Query String

**Difficulty:** ⭐

Parse a URL query string into an object

**Solution:**
```javascript
function parseQuery(queryString) {
  const params = {};

  if (!queryString) return params;

  queryString
    .replace(/^\?/, '') // Remove leading ?
    .split('&')
    .forEach(param => {
      const [key, value] = param.split('=');
      params[decodeURIComponent(key)] = decodeURIComponent(value || '');
    });

  return params;
}

// Usage
const query = parseQuery('?name=john&age=30&city=NYC');
console.log(query); 
// { name: 'john', age: '30', city: 'NYC' }
```

---

### Challenge 10: Flatten Deep Nested Array

**Difficulty:** ⭐⭐

Create a function to flatten deeply nested arrays

**Solution:**
```javascript
function flatten(arr) {
  return arr.reduce((result, item) => {
    if (Array.isArray(item)) {
      result.push(...flatten(item));
    } else {
      result.push(item);
    }
    return result;
  }, []);
}

// Usage
const nested = [1, [2, [3, [4, 5]]], 6];
console.log(flatten(nested)); // [1, 2, 3, 4, 5, 6]

// Or using flat() method
console.log(nested.flat(Infinity)); // [1, 2, 3, 4, 5, 6]
```

---

## Summary

**Key Takeaways:**
1. Master asynchronous programming (callbacks, promises, async/await)
2. Understand the event loop
3. Know streams and when to use them
4. Understand middleware pattern
5. Handle errors properly
6. Optimize for performance
7. Practice coding challenges

**Preparation Tips:**
- Code regularly
- Understand concepts deeply, not just syntax
- Practice live coding
- Discuss trade-offs
- Ask clarifying questions

