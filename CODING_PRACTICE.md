# JavaScript Coding Practice Exercises

Complete collection of practice problems to reinforce interview concepts.

---

## 🟢 EASY LEVEL (Start Here)

### 1. Simple Counter with Closure
**Problem:** Create a function that returns a counter object with increment and decrement methods.

```javascript
// Solution
function createCounter() {
  let count = 0;
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount()); // 1
```

---

### 2. Flatten Array
**Problem:** Write a function to flatten a nested array.

```javascript
// Solution 1: Using flat()
function flatten(arr) {
  return arr.flat(Infinity);
}

// Solution 2: Using recursion
function flattenRecursive(arr) {
  return arr.reduce((flat, item) => 
    flat.concat(Array.isArray(item) ? flattenRecursive(item) : item), 
  []);
}

console.log(flatten([1, [2, [3, [4]]]])); // [1, 2, 3, 4]
```

---

### 3. Remove Duplicates
**Problem:** Remove duplicate values from an array.

```javascript
// Solution 1: Using Set
function removeDuplicates(arr) {
  return [...new Set(arr)];
}

// Solution 2: Using filter
function removeDuplicatesFilter(arr) {
  return arr.filter((value, index, self) => self.indexOf(value) === index);
}

console.log(removeDuplicates([1, 2, 2, 3, 3, 3])); // [1, 2, 3]
```

---

### 4. Debounce Function
**Problem:** Create a debounce function that delays execution until activity stops.

```javascript
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func(...args), delay);
  };
}

// Usage
const search = debounce((query) => console.log(`Searching: ${query}`), 300);
search('j');
search('ja');
search('jav');
search('java'); // Only this logs after 300ms
```

---

### 5. Throttle Function
**Problem:** Create a throttle function that limits execution frequency.

```javascript
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Usage
const scroll = throttle(() => console.log('Scrolled'), 1000);
// Logs max once per second
```

---

### 6. Find Maximum Number
**Problem:** Find the maximum number in an array.

```javascript
// Solution 1: Using Math.max
function findMax(arr) {
  return Math.max(...arr);
}

// Solution 2: Using reduce
function findMaxReduce(arr) {
  return arr.reduce((max, num) => num > max ? num : max);
}

console.log(findMax([1, 5, 3, 9, 2])); // 9
```

---

### 7. Reverse String
**Problem:** Reverse a string.

```javascript
// Solution 1: Using built-in methods
function reverseString(str) {
  return str.split('').reverse().join('');
}

// Solution 2: Using loop
function reverseStringLoop(str) {
  let result = '';
  for (let i = str.length - 1; i >= 0; i--) {
    result += str[i];
  }
  return result;
}

console.log(reverseString('hello')); // 'olleh'
```

---

### 8. Check if Palindrome
**Problem:** Check if a string is a palindrome.

```javascript
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === cleaned.split('').reverse().join('');
}

console.log(isPalindrome('A man, a plan, a canal: Panama')); // true
```

---

### 9. Deep Clone Object
**Problem:** Create a deep copy of an object.

```javascript
// Solution 1: Using JSON (limited)
function deepCloneJSON(obj) {
  return JSON.parse(JSON.stringify(obj));
}

// Solution 2: Recursive (better)
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(el => deepClone(el));
  if (obj instanceof Object) {
    const cloned = {};
    for (let key in obj) {
      if (obj.hasOwnProperty(key)) {
        cloned[key] = deepClone(obj[key]);
      }
    }
    return cloned;
  }
}

const original = { a: { b: { c: 1 } } };
const cloned = deepClone(original);
cloned.a.b.c = 2;
console.log(original.a.b.c); // 1 (unchanged)
```

---

### 10. First Non-Repeating Character
**Problem:** Find the first non-repeating character in a string.

```javascript
function firstNonRepeating(str) {
  const chars = {};
  for (let char of str) {
    chars[char] = (chars[char] || 0) + 1;
  }
  for (let char of str) {
    if (chars[char] === 1) return char;
  }
  return null;
}

console.log(firstNonRepeating('abracadabra')); // 'c'
```

