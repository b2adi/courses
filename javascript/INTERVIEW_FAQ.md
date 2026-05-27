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

---

(Continued with remaining JavaScript content...)
