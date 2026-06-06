# Asynchronous Programming in Node.js

## Table of Contents
1. [Callbacks](#callbacks)
2. [Promises](#promises)
3. [Async/Await](#asyncawait)
4. [Error Handling](#error-handling)
5. [Promise Patterns](#promise-patterns)

---

## Callbacks

Callbacks are functions passed as arguments to be executed later.

### Basic Example
```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback(null, 'Data fetched');
  }, 1000);
}

fetchData((err, data) => {
  if (err) console.error(err);
  console.log(data); // Output: Data fetched
});
```

### Callback Hell (Pyramid of Doom)
```javascript
// ❌ Avoid this
function processData() {
  readFile('file1.txt', (err, data1) => {
    processFile(data1, (err, data2) => {
      saveFile(data2, (err, result) => {
        notifyUser(result, (err, notification) => {
          console.log('Done');
        });
      });
    });
  });
}
```

### Node.js Callback Convention
```javascript
// Error-first callback
function operation(param, callback) {
  if (error) {
    callback(error); // Error as first argument
  } else {
    callback(null, result); // null as first argument if no error
  }
}
```

---

## Promises

A Promise represents a value that may not be available yet but will be resolved eventually.

### Promise States
1. **Pending** - Initial state
2. **Resolved (Fulfilled)** - Operation completed successfully
3. **Rejected** - Operation failed

### Creating Promises
```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('Success!');
  }, 1000);
});

promise
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

### Promise Chain
```javascript
fetch('https://api.example.com/user/1')
  .then(response => response.json())
  .then(user => {
    console.log(user);
    return fetch(`https://api.example.com/posts/${user.id}`);
  })
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error(error));
```

### Promise Utility Methods
```javascript
// Wait for all promises
Promise.all([promise1, promise2, promise3])
  .then(results => console.log(results))
  .catch(error => console.error('One failed:', error));

// Wait for first to resolve
Promise.race([promise1, promise2])
  .then(result => console.log('First:', result));

// Wait for all (settled or rejected)
Promise.allSettled([promise1, promise2])
  .then(results => console.log(results));

// Create resolved promise
Promise.resolve(value).then(v => console.log(v));

// Create rejected promise
Promise.reject(error).catch(e => console.error(e));
```

---

## Async/Await

Modern syntax for handling asynchronous operations, built on top of Promises.

### Basic Syntax
```javascript
async function getData() {
  try {
    const response = await fetch('https://api.example.com/user');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
  }
}

getData().then(data => console.log(data));
```

### Key Points
- `async` function always returns a Promise
- `await` pauses execution until Promise settles
- Can only use `await` in `async` functions
- Makes code look synchronous (more readable)

### Parallel Execution
```javascript
// ❌ Sequential (slower)
async function process() {
  const result1 = await fetch('url1');
  const result2 = await fetch('url2');
  return [result1, result2];
}

// ✅ Parallel (faster)
async function process() {
  const [result1, result2] = await Promise.all([
    fetch('url1'),
    fetch('url2')
  ]);
  return [result1, result2];
}
```

### Top-Level Await
```javascript
// Node.js 14.8+
const data = await fetch('https://api.example.com/data');
console.log(data);
```

---

## Error Handling

### Try-Catch with Async/Await
```javascript
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw error; // Re-throw for caller handling
  } finally {
    console.log('Request completed');
  }
}
```

### Promise .catch()
```javascript
fetch('/api/data')
  .then(response => response.json())
  .then(data => processData(data))
  .catch(error => {
    console.error('Error:', error);
    // Handle error
  });
```

### Global Error Handlers
```javascript
// Unhandled promise rejection
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});

// Uncaught exception
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1);
});
```

---

## Promise Patterns

### Retry Pattern
```javascript
async function retryOperation(operation, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}

// Usage
retryOperation(() => fetch('/api/data'));
```

### Timeout Pattern
```javascript
function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
}

// Usage
const result = await withTimeout(fetch('/api/data'), 5000);
```

### Queue Pattern
```javascript
async function processQueue(items, concurrency = 2) {
  const results = [];
  for (let i = 0; i < items.length; i += concurrency) {
    const batch = items.slice(i, i + concurrency);
    const batchResults = await Promise.all(
      batch.map(item => processItem(item))
    );
    results.push(...batchResults);
  }
  return results;
}
```

### Debounce Pattern
```javascript
function debounce(fn, delay) {
  let timeoutId;
  return async function(...args) {
    clearTimeout(timeoutId);
    return new Promise(resolve => {
      timeoutId = setTimeout(() => resolve(fn(...args)), delay);
    });
  };
}

// Usage
const debouncedSearch = debounce(async (query) => {
  return await fetch(`/api/search?q=${query}`);
}, 300);
```

---

## Best Practices

1. ✅ Prefer `async/await` over `.then()` chains
2. ✅ Always handle errors with `try-catch`
3. ✅ Use `Promise.all()` for parallel operations
4. ✅ Avoid creating promises when unnecessary
5. ✅ Don't nest promises unnecessarily
6. ✅ Handle unhandled rejections at application level