---

## 🟡 MEDIUM LEVEL

### 11. Implement Promise
**Problem:** Create a simplified Promise implementation.

```javascript
class SimplePromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.handlers = [];

    executor(
      (value) => this.resolve(value),
      (error) => this.reject(error)
    );
  }

  resolve(value) {
    if (this.state === 'pending') {
      this.state = 'fulfilled';
      this.value = value;
      this.handlers.forEach(h => h.onFulfilled(value));
    }
  }

  reject(error) {
    if (this.state === 'pending') {
      this.state = 'rejected';
      this.value = error;
      this.handlers.forEach(h => h.onRejected(error));
    }
  }

  then(onFulfilled, onRejected) {
    return new SimplePromise((resolve, reject) => {
      this.handlers.push({
        onFulfilled: (value) => {
          try {
            resolve(onFulfilled(value));
          } catch (e) {
            reject(e);
          }
        },
        onRejected: (error) => {
          try {
            reject(onRejected(error));
          } catch (e) {
            reject(e);
          }
        }
      });
    });
  }
}
```

---

### 12. Curry Function
**Problem:** Create a currying utility.

```javascript
function curry(fn) {
  const arity = fn.length;

  return function $curry(...args) {
    if (args.length < arity) {
      return $curry.bind(null, ...args);
    }
    return fn.call(null, ...args);
  };
}

// Usage
const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
```

---

### 13. Memoization
**Problem:** Create a memoization function to cache results.

```javascript
function memoize(fn) {
  const cache = {};

  return function(...args) {
    const key = JSON.stringify(args);
    
    if (key in cache) {
      return cache[key];
    }

    const result = fn(...args);
    cache[key] = result;
    return result;
  };
}

// Usage
const fibonacci = memoize((n) => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // 55
```

---

### 14. Event Emitter
**Problem:** Create a simple event emitter class.

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }

  off(event, listenerToRemove) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(
      listener => listener !== listenerToRemove
    );
  }

  emit(event, ...args) {
    if (!this.events[event]) return;
    this.events[event].forEach(listener => listener(...args));
  }
}

// Usage
const emitter = new EventEmitter();
emitter.on('greet', (name) => console.log(`Hello ${name}`));
emitter.emit('greet', 'World'); // Hello World
```

---

### 15. Implement Array.map()
**Problem:** Create your own map function.

```javascript
Array.prototype.myMap = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i, this));
  }
  return result;
};

// Usage
const nums = [1, 2, 3];
const doubled = nums.myMap(x => x * 2);
console.log(doubled); // [2, 4, 6]
```

---

### 16. Implement Array.filter()
**Problem:** Create your own filter function.

```javascript
Array.prototype.myFilter = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};

// Usage
const nums = [1, 2, 3, 4, 5];
const evens = nums.myFilter(x => x % 2 === 0);
console.log(evens); // [2, 4]
```

---

### 17. Implement Array.reduce()
**Problem:** Create your own reduce function.

```javascript
Array.prototype.myReduce = function(callback, initial) {
  let acc = initial;
  let start = initial === undefined ? 1 : 0;
  
  if (initial === undefined) acc = this[0];

  for (let i = start; i < this.length; i++) {
    acc = callback(acc, this[i], i, this);
  }
  return acc;
};

// Usage
const nums = [1, 2, 3, 4];
const sum = nums.myReduce((acc, val) => acc + val, 0);
console.log(sum); // 10
```

---

### 18. Batch API Calls
**Problem:** Execute API calls in batches with concurrency limit.

```javascript
async function batchRequests(urls, batchSize = 3) {
  const results = [];
  for (let i = 0; i < urls.length; i += batchSize) {
    const batch = urls.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(url => fetch(url).then(r => r.json()))
    );
    results.push(...batchResults);
  }
  return results;
}
```

---

### 19. Retry Mechanism
**Problem:** Implement retry logic for failed operations.

```javascript
async function retry(fn, retries = 3, delay = 1000) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
const result = await retry(() => fetch('/api/data'), 3, 1000);
```

---

### 20. Compose Functions
**Problem:** Create a function composition utility.

```javascript
function compose(...fns) {
  return (x) => fns.reduceRight((acc, fn) => fn(acc), x);
}

