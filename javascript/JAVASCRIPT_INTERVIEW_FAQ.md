# JavaScript Interview FAQ - 100+ Questions

## Table of Contents
1. [Coding Questions](#coding-questions)
2. [Verbal/Conceptual Questions](#verbal-conceptual-questions)

---

## CODING QUESTIONS

### Closures & Scope (Most Asked)

1. **Explain closures and provide a practical example**
   - Answer: Closures allow functions to access variables from their outer scope even after the outer function returns. They're used for data privacy and callback functions.
   ```javascript
   function outer() {
     let count = 0;
     return function inner() {
       count++;
       return count;
     };
   }
   const counter = outer();
   counter(); // 1
   counter(); // 2
   ```

2. **What's the difference between var, let, and const?**
   - `var`: Function-scoped, hoisted, can be redeclared
   - `let`: Block-scoped, hoisted (temporal dead zone), cannot be redeclared in same scope
   - `const`: Block-scoped, hoisted (temporal dead zone), cannot be reassigned
   ```javascript
   var a = 1;
   let b = 2;
   const c = 3;
   ```

3. **Implement a function that returns a function that increments a counter**
   ```javascript
   function makeCounter() {
     let count = 0;
     return () => ++count;
   }
   ```

4. **What is scope chain?**
   - Answer: JavaScript looks for variables in the current scope, then outer scopes, until it reaches global scope.

5. **How do closures work with loops?**
   ```javascript
   // Problem
   for (var i = 0; i < 3; i++) {
     setTimeout(() => console.log(i), 0); // 3, 3, 3
   }
   
   // Solution 1: let
   for (let i = 0; i < 3; i++) {
     setTimeout(() => console.log(i), 0); // 0, 1, 2
   }
   
   // Solution 2: IIFE closure
   for (var i = 0; i < 3; i++) {
     (function(j) {
       setTimeout(() => console.log(j), 0);
     })(i);
   }
   ```

---

### Hoisting & Temporal Dead Zone

6. **What is hoisting?**
   - Answer: Variable and function declarations are moved to the top of their scope during compilation.
   ```javascript
   console.log(x); // undefined (var hoisted, not initialized)
   var x = 5;
   
   console.log(y); // ReferenceError (let/const hoisted but not initialized - TDZ)
   let y = 10;
   
   sayHi(); // "Hi" (function declarations fully hoisted)
   function sayHi() { console.log("Hi"); }
   ```

7. **Explain temporal dead zone (TDZ)**
   - Answer: The period between entering a scope and when a let/const variable is declared where the variable cannot be accessed.

8. **What happens if you use a variable before declaring it with var?**
   - Answer: Returns `undefined` due to hoisting.

9. **How are function declarations hoisted vs function expressions?**
   - Function declarations are fully hoisted
   - Function expressions are not hoisted (only the variable is)

10. **What is the difference between hoisting var and let/const?**
    - var is hoisted and initialized to undefined
    - let/const are hoisted but not initialized (TDZ)

---

### this Keyword

11. **What does `this` refer to? (Most important)**
    - Depends on how function is called:
      - Method call: refers to the object
      - Regular function call: global object (window/global)
      - Arrow function: lexical `this` from surrounding scope
      - Constructor call: new object
    ```javascript
    const obj = {
      value: 42,
      regular: function() { console.log(this.value); },
      arrow: () => { console.log(this.value); }
    };
    obj.regular(); // 42
    obj.arrow(); // undefined
    ```

12. **Explain call(), apply(), and bind()**
    ```javascript
    function greet(greeting, name) {
      console.log(greeting + ', ' + name);
    }
    
    greet.call({}, 'Hello', 'John'); // Hello, John
    greet.apply({}, ['Hello', 'John']); // Hello, John
    const boundGreet = greet.bind({}, 'Hello');
    boundGreet('John'); // Hello, John
    ```

13. **What's the difference between arrow functions and regular functions?**
    - Arrow functions: lexical `this`, cannot be used as constructors, no `arguments` object
    - Regular functions: dynamic `this`, can be constructors, have `arguments` object

14. **How do you change the value of `this`?**
    - Answer: Using `call()`, `apply()`, or `bind()`

15. **What happens when you use `this` in an arrow function?**
    - Answer: Arrow functions don't have their own `this`; they inherit it from the enclosing scope.

16. **Implement a function that locks `this` to a specific object**
    ```javascript
    function createLocker(obj) {
      return function() {
        console.log(this);
      }.bind(obj);
    }
    ```

---

### Prototypes & Inheritance

17. **What is prototype in JavaScript?**
    - Answer: Every object has a prototype. It's an object used as a fallback for property lookups.
    ```javascript
    const parent = { greet: () => 'Hello' };
    const child = Object.create(parent);
    console.log(child.greet()); // 'Hello'
    ```

18. **Explain prototype chain**
    - Answer: When accessing a property, JavaScript looks in the object, then its prototype, then prototype's prototype, until null.

19. **What's the difference between `__proto__` and `prototype`?**
    - `prototype`: Property of functions, used to establish inheritance
    - `__proto__`: Reference to an object's actual prototype

20. **How do you implement classical inheritance in JavaScript?**
    ```javascript
    function Animal(name) {
      this.name = name;
    }
    Animal.prototype.speak = function() {
      console.log(this.name + ' speaks');
    };
    
    function Dog(name) {
      Animal.call(this, name);
    }
    Dog.prototype = Object.create(Animal.prototype);
    Dog.prototype.constructor = Dog;
    ```

21. **What is Object.create()?**
    - Answer: Creates a new object with the specified prototype.
    ```javascript
    const proto = { x: 10 };
    const obj = Object.create(proto);
    console.log(obj.x); // 10
    ```

22. **How does instanceof work?**
    - Answer: Checks if an object's prototype chain contains a constructor's prototype.
    ```javascript
    class Animal {}
    const dog = new Animal();
    console.log(dog instanceof Animal); // true
    ```

---

### Callbacks, Promises & Async/Await

23. **What is callback hell and how do you avoid it?**
    - Answer: Deeply nested callbacks. Avoid with Promises or async/await.
    ```javascript
    // Callback hell
    getData((a) => {
      getMoreData(a, (b) => {
        getMoreData(b, (c) => { /*...*/ });
      });
    });
    
    // Better: Promises/async
    async function process() {
      const a = await getData();
      const b = await getMoreData(a);
      const c = await getMoreData(b);
    }
    ```

24. **Explain Promises and their states**
    - States: pending, fulfilled, rejected
    ```javascript
    const promise = new Promise((resolve, reject) => {
      if (success) resolve(value);
      else reject(error);
    });
    ```

25. **What's the difference between Promise.all() and Promise.race()?**
    - `all()`: Waits for all promises; returns array of results or first rejection
    - `race()`: Returns first settled promise (fulfilled or rejected)
    ```javascript
    Promise.all([p1, p2, p3]); // Wait for all
    Promise.race([p1, p2, p3]); // First to finish
    ```

26. **Implement a simple Promise**
    ```javascript
    class SimplePromise {
      constructor(executor) {
        this.value = undefined;
        this.state = 'pending';
        this.handlers = [];
        executor(
          (value) => { this.state = 'fulfilled'; this.value = value; },
          (error) => { this.state = 'rejected'; this.value = error; }
        );
      }
      then(onFulfilled) {
        // Implementation
      }
    }
    ```

27. **What's the difference between Promise.then() and Promise.catch()?**
    - `then(onFulfilled, onRejected)`: Handles both cases
    - `catch(onRejected)`: Syntactic sugar for then(null, onRejected)

28. **Explain async/await**
    - Answer: Syntactic sugar for Promises. Allows writing asynchronous code synchronously.
    ```javascript
    async function fetchData() {
      try {
        const data = await fetch(url);
        return await data.json();
      } catch (error) {
        console.error(error);
      }
    }
    ```

29. **What happens if you forget await?**
    - Answer: The function returns a Promise that hasn't settled yet.

30. **How do you handle errors in async/await?**
    - Answer: Using try/catch block.

---

### Higher-Order Functions & Functional Programming

31. **What is a higher-order function?**
    - Answer: A function that takes a function as argument or returns a function.
    ```javascript
    const map = (arr, fn) => arr.map(fn);
    const add = (a) => (b) => a + b;
    ```

32. **Implement map, filter, and reduce from scratch**
    ```javascript
    function map(arr, fn) {
      const result = [];
      for (let i = 0; i < arr.length; i++) {
        result.push(fn(arr[i], i, arr));
      }
      return result;
    }
    
    function filter(arr, fn) {
      const result = [];
      for (let i = 0; i < arr.length; i++) {
        if (fn(arr[i], i, arr)) result.push(arr[i]);
      }
      return result;
    }
    
    function reduce(arr, fn, initial) {
      let acc = initial;
      for (let i = 0; i < arr.length; i++) {
        acc = fn(acc, arr[i], i, arr);
      }
      return acc;
    }
    ```

33. **What is currying?**
    - Answer: Transforming a function that takes multiple arguments into a series of functions that each take one argument.
    ```javascript
    function add(a, b, c) {
      return a + b + c;
    }
    const curriedAdd = (a) => (b) => (c) => a + b + c;
    curriedAdd(1)(2)(3); // 6
    ```

34. **Implement a function that curries another function**
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
    ```

35. **What is function composition?**
    - Answer: Combining multiple functions to create a new function.
    ```javascript
    const compose = (...fns) => (x) => fns.reduceRight((v, f) => f(v), x);
    const add1 = (x) => x + 1;
    const double = (x) => x * 2;
    const composed = compose(double, add1);
    composed(5); // (5 + 1) * 2 = 12
    ```

36. **Implement a memoization function**
    ```javascript
    function memoize(fn) {
      const cache = {};
      return function(...args) {
        const key = JSON.stringify(args);
        if (key in cache) return cache[key];
        const result = fn(...args);
        cache[key] = result;
        return result;
      };
    }
    ```

37. **What is the difference between map, flatMap, and flat?**
    - `map`: Transform elements
    - `flatMap`: Map then flatten one level
    - `flat`: Flatten array to specified depth

38. **Implement a debounce function**
    ```javascript
    function debounce(fn, delay) {
      let timeoutId;
      return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn(...args), delay);
      };
    }
    ```

39. **Implement a throttle function**
    ```javascript
    function throttle(fn, delay) {
      let lastCall = 0;
      return function(...args) {
        const now = Date.now();
        if (now - lastCall >= delay) {
          fn(...args);
          lastCall = now;
        }
      };
    }
    ```

40. **What's the difference between debounce and throttle?**
    - Debounce: Waits for inactivity before executing
    - Throttle: Executes at regular intervals

---

### Array Methods & Manipulation

41. **Explain spread operator (...) and its uses**
    ```javascript
    const arr1 = [1, 2, 3];
    const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]
    const obj = { a: 1, ...{ b: 2 } }; // { a: 1, b: 2 }
    ```

42. **What's the difference between slice and splice?**
    - `slice(start, end)`: Returns new array, doesn't mutate
    - `splice(start, deleteCount, ...items)`: Mutates original array
    ```javascript
    const arr = [1, 2, 3, 4, 5];
    arr.slice(1, 3); // [2, 3]
    arr.splice(1, 2, 'a', 'b'); // arr is now [1, 'a', 'b', 4, 5]
    ```

43. **Implement Array.prototype.filter**
    ```javascript
    Array.prototype.filter = function(callback) {
      const result = [];
      for (let i = 0; i < this.length; i++) {
        if (callback(this[i], i, this)) result.push(this[i]);
      }
      return result;
    };
    ```

44. **How do you flatten a nested array?**
    ```javascript
    const flatten = (arr) => arr.flat(Infinity);
    // Or manually:
    const flattenManual = (arr) => 
      arr.reduce((flat, el) => 
        flat.concat(Array.isArray(el) ? flattenManual(el) : el), []);
    ```

45. **Implement Array.prototype.map**
    ```javascript
    Array.prototype.map = function(callback) {
      const result = [];
      for (let i = 0; i < this.length; i++) {
        result.push(callback(this[i], i, this));
      }
      return result;
    };
    ```

46. **What's the difference between for...in and for...of?**
    - `for...in`: Iterates over enumerable keys
    - `for...of`: Iterates over iterable values
    ```javascript
    const obj = { a: 1, b: 2 };
    for (let key in obj) console.log(key); // a, b
    
    const arr = [1, 2, 3];
    for (let val of arr) console.log(val); // 1, 2, 3
    ```

47. **Implement Array.prototype.reduce**
    ```javascript
    Array.prototype.reduce = function(callback, initial) {
      let acc = initial;
      let start = initial === undefined ? 1 : 0;
      if (initial === undefined) acc = this[0];
      for (let i = start; i < this.length; i++) {
        acc = callback(acc, this[i], i, this);
      }
      return acc;
    };
    ```

48. **How do you check if an array includes a value?**
    ```javascript
    const arr = [1, 2, 3];
    arr.includes(2); // true
    arr.indexOf(2) !== -1; // true
    ```

49. **Find the maximum value in an array**
    ```javascript
    const max = (arr) => Math.max(...arr);
    // Or:
    const maxManual = (arr) => arr.reduce((a, b) => a > b ? a : b);
    ```

50. **Remove duplicates from an array**
    ```javascript
    const unique = (arr) => [...new Set(arr)];
    // Or:
    const uniqueManual = (arr) => arr.filter((v, i, a) => a.indexOf(v) === i);
    ```

---

### Object Methods & Manipulation

51. **Explain Object.keys(), Object.values(), and Object.entries()**
    ```javascript
    const obj = { a: 1, b: 2 };
    Object.keys(obj); // ['a', 'b']
    Object.values(obj); // [1, 2]
    Object.entries(obj); // [['a', 1], ['b', 2]]
    ```

52. **What's the difference between shallow copy and deep copy?**
    ```javascript
    // Shallow copy
    const shallow = { ...obj };
    const shallow2 = Object.assign({}, obj);
    
    // Deep copy
    const deep = JSON.parse(JSON.stringify(obj));
    // Better deep copy:
    function deepClone(obj) {
      if (obj === null || typeof obj !== 'object') return obj;
      if (obj instanceof Date) return new Date(obj);
      if (obj instanceof Array) return obj.map(el => deepClone(el));
      if (obj instanceof Object) {
        const cloned = {};
        for (let key in obj) {
          cloned[key] = deepClone(obj[key]);
        }
        return cloned;
      }
    }
    ```

53. **Implement Object.freeze()**
    ```javascript
    function freeze(obj) {
      Object.preventExtensions(obj);
      Object.keys(obj).forEach(key => {
        Object.defineProperty(obj, key, { writable: false });
      });
      return obj;
    }
    ```

54. **What is Object.defineProperty()?**
    - Answer: Defines or modifies a property on an object with detailed control.
    ```javascript
    const obj = {};
    Object.defineProperty(obj, 'x', {
      value: 42,
      writable: false,
      enumerable: true,
      configurable: false
    });
    ```

55. **Implement a simple object validator**
    ```javascript
    function validate(obj, schema) {
      for (let key in schema) {
        if (typeof obj[key] !== schema[key]) return false;
      }
      return true;
    }
    ```

56. **How do you merge objects?**
    ```javascript
    const obj1 = { a: 1 };
    const obj2 = { b: 2 };
    const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2 }
    // Or:
    Object.assign({}, obj1, obj2);
    ```

57. **What's the difference between Object.assign() and spread operator?**
    - Both perform shallow copy and merge
    - Spread is more modern and readable
    - Object.assign() mutates first argument

58. **Implement Object.create() polyfill**
    ```javascript
    function create(proto) {
      function F() {}
      F.prototype = proto;
      return new F();
    }
    ```

---

### Type Coercion & Equality

59. **Explain type coercion in JavaScript**
    - Answer: JavaScript automatically converts values between types.
    ```javascript
    '5' + 3; // '53' (string concatenation)
    '5' - 3; // 2 (numeric coercion)
    '5' == 5; // true (loose equality with coercion)
    '5' === 5; // false (strict equality)
    ```

60. **What's the difference between == and ===?**
    - `==`: Loose equality, performs type coercion
    - `===`: Strict equality, no type coercion
    ```javascript
    0 == false; // true
    0 === false; // false
    ```

61. **What are truthy and falsy values?**
    - Falsy: false, 0, '', null, undefined, NaN
    - Truthy: Everything else
    ```javascript
    if ('0') console.log('truthy'); // '0' is truthy!
    ```

62. **Explain the ternary operator**
    ```javascript
    condition ? valueIfTrue : valueIfFalse;
    const age = 25;
    const status = age >= 18 ? 'adult' : 'minor';
    ```

63. **What is nullish coalescing (??)?**
    - Answer: Returns right operand if left is null or undefined.
    ```javascript
    const x = null;
    const y = x ?? 'default'; // 'default'
    const a = 0;
    const b = a ?? 'default'; // 0 (0 is not nullish)
    ```

64. **Explain optional chaining (?.)**
    ```javascript
    const obj = { a: { b: { c: 42 } } };
    obj.a?.b?.c; // 42
    obj.x?.y?.z; // undefined (doesn't throw error)
    ```

65. **What does void operator do?**
    - Answer: Evaluates expression and returns undefined.
    ```javascript
    void 0; // undefined
    void(1 + 1); // undefined
    ```

---

### Classes & Constructor Functions

66. **Explain ES6 classes**
    ```javascript
    class Animal {
      constructor(name) {
        this.name = name;
      }
      
      speak() {
        console.log(this.name + ' speaks');
      }
      
      static info() {
        console.log('This is an animal class');
      }
    }
    
    const dog = new Animal('Dog');
    dog.speak(); // Dog speaks
    Animal.info(); // This is an animal class
    ```

67. **What's the difference between class and constructor function?**
    - Classes are syntactic sugar for constructor functions
    - Classes have static methods, getters, setters
    - Class methods are non-enumerable
    - Class cannot be called without `new`

68. **Explain static methods**
    ```javascript
    class MathUtil {
      static add(a, b) {
        return a + b;
      }
    }
    MathUtil.add(5, 3); // 8
    ```

69. **Explain getters and setters**
    ```javascript
    class User {
      constructor(name) {
        this._name = name;
      }
      
      get name() {
        return this._name;
      }
      
      set name(value) {
        this._name = value;
      }
    }
    ```

70. **Implement class inheritance**
    ```javascript
    class Animal {
      constructor(name) {
        this.name = name;
      }
      speak() {
        console.log(this.name + ' speaks');
      }
    }
    
    class Dog extends Animal {
      constructor(name, breed) {
        super(name);
        this.breed = breed;
      }
      speak() {
        super.speak();
        console.log('Bark!');
      }
    }
    ```

---

### Event Loop & Asynchronous JavaScript

71. **Explain the event loop**
    - Answer: JavaScript executes synchronous code, then checks callback queue and microtask queue.
    ```
    Stack → Microtask Queue (Promises) → Callback Queue (setTimeout, events)
    ```

72. **What's the difference between microtask and macrotask queue?**
    - Microtasks: Promises, MutationObserver, queueMicrotask()
    - Macrotasks: setTimeout, setInterval, I/O, UI events
    - Microtasks are processed before next macrotask

73. **Predict the output of this code**
    ```javascript
    console.log('1');
    setTimeout(() => console.log('2'), 0);
    Promise.resolve().then(() => console.log('3'));
    console.log('4');
    // Output: 1, 4, 3, 2
    ```

74. **Explain message queue and event loop in detail**
    - Answer: Call stack executes sync code. When empty, event loop checks microtask queue (Promises), then macrotask queue (setTimeout).

75. **What does setTimeout with 0 do?**
    - Answer: Defers execution to next macrotask, allowing other code to run first.

76. **How do you create a delay in JavaScript?**
    ```javascript
    function delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }
    await delay(1000);
    ```

---

### Error Handling

77. **Explain try, catch, finally**
    ```javascript
    try {
      // Code that might throw error
      riskyOperation();
    } catch (error) {
      console.error('Error:', error.message);
    } finally {
      // Always executes
      cleanup();
    }
    ```

78. **How do you create custom errors?**
    ```javascript
    class CustomError extends Error {
      constructor(message) {
        super(message);
        this.name = 'CustomError';
      }
    }
    throw new CustomError('Something went wrong');
    ```

79. **What's the difference between throw and catch?**
    - `throw`: Raises an error
    - `catch`: Handles thrown errors

80. **How do you handle errors in async functions?**
    ```javascript
    async function safeFetch(url) {
      try {
        const res = await fetch(url);
        if (!res.ok) throw new Error('Network error');
        return await res.json();
      } catch (error) {
        console.error('Fetch error:', error);
        return null;
      }
    }
    ```

---

### Regular Expressions

81. **Explain regular expressions and their flags**
    ```javascript
    const regex = /pattern/gim;
    // g: global, i: case-insensitive, m: multiline
    
    'Hello hello'.match(/hello/i); // ['Hello']
    'Hello hello'.match(/hello/gi); // ['Hello', 'hello']
    ```

82. **Implement regex matching for email validation**
    ```javascript
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    emailRegex.test('user@example.com'); // true
    ```

83. **What is the difference between exec() and match()?**
    - `exec()`: Returns array with match and groups, can iterate
    - `match()`: Returns all matches with global flag
    ```javascript
    /\d+/.exec('abc123def456'); // ['123', index: 3, ...]
    'abc123def456'.match(/\d+/g); // ['123', '456']
    ```

---

### DOM & Browser APIs

84. **Explain event delegation**
    ```javascript
    // Instead of adding listeners to each item:
    // Add one listener to parent
    document.getElementById('parent').addEventListener('click', (e) => {
      if (e.target.classList.contains('item')) {
        handleItemClick(e.target);
      }
    });
    ```

85. **What's the difference between preventDefault() and stopPropagation()?**
    - `preventDefault()`: Prevents default browser behavior
    - `stopPropagation()`: Prevents event bubbling to parent elements

86. **Explain event bubbling and capturing**
    - Capturing: Event travels down from root to target
    - Bubbling: Event travels up from target to root
    ```javascript
    element.addEventListener('click', handler, true); // Capturing
    element.addEventListener('click', handler, false); // Bubbling (default)
    ```

87. **How do you manipulate the DOM?**
    ```javascript
    document.getElementById('id').textContent = 'New text';
    document.querySelector('.class').classList.add('active');
    const newEl = document.createElement('div');
    parent.appendChild(newEl);
    ```

88. **Implement a simple observer pattern for DOM changes**
    ```javascript
    const observer = new MutationObserver((mutations) => {
      mutations.forEach(mutation => {
        console.log('DOM changed:', mutation);
      });
    });
    observer.observe(document.body, { childList: true, subtree: true });
    ```

---

### JSONP, Fetch & AJAX

89. **Explain fetch API**
    ```javascript
    fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    })
    .then(res => res.json())
    .catch(error => console.error('Error:', error));
    ```

90. **What's the difference between XMLHttpRequest and fetch?**
    - Fetch: Cleaner syntax, returns Promise, uses streams
    - XMLHttpRequest: Older, callback-based, event listeners

91. **Implement a retry mechanism for fetch**
    ```javascript
    async function fetchWithRetry(url, retries = 3) {
      try {
        const res = await fetch(url);
        if (!res.ok) throw new Error('Failed');
        return await res.json();
      } catch (error) {
        if (retries > 0) {
          await new Promise(resolve => setTimeout(resolve, 1000));
          return fetchWithRetry(url, retries - 1);
        }
        throw error;
      }
    }
    ```

---

### Destructuring & Rest Parameters

92. **Explain array destructuring**
    ```javascript
    const [a, b, c] = [1, 2, 3];
    const [x, , z] = [1, 2, 3]; // Skip element
    const [first, ...rest] = [1, 2, 3, 4]; // Rest operator
    ```

93. **Explain object destructuring**
    ```javascript
    const { name, age } = { name: 'John', age: 30 };
    const { name: userName } = { name: 'John' }; // Rename
    const { a, b = 10 } = { a: 5 }; // Default value
    ```

94. **What are rest parameters and how do they differ from arguments?**
    ```javascript
    function sum(...numbers) {
      return numbers.reduce((a, b) => a + b);
    }
    sum(1, 2, 3); // 6
    
    // arguments is array-like, not true array
    function oldSum() {
      console.log(arguments); // [1, 2, 3]
    }
    ```

---

### JSON

95. **Explain JSON.stringify() and JSON.parse()**
    ```javascript
    const obj = { a: 1, b: 2 };
    const str = JSON.stringify(obj); // '{"a":1,"b":2}'
    const parsed = JSON.parse(str); // { a: 1, b: 2 }
    
    // With replacer and space
    JSON.stringify(obj, null, 2); // Pretty print
    ```

96. **How do you handle circular references in JSON?**
    ```javascript
    const seen = new WeakSet();
    const json = JSON.stringify(obj, (key, value) => {
      if (typeof value === 'object' && value !== null) {
        if (seen.has(value)) return undefined;
        seen.add(value);
      }
      return value;
    });
    ```

---

### Math & Number Operations

97. **Explain common Math methods**
    ```javascript
    Math.round(4.5); // 4 or 5
    Math.floor(4.9); // 4
    Math.ceil(4.1); // 5
    Math.abs(-5); // 5
    Math.pow(2, 3); // 8
    Math.sqrt(16); // 4
    Math.max(1, 2, 3); // 3
    Math.min(1, 2, 3); // 1
    Math.random(); // 0 to 1
    ```

98. **Generate a random number in a range**
    ```javascript
    function random(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    }
    random(1, 10); // 1 to 10
    ```

99. **What's the difference between Number.isNaN() and isNaN()?**
    ```javascript
    isNaN('hello'); // true (coerces to NaN)
    Number.isNaN('hello'); // false (doesn't coerce)
    Number.isNaN(NaN); // true
    ```

100. **Explain Number methods**
    ```javascript
    Number.isInteger(5); // true
    Number.isFinite(Infinity); // false
    Number.parseFloat('3.14'); // 3.14
    Number.parseInt('123abc'); // 123
    5.toFixed(2); // '5.00'
    ```

---

## VERBAL/CONCEPTUAL QUESTIONS

### Fundamental Concepts (Most Asked)

1. **What is JavaScript and what can it do?**
   - Answer: Lightweight, interpreted programming language that runs in browsers. Can manipulate DOM, handle events, make API calls, store data, create interactive user experiences.

2. **What are the primitive data types in JavaScript?**
   - Answer: string, number, boolean, null, undefined, symbol, bigint
   ```javascript
   typeof 'hello'; // 'string'
   typeof 42; // 'number'
   typeof true; // 'boolean'
   typeof undefined; // 'undefined'
   typeof Symbol('id'); // 'symbol'
   typeof 123n; // 'bigint'
   ```

3. **What is the difference between primitive and object types?**
   - Primitives: Immutable, passed by value, stored on stack
   - Objects: Mutable, passed by reference, stored on heap

4. **Explain null vs undefined**
   - `null`: Intentional absence of value (developer sets it)
   - `undefined`: Variable declared but not assigned, missing function parameter

5. **What is NaN and how do you check for it?**
   ```javascript
   typeof NaN; // 'number'
   NaN === NaN; // false
   Number.isNaN(NaN); // true
   Object.is(NaN, NaN); // true
   ```

6. **What is the difference between == and ===?**
   - == performs type coercion
   - === does strict comparison without coercion

7. **Explain the concept of objects in JavaScript**
   - Answer: Objects are collections of key-value pairs. Everything (except primitives) is an object.

8. **What are arrays and how do they differ from objects?**
   - Arrays: Ordered collections with numeric indices
   - Objects: Unordered collections with string keys

9. **What is a function and how is it different from a method?**
   - Function: Standalone code block
   - Method: Function attached to an object

10. **What is the window object?**
    - Answer: Global object in browsers containing all global variables and functions.

---

### Object-Oriented Programming

11. **Explain the four pillars of OOP**
    - Encapsulation: Bundling data and methods
    - Abstraction: Hiding complexity
    - Inheritance: Deriving from base classes
    - Polymorphism: Multiple forms, method overriding

12. **What is encapsulation?**
    - Answer: Bundling data (properties) and behavior (methods) together and hiding internal details.
    ```javascript
    class BankAccount {
      #balance = 0; // Private field
      deposit(amount) {
        this.#balance += amount;
      }
      getBalance() {
        return this.#balance;
      }
    }
    ```

13. **What is the difference between public, private, and protected?**
    - Public: Accessible everywhere
    - Private: Accessible only within the class
    - Protected: Accessible within class and subclasses (not standard in JS)

14. **Explain polymorphism**
    - Answer: Objects of different types respond to same method differently.
    ```javascript
    class Dog { speak() { console.log('Woof'); } }
    class Cat { speak() { console.log('Meow'); } }
    ```

15. **What is method overriding?**
    - Answer: Subclass providing its own implementation of parent's method.

---

### Functional Programming

16. **What is functional programming?**
    - Answer: Programming paradigm treating computation as evaluation of functions, avoiding mutable data.

17. **What are pure functions?**
    - Answer: Functions that produce same output for same input and have no side effects.
    ```javascript
    // Pure
    const add = (a, b) => a + b;
    
    // Impure
    let counter = 0;
    const increment = () => ++counter;
    ```

18. **What are side effects?**
    - Answer: When function modifies external state (global variables, I/O operations, DOM manipulation).

19. **What is immutability?**
    - Answer: Data cannot be changed after creation. Create new data instead of modifying.
    ```javascript
    const user = { name: 'John' };
    const updatedUser = { ...user, age: 30 }; // New object
    ```

20. **What is the difference between imperative and declarative?**
    - Imperative: How to do something (step-by-step)
    - Declarative: What to do (desired result)
    ```javascript
    // Imperative
    for (let i = 0; i < arr.length; i++) {
      console.log(arr[i]);
    }
    
    // Declarative
    arr.forEach(item => console.log(item));
    ```

---

### Event-Driven Architecture

21. **Explain pub/sub (publish/subscribe) pattern**
    - Answer: Objects communicate through events. Publishers emit events, subscribers listen.
    ```javascript
    class EventEmitter {
      constructor() {
        this.events = {};
      }
      subscribe(event, handler) {
        if (!this.events[event]) this.events[event] = [];
        this.events[event].push(handler);
      }
      publish(event, data) {
        if (this.events[event]) {
          this.events[event].forEach(handler => handler(data));
        }
      }
    }
    ```

22. **What is event-driven programming?**
    - Answer: Program flow determined by events (user actions, system events).

23. **Explain the observer pattern**
    - Answer: One object notifies multiple observers of state changes.

---

### Design Patterns

24. **What is the singleton pattern?**
    - Answer: Ensures only one instance of a class exists.
    ```javascript
    class Singleton {
      static instance;
      constructor() {
        if (Singleton.instance) return Singleton.instance;
        Singleton.instance = this;
      }
    }
    ```

25. **Explain the factory pattern**
    - Answer: Creates objects without specifying exact classes.
    ```javascript
    function createUser(type) {
      if (type === 'admin') return new Admin();
      if (type === 'user') return new User();
    }
    ```

26. **What is the module pattern?**
    - Answer: Encapsulates code to avoid global namespace pollution.
    ```javascript
    const module = (() => {
      const private = {};
      return {
        public: () => {}
      };
    })();
    ```

27. **Explain the decorator pattern**
    - Answer: Adds functionality to objects dynamically without altering structure.

28. **What is the builder pattern?**
    - Answer: Constructs complex objects step by step.

---

### Memory & Performance

29. **What is garbage collection?**
    - Answer: Automatic memory management removing unreachable objects.

30. **What is a memory leak and how does it occur?**
    - Answer: Memory that should be freed but isn't, causing memory to accumulate.
    ```javascript
    // Memory leak: event listener never removed
    element.addEventListener('click', handler);
    // Should be: element.removeEventListener('click', handler);
    ```

31. **How do you prevent memory leaks?**
    - Remove event listeners
    - Clear timers and intervals
    - Clean up references
    - Use weak references when appropriate

32. **What are weak references in JavaScript?**
    - Answer: References that don't prevent garbage collection.
    ```javascript
    const weakMap = new WeakMap();
    const obj = { id: 1 };
    weakMap.set(obj, 'value');
    ```

33. **Explain the concept of scope and how it affects memory**
    - Answer: Scope determines variable lifetime. Variables are garbage collected when scope is exited.

---

### Optimization & Best Practices

34. **What are the common performance bottlenecks?**
    - DOM manipulation, Reflow/Repaint, Inefficient loops, Memory leaks, Network requests

35. **What is reflow and repaint?**
    - Reflow: Recalculating element positions and sizes
    - Repaint: Redrawing elements on screen
    - Both are expensive; minimize them

36. **How do you optimize JavaScript performance?**
    - Minimize reflows/repaints
    - Debounce/throttle events
    - Use event delegation
    - Lazy load resources
    - Code splitting

37. **What is tree-shaking and how does it work?**
    - Answer: Process to remove unused code from bundles in ES6 modules.

38. **What is code splitting?**
    - Answer: Dividing code into chunks loaded on-demand, reducing initial bundle size.

39. **Explain lazy loading**
    - Answer: Deferring resource loading until needed.
    ```javascript
    const image = new Image();
    image.src = imageUrl; // Loads when added to DOM
    ```

40. **What is minification and why is it important?**
    - Answer: Reducing code size by removing unnecessary characters, improving load time.

---

### Scope & Execution Context

41. **What is execution context?**
    - Answer: Environment where JavaScript code is executed, containing variables, functions, and `this`.

42. **Explain the three phases of execution context creation**
    - Creation phase: Variables declared, functions hoisted, `this` bound
    - Execution phase: Code executed, variables assigned
    - Completion phase: Cleanup

43. **What is the call stack?**
    - Answer: Data structure tracking function execution. Last-in-first-out (LIFO).

44. **What is stack overflow?**
    - Answer: Call stack exceeded maximum size, usually from infinite recursion.

---

### Browser APIs & Web Technologies

45. **What is localStorage and sessionStorage?**
    - localStorage: Persists across sessions
    - sessionStorage: Cleared when tab closes
    ```javascript
    localStorage.setItem('key', 'value');
    localStorage.getItem('key');
    sessionStorage.setItem('key', 'value');
    ```

46. **What is the difference between localStorage and cookies?**
    - localStorage: Larger capacity (5-10MB), not sent with HTTP requests
    - Cookies: Smaller capacity (4KB), sent with every request

47. **What are web workers?**
    - Answer: Background threads for heavy computation without blocking main thread.

48. **Explain IndexedDB**
    - Answer: Browser database for storing larger amounts of structured data.

49. **What is ServiceWorker?**
    - Answer: Script running separately from main thread, enabling offline functionality.

50. **Explain the Geolocation API**
    - Answer: Allows access to user's geographic location.

---

### Module Systems

51. **Explain CommonJS**
    - Answer: Node.js module system using require() and module.exports.
    ```javascript
    const module = require('./module.js');
    module.exports = { func: () => {} };
    ```

52. **Explain ES6 modules**
    - Answer: Standard module system using import and export.
    ```javascript
    import { func } from './module.js';
    export const func = () => {};
    ```

53. **What's the difference between CommonJS and ES6 modules?**
    - CommonJS: Synchronous, dynamic
    - ES6: Asynchronous, static (for better optimization)

54. **What is AMD (Asynchronous Module Definition)?**
    - Answer: Older module system for browser-based modules using define().

---

### Package Management

55. **What is npm and why is it important?**
    - Answer: Node Package Manager for managing JavaScript packages and dependencies.

56. **What are dependencies vs devDependencies?**
    - dependencies: Required for production
    - devDependencies: Required only for development

57. **What is package-lock.json?**
    - Answer: Locks exact versions of dependencies for reproducible installs.

58. **What is semantic versioning?**
    - Answer: Version format: MAJOR.MINOR.PATCH
    - MAJOR: Breaking changes
    - MINOR: New features
    - PATCH: Bug fixes

---

### Testing

59. **What are the different types of testing?**
    - Unit testing: Individual functions
    - Integration testing: Multiple modules together
    - End-to-end testing: Complete user flows

60. **What is mocking?**
    - Answer: Replacing real objects with test doubles for testing.

61. **What is test coverage?**
    - Answer: Percentage of code executed by tests.

62. **Name popular testing frameworks**
    - Jest, Mocha, Jasmine, Cypress, Vitest

---

### Security

63. **What is XSS (Cross-Site Scripting)?**
    - Answer: Injecting malicious scripts into web pages viewed by users.
    ```javascript
    // Vulnerable
    element.innerHTML = userInput;
    
    // Safe
    element.textContent = userInput;
    ```

64. **What is CSRF (Cross-Site Request Forgery)?**
    - Answer: Tricking users into making unwanted requests on their behalf.

65. **What is the same-origin policy?**
    - Answer: Security feature restricting scripts from accessing content from other origins.

66. **What is CORS?**
    - Answer: Mechanism allowing restricted resources to be accessed from different origins.

67. **How do you prevent SQL injection?**
    - Answer: Use parameterized queries, validate input, use ORMs.

68. **What is content security policy (CSP)?**
    - Answer: Security standard preventing certain types of attacks.

---

### API & HTTP

69. **Explain REST API principles**
    - Answer: Architectural style for web services using HTTP methods (GET, POST, PUT, DELETE) on resources.

70. **What are HTTP methods?**
    - GET: Retrieve data
    - POST: Create data
    - PUT: Update entire resource
    - PATCH: Update partial resource
    - DELETE: Delete data

71. **What are HTTP status codes?**
    - 2xx: Success (200, 201, 204)
    - 3xx: Redirect (301, 302, 304)
    - 4xx: Client error (400, 401, 403, 404)
    - 5xx: Server error (500, 502, 503)

72. **What is the difference between PUT and PATCH?**
    - PUT: Replaces entire resource
    - PATCH: Modifies specific fields

73. **What is GraphQL?**
    - Answer: Query language for APIs allowing clients to request exactly needed data.

74. **What is the difference between REST and GraphQL?**
    - REST: Multiple endpoints, fixed data structure
    - GraphQL: Single endpoint, flexible queries

75. **What is caching and why is it important?**
    - Answer: Storing copies of resources to reduce load and improve performance.

---

### Debugging & Tools

76. **What is console.log() and how do you use it effectively?**
    ```javascript
    console.log('message'); // Basic
    console.table(data); // Tabular
    console.time('label'); // Timing
    console.group('Group'); // Grouping
    ```

77. **What are browser DevTools?**
    - Answer: Built-in development tools for inspecting, debugging, and profiling web pages.

78. **How do you debug JavaScript?**
    - Use console methods
    - Set breakpoints in DevTools
    - Use debugger statement
    - Use browser DevTools profiler

79. **What is the debugger statement?**
    ```javascript
    debugger; // Pauses execution when DevTools open
    ```

80. **How do you find and fix performance issues?**
    - Profile with DevTools
    - Check network tab
    - Analyze flame graphs
    - Identify bottlenecks

---

### Naming & Conventions

81. **What are JavaScript naming conventions?**
    - Variables: camelCase
    - Constants: UPPER_SNAKE_CASE
    - Classes: PascalCase
    - Private methods: _methodName()

82. **Why is code readability important?**
    - Answer: Makes code easier to maintain, debug, and collaborate on.

---

### Documentation & Comments

83. **What is JSDoc?**
    - Answer: Standard for documenting JavaScript functions and classes.
    ```javascript
    /**
     * Adds two numbers
     * @param {number} a - First number
     * @param {number} b - Second number
     * @returns {number} Sum of a and b
     */
    function add(a, b) {
      return a + b;
    }
    ```

84. **When should you write comments?**
    - Explain why, not what
    - Document complex logic
    - Note edge cases

---

### Version Control

85. **What is Git and why is it important?**
    - Answer: Distributed version control system for tracking code changes.

86. **What are branches and why do you use them?**
    - Answer: Separate lines of development, allowing parallel work without conflicts.

87. **What is a merge conflict and how do you resolve it?**
    - Answer: When changes in different branches affect same lines. Manually edit and choose changes.

88. **What is rebase and how does it differ from merge?**
    - Merge: Integrates two branches, creates merge commit
    - Rebase: Rewrites history, replays commits

---

### Career & Professional Development

89. **What qualities make a good JavaScript developer?**
    - Problem-solving skills
    - Deep understanding of fundamentals
    - Code quality and readability
    - Continuous learning
    - Debugging ability

90. **How do you stay updated with JavaScript changes?**
    - Follow blogs and newsletters
    - Read documentation
    - Participate in communities
    - Build projects

91. **What is the importance of code reviews?**
    - Knowledge sharing, catching bugs, maintaining standards, improving code quality

92. **How do you approach learning a new framework?**
    - Read official documentation
    - Build small projects
    - Understand core concepts
    - Practice

93. **What is technical debt?**
    - Answer: Cost of quick fixes and shortcuts that need to be fixed later.

---

### Real-World Scenarios

94. **How would you implement authentication in a web application?**
    - Answer: Token-based (JWT), session-based, OAuth2, multi-factor authentication

95. **How do you handle real-time updates in a web application?**
    - WebSockets, Server-Sent Events (SSE), Polling

96. **How would you optimize a slow website?**
    - Minimize bundle size
    - Lazy load images
    - Cache resources
    - Optimize database queries
    - Use CDN

97. **How do you handle state management in a large application?**
    - Redux, MobX, Context API, Zustand, Recoil

98. **How would you implement error tracking and monitoring?**
    - Sentry, Rollbar, LogRocket, custom error handlers

99. **How do you ensure code quality in a team?**
    - Code reviews, linting, formatting, testing, documentation

100. **What would you do if a critical bug is found in production?**
     - Identify root cause, create hotfix, deploy immediately, document, prevent recurrence

---

## Quick Reference: Most Common Interview Topics

**Top 20 Most Asked Topics:**
1. Closures
2. this keyword
3. Prototypes & Inheritance
4. Promises & async/await
5. Event Loop
6. Hoisting
7. Scope & Execution Context
8. Array Methods (map, filter, reduce)
9. Object Methods & Manipulation
10. Type Coercion & Equality
11. Classes & Constructors
12. Higher-Order Functions
13. Destructuring
14. Spread Operator
15. Error Handling
16. Function vs Method
17. Callback Functions
18. Arrow Functions
19. Template Literals
20. Module Systems

**Good Luck with Your Interview! 🚀**