// Usage
const add1 = (x) => x + 1;
const double = (x) => x * 2;
const subtract3 = (x) => x - 3;

const composed = compose(subtract3, double, add1);
console.log(composed(5)); // ((5 + 1) * 2) - 3 = 9
```

---

## 🔴 HARD LEVEL

### 21. Implement Async Queue
**Problem:** Create a queue that processes items asynchronously.

```javascript
class AsyncQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }

  async add(fn) {
    return new Promise((resolve) => {
      this.queue.push({ fn, resolve });
      this.process();
    });
  }

  async process() {
    while (this.running < this.concurrency && this.queue.length > 0) {
      this.running++;
      const { fn, resolve } = this.queue.shift();
      try {
        const result = await fn();
        resolve(result);
      } finally {
        this.running--;
        this.process();
      }
    }
  }
}
```

---

### 22. Implement Lodash Debounce Advanced
**Problem:** Create an advanced debounce with leading and trailing options.

```javascript
function debounce(func, delay, options = {}) {
  let timeoutId;
  let lastCallTime;
  let lastResult;

  return function(...args) {
    const now = Date.now();
    const isLeading = options.leading && !lastCallTime;

    if (isLeading) {
      lastResult = func(...args);
    }

    clearTimeout(timeoutId);
    lastCallTime = now;

    timeoutId = setTimeout(() => {
      if (options.trailing !== false) {
        lastResult = func(...args);
      }
      lastCallTime = undefined;
    }, delay);

    return lastResult;
  };
}
```

---

### 23. Implement Promise.all()
**Problem:** Create your own Promise.all() implementation.

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;

    if (promises.length === 0) {
      resolve([]);
      return;
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise).then((value) => {
        results[index] = value;
        completed++;
        if (completed === promises.length) {
          resolve(results);
        }
      }).catch(reject);
    });
  });
}
```

---

### 24. Implement Proxy Pattern
**Problem:** Create a proxy for lazy loading.

```javascript
function createLazyProxy(target, initializer) {
  return new Proxy({}, {
    get(obj, prop) {
      if (!(prop in obj)) {
        obj[prop] = initializer(prop);
      }
      return obj[prop];
    }
  });
}

// Usage
const users = createLazyProxy({}, (userId) => {
  console.log(`Loading user ${userId}`);
  return { id: userId, name: `User ${userId}` };
});

console.log(users[1]); // Loads user 1
console.log(users[1]); // Returns cached
```

---

### 25. Implement Module Loader
**Problem:** Create a simple module loader system.

```javascript
class ModuleLoader {
  constructor() {
    this.modules = {};
    this.modules.exports = {};
  }

  define(id, dependencies, factory) {
    const exports = {};
    const requires = dependencies.map(dep => this.modules[dep]);
    factory(...requires, exports);
    this.modules[id] = exports;
  }

  require(id) {
    return this.modules[id];
  }
}

// Usage
const loader = new ModuleLoader();
loader.define('math', [], (exports) => {
  exports.add = (a, b) => a + b;
});
const math = loader.require('math');
console.log(math.add(2, 3)); // 5
```

---

## 📋 Practice Tips

1. **Start with Easy Problems** - Build confidence
2. **Understand Before Coding** - Read and understand the problem
3. **Write Test Cases** - Test edge cases
4. **Optimize Later** - Get working first
5. **Explain Your Code** - Practice verbal explanation
6. **Time Yourself** - Simulate interview conditions
7. **Refactor** - Improve solutions
8. **Study Others' Solutions** - Learn different approaches

---

## 🎯 Practice Schedule

- **Week 1:** Easy problems (1-10)
- **Week 2:** Medium problems (11-20)
- **Week 3:** Hard problems (21-25) + Repeat weak areas
- **Week 4:** Mix all levels, focus on speed

---

**Happy coding! Practice makes perfect! 💪**
