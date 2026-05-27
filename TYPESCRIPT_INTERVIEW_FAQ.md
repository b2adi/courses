# TypeScript Interview FAQ - 100+ Questions with Detailed Explanations

## Table of Contents
1. [Coding Questions](#coding-questions)
2. [Verbal/Conceptual Questions](#verbal-conceptual-questions)

---

## CODING QUESTIONS

### Type System Basics (Most Asked)

1. **What are the basic types in TypeScript?**

**Explanation:** TypeScript extends JavaScript with a type system. Basic types are the primitive types you can assign to variables. These include string, number, boolean, and special types like any, unknown, and never. Understanding these is fundamental because every variable in TypeScript has a type, either explicitly declared or inferred.

```typescript
let num: number = 42;        // Numeric type - any number
let str: string = "hello";   // String type - text values
let bool: boolean = true;    // Boolean type - true/false
let arr: number[] = [1, 2, 3]; // Array of numbers
let arr2: Array<string> = ["a", "b"]; // Generic array syntax
let any_type: any = "anything"; // Any type - bypasses type checking
let unknown_type: unknown = "unknown"; // Unknown type - safe any
let never_type: never = (() => { throw new Error(); })(); // Never type - unreachable
```

---

2. **What is the difference between any and unknown?**

**Explanation:** Both any and unknown can hold any value, but unknown is safer. With `any`, you can call any method without checking. With `unknown`, you must perform type checking first. `any` bypasses type checking entirely (dangerous), while `unknown` forces you to narrow the type before use. This makes `unknown` the preferred choice for truly unknown types.

```typescript
// any - unsafe, no type checking required
let a: any = "string";
a.toUpperCase(); // OK (wrong!)
a.nonexistent(); // OK (very wrong!)

// unknown - safe, requires type checking
let u: unknown = "string";
if (typeof u === 'string') {
  u.toUpperCase(); // OK - type narrowed to string
}
// u.toUpperCase(); // Error - still unknown

// Why it matters: 
// Any lets bugs slip through - unknown prevents them
// Use unknown for external data, APIs, user input
```

---

3. **Explain type annotations**

**Explanation:** Type annotations are explicit declarations of what type a variable or parameter should have. They use the `: Type` syntax. Annotations tell TypeScript (and other developers) what type is expected. Without annotations, TypeScript infers types, but annotations make intent explicit and catch mistakes earlier.

```typescript
// Function parameter annotations - specify what types function accepts
function add(a: number, b: number): number {
  return a + b;
}

// Variable annotations - specify variable type
const name: string = "John";
const count: number = 42;
const items: string[] = ["a", "b"];

// Why annotations matter:
// 1. Self-documenting - readers know what types to pass
// 2. Compile-time errors - catches type mismatches early
// 3. IDE autocomplete - better suggestions
// 4. Prevents silent bugs - type confusion caught immediately
```

---

4. **What is type inference?**

**Explanation:** Type inference means TypeScript automatically determines the type of a variable based on its value. You don't always need explicit annotations—TypeScript is smart enough to figure types out. This reduces boilerplate while maintaining type safety. However, sometimes explicit annotations are clearer for readability.

```typescript
let message = "hello"; // Inferred as string type
let count = 42;        // Inferred as number type
let arr = [1, 2, 3];   // Inferred as number[] type
let obj = { name: "John" }; // Inferred as { name: string }

// TypeScript inferred these without explicit annotations
// If you hover in an IDE, you'd see the inferred types

// Why inference is useful:
// Reduces code verbosity, maintains type safety
// But sometimes explicit is better for clarity

// Example where explicit is better:
const user: Partial<User> = {}; // Clear intent vs const user = {};
```

---

5. **Explain union types**

**Explanation:** Union types allow a variable to be one of several types. They're useful when a value can be multiple types. The `|` operator combines types. When using union types, you must handle all possible cases or narrow to a specific type before using type-specific operations.

```typescript
// Union types - variable can be one of these types
type Status = "success" | "error" | "pending";
let status: Status = "success"; // OK
// status = "invalid"; // Error - not in union

let id: string | number;
id = "123"; // OK - is string
id = 123;   // OK - is number
// id = true; // Error - boolean not in union

// Why unions matter:
// 1. Flexibility - handles multiple types
// 2. Type safety - limits what's allowed
// 3. Intent - shows all valid possibilities

// Using unions requires type narrowing:
function process(value: string | number) {
  if (typeof value === 'string') {
    console.log(value.toUpperCase()); // Now definitely string
  } else {
    console.log(value.toFixed(2)); // Now definitely number
  }
}
```

---

6. **Explain literal types**

**Explanation:** Literal types are types that represent specific, exact values rather than any value of that type. For example, `"success"` is more specific than `string`. Literal types are useful for restricted sets of values. You can combine them with unions to create exhaustive lists of valid values.

```typescript
// Single literal types
type SuccessStatus = "success"; // Only "success" allowed
type ErrorStatus = "error";     // Only "error" allowed

// Union of literals (most common pattern)
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Direction = "up" | "down" | "left" | "right";

let method: HttpMethod = "GET"; // OK
// method = "PATCH"; // Error - not in literal union

// Numeric literals
type SmallNumbers = 1 | 2 | 3 | 4 | 5;
let num: SmallNumbers = 3; // OK

// Why literal types are useful:
// 1. Exhaustiveness - compiler ensures all cases handled
// 2. Autocomplete - IDE suggests exact values
// 3. Documentation - shows exactly what's valid
// 4. Prevents typos - "succes" would be caught as error
```

---

7. **What is type narrowing?**

**Explanation:** Type narrowing is the process of refining a variable's type from a broader type to a more specific type. When you have a union type, narrowing restricts what operations you can perform. Common narrowing techniques are typeof checks, instanceof checks, truthiness checks, and type guards. Narrowing is essential when working with union types.

```typescript
// Type narrowing with typeof
function process(value: string | number): void {
  // typeof guard - narrows to specific type
  if (typeof value === 'string') {
    console.log(value.toUpperCase()); // value is string here
  } else {
    console.log(value.toFixed(2)); // value is number here
  }
}

// Type narrowing with instanceof
class Dog {}
class Cat {}

function speak(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark(); // animal is Dog here
  } else {
    animal.meow(); // animal is Cat here
  }
}

// Why narrowing is important:
// Union types are flexible but unsafe without narrowing
// Narrowing tells TypeScript (and you) what operations are safe
```

---

### Interfaces & Types

8. **What is an interface?**

**Explanation:** Interfaces define the structure/contract that objects must follow. They specify what properties and methods an object should have. Interfaces are primarily for object types and are better for defining contracts. They use structural typing—TypeScript checks if the structure matches, not the name.

```typescript
// Interface defines a contract
interface User {
  id: number;           // Required property
  name: string;         // Required property
  email?: string;       // Optional property (may or may not exist)
  readonly created: Date; // Readonly property (cannot be changed)
}

const user: User = {
  id: 1,
  name: "John",
  created: new Date()
  // email is optional, so OK to omit
};

// Why interfaces are useful:
// 1. Clear contract - defines required structure
// 2. Reusable - can implement or extend
// 3. Optional properties - flexibility
// 4. Readonly - prevent accidental modifications
```

---

9. **What's the difference between interface and type?**

**Explanation:** Both interfaces and types define shapes, but they're different. Interfaces are better for object contracts and support declaration merging (combining multiple declarations). Types are more flexible and can represent anything (unions, primitives, tuples). For object contracts, prefer interfaces; for unions or complex types, use type.

```typescript
// Interface - better for objects
interface Animal {
  name: string;
  speak(): void;
}

// Can extend other interfaces
interface Dog extends Animal {
  breed: string;
}

// Declaration merging - can add properties to interface later
interface Animal {
  age: number; // Gets merged into original Animal
}

// Type - can be anything
type AnimalType = {
  name: string;
  speak(): void;
};

// Use intersection with types
type DogType = AnimalType & {
  breed: string;
};

// Types cannot declare merge
// But types can represent unions
type Pet = Dog | Cat | Bird;

// Key differences:
// Interface: Object contracts, extensible, declaration merging
// Type: Anything, unions, primitives, cannot merge
```

---

10. **Explain interface declaration merging**

**Explanation:** Declaration merging means combining multiple interface declarations with the same name. All properties get combined into one interface. This is unique to interfaces (types cannot merge). It's useful for extending types across files or allowing others to extend your library's types.

```typescript
// First declaration
interface User {
  id: number;
}

// Second declaration - gets merged
interface User {
  name: string;
}

// Result is equivalent to:
// interface User {
//   id: number;
//   name: string;
// }

// This is useful for:
// 1. Extending library types from separate files
// 2. Plugin systems where plugins add properties
// 3. Global type augmentation

// Example - extending global Window object
interface Window {
  myCustomProperty: string;
}

// Now Window includes myCustomProperty everywhere
```

---

### Generics (Very Important)

11. **What are generics?**

**Explanation:** Generics allow functions, classes, and types to work with multiple types while maintaining type safety. A generic is a placeholder for a type that gets specified later. Instead of writing the same function for every type, you write it once with a generic parameter. This provides flexibility while keeping type safety.

```typescript
// Generic function - T is a placeholder for any type
function identity<T>(arg: T): T {
  return arg;
}

// When called, T gets a specific type
identity<string>("hello");  // T = string
identity<number>(42);       // T = number
identity("hello");          // T inferred as string

// Generic with multiple parameters
function combine<T, U>(a: T, b: U): [T, U] {
  return [a, b];
}

combine<string, number>("hello", 42); // [string, number]

// Why generics are powerful:
// 1. Reuse code for multiple types
// 2. Maintain type safety
// 3. Better than 'any' - preserves types
// 4. Enable powerful abstractions
```

---

12. **Explain generic constraints**

**Explanation:** Generic constraints restrict what types can be used as the generic parameter. Instead of accepting any type, you specify requirements the type must meet. Common constraints require types to have specific properties. Constraints let you assume the parameter has certain properties, enabling safer code.

```typescript
// Constraint: T must have a length property
function getLength<T extends { length: number }>(arg: T): number {
  return arg.length; // Safe - we know length exists
}

getLength("hello");  // OK - strings have length
getLength([1, 2, 3]); // OK - arrays have length
// getLength(123); // Error - numbers don't have length

// Constraint: T must be a key of an object
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "John", age: 30 };
getProperty(user, "name");  // OK - "name" is valid key
// getProperty(user, "email"); // Error - "email" not in user

// Why constraints are useful:
// 1. Prevent invalid types
// 2. Enable assumptions about the type
// 3. Provide better error messages
// 4. Enable powerful patterns
```

---

13. **What are generic classes?**

**Explanation:** Generic classes are classes that work with multiple types. The type is specified when creating an instance. This is useful for containers like lists, queues, or other generic data structures. The generic parameter can be used throughout the class methods and properties.

```typescript
// Generic class - works with any type
class Box<T> {
  private value: T;
  
  constructor(value: T) {
    this.value = value;
  }
  
  getValue(): T {
    return this.value;
  }
  
  setValue(value: T): void {
    this.value = value;
  }
}

// Specify type when creating instance
const stringBox = new Box<string>("hello");
const numBox = new Box<number>(42);

stringBox.getValue(); // Returns string
numBox.getValue();    // Returns number

// Generic with constraints
class NumberBox<T extends number> {
  constructor(private value: T) {}
  double(): T {
    return (this.value * 2) as T;
  }
}

// Why generic classes are useful:
// 1. Reusable data structures
// 2. Type-safe collections
// 3. Works with any type
```

---

14. **Explain keyof operator**

**Explanation:** The `keyof` operator produces a type representing the keys of an object type as a union. It's powerful for ensuring you're accessing valid properties. When used in generics, it ensures you only access properties that actually exist on the object.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// keyof extracts all keys as union type
type UserKeys = keyof User; // "id" | "name" | "email"

// Use in function to ensure valid property access
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]; // Safe - K is definitely a valid key of T
}

const user: User = { id: 1, name: "John", email: "john@example.com" };
getProperty(user, "name");  // OK - "name" is valid
// getProperty(user, "age"); // Error - "age" not in keyof User

// Why keyof is powerful:
// 1. Type-safe property access
// 2. Prevents accessing invalid properties
// 3. Enables powerful generic patterns
// 4. Automated from object structure
```

---

15. **What is typeof operator in TypeScript?**

**Explanation:** The `typeof` operator in TypeScript creates a type from a value. Unlike JavaScript's typeof (which checks runtime type), TypeScript's typeof extracts the compile-time type. This is useful for reusing types from existing variables and creating types from complex expressions.

```typescript
const user = { name: "John", age: 30 };
type UserType = typeof user; // { name: string; age: number }

const num = 42;
type NumType = typeof num; // number

// Useful for creating types from actual values
const config = {
  host: "localhost",
  port: 3000,
  debug: true
};
type ConfigType = typeof config;
// Result: { host: string; port: number; debug: boolean }

// Why typeof is useful:
// 1. Create types from values
// 2. Don't repeat type definitions
// 3. Refactor safely - type follows value
// 4. Works with any expression
```

---

16. **Explain conditional types**

**Explanation:** Conditional types use the ternary operator to define types conditionally. They enable conditional logic at the type level. A conditional type checks if one type extends another, then returns different types based on the result. Powerful for creating sophisticated type transformations.

```typescript
// Simple conditional type
type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">;  // true
type B = IsString<42>;       // false

// Practical example - unwrap array
type Flatten<T> = T extends Array<infer U> ? U : T;

type Str = Flatten<string[]>;  // string
type Num = Flatten<number>;    // number

// Complex example - extract promise type
type Awaited<T> = T extends Promise<infer U> ? U : T;

type A = Awaited<Promise<string>>; // string
type B = Awaited<number>;          // number

// Why conditional types are powerful:
// 1. Type-level logic
// 2. Extract nested types (infer)
// 3. Create smart type transformations
// 4. Enable advanced patterns
```

---

17. **What are mapped types?**

**Explanation:** Mapped types iterate over all properties of a type and create new properties. They use `in` to iterate over union types (often from keyof). Mapped types enable transforming one type into another systematically. Common mappings include making properties optional, readonly, or changing their types.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// Make all properties readonly
type ReadonlyUser = {
  readonly [K in keyof User]: User[K];
};

// Make all properties optional
type PartialUser = {
  [K in keyof User]?: User[K];
};

// Make all properties function types
type Getters<T> = {
  [K in keyof T]: () => T[K];
};

type UserGetters = Getters<User>;
// Result: { id: () => number; name: () => string; email: () => string }

// Why mapped types are useful:
// 1. Transform types systematically
// 2. Avoid code duplication
// 3. Create variants (readonly, optional, etc)
// 4. Enable powerful abstractions
```

---

18. **Explain infer keyword**

**Explanation:** The `infer` keyword enables extracting types from within a conditional type. Instead of specifying what to extract, `infer` declares a variable that TypeScript fills in. This is crucial for extracting nested types like function parameters or return types.

```typescript
// Extract function return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getString() {
  return "hello";
}

type Result = ReturnType<typeof getString>; // string

// Extract function parameters
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

type Params = Parameters<(a: string, b: number) => void>;
// Result: [string, number]

// Extract promise type
type Awaited<T> = T extends Promise<infer U> ? U : T;

type PromiseString = Awaited<Promise<string>>; // string

// Why infer is powerful:
// 1. Extract nested types
// 2. Enable type inference within conditionals
// 3. Works with any complex type structure
// 4. Foundation for advanced patterns
```

---

19. **What is Record type?**

**Explanation:** `Record<K, V>` creates an object type with properties of type K and values of type V. It's useful when you have a fixed set of keys and want all to map to the same value type. Record ensures you provide all required keys.

```typescript
type Color = "red" | "green" | "blue";

// Record creates object with all Color values mapped to strings
const colorMap: Record<Color, string> = {
  red: "#FF0000",
  green: "#00FF00",
  blue: "#0000FF"
};

// If you forget a key, TypeScript errors:
// Missing property: 'blue'

// Common use cases
type RolePermissions = Record<"admin" | "user" | "guest", string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// Why Record is useful:
// 1. Ensure all keys are provided
// 2. Type-safe configuration maps
// 3. Prevents missing properties
```

---

20. **Explain Pick and Omit**

**Explanation:** `Pick<T, K>` selects specific properties from a type, while `Omit<T, K>` excludes them. Both are utility types that create new types by transforming existing ones. Pick is useful for exposing only certain properties; Omit is useful for hiding sensitive properties.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Pick - select only specific properties
type UserPreview = Pick<User, "id" | "name">;
// Result: { id: number; name: string }

// Omit - exclude specific properties
type UserPublic = Omit<User, "password">;
// Result: { id: number; name: string; email: string }

// Practical examples
type UserProfile = Pick<User, "id" | "name" | "email">;
type UserUpdate = Omit<User, "id" | "password">;

// Why Pick and Omit are useful:
// 1. Create variants of types
// 2. Hide sensitive properties
// 3. API responses vs internal representation
// 4. Prevent accidental data exposure
```

---

### Classes & OOP

21. **What is a class in TypeScript?**

**Explanation:** Classes are blueprints for creating objects with properties and methods. TypeScript enhances JavaScript classes with access modifiers (public, private, protected), type annotations, and abstract classes. Classes provide structure and enable object-oriented programming patterns.

```typescript
class User {
  id: number;
  name: string;
  
  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }
  
  greet(): string {
    return `Hello, ${this.name}`;
  }
}

const user = new User(1, "John");
console.log(user.greet()); // "Hello, John"

// Why classes are useful:
// 1. Organize related data and behavior
// 2. Enable inheritance and polymorphism
// 3. Type-safe with TypeScript
// 4. Clear object contracts
```

---

22. **Explain access modifiers**

**Explanation:** Access modifiers control where class members can be accessed. Public (default) is accessible everywhere. Private restricts access to the class only. Protected allows access in the class and subclasses. These enable encapsulation and prevent external code from misusing internal state.

```typescript
class User {
  public id: number;        // Accessible everywhere
  private password: string; // Only in this class
  protected role: string;   // In this class and subclasses
  
  constructor(id: number, password: string, role: string) {
    this.id = id;
    this.password = password;
    this.role = role;
  }
  
  private hashPassword(): string {
    // Only User can call this
    return "hashed";
  }
}

const user = new User(1, "secret", "admin");
console.log(user.id);        // OK - public
// console.log(user.password); // Error - private
// console.log(user.hashPassword()); // Error - private

// Why access modifiers matter:
// 1. Encapsulation - hide internal details
// 2. Prevent misuse - restrict what's accessible
// 3. Enable refactoring - change private without breaking code
// 4. Clear intent - shows what's part of public API
```

---

### Error Handling & Validation

23. **How do you handle errors in TypeScript?**

**Explanation:** Error handling in TypeScript uses try-catch blocks similar to JavaScript, but with type safety. You can type the caught error and create custom error types. Proper error handling makes code more robust and maintainable. Always assume external data might cause errors.

```typescript
try {
  // Code that might throw
  riskyOperation();
} catch (error) {
  // Type guard to check error type
  if (error instanceof Error) {
    console.error(error.message);
  } else if (typeof error === 'string') {
    console.error(error);
  } else {
    console.error("Unknown error:", error);
  }
} finally {
  console.log("Cleanup - always runs");
}

// Why proper error handling is important:
// 1. Prevents crashes
// 2. Provides meaningful error messages
// 3. Enables recovery
// 4. Makes debugging easier
```

---

24. **What are custom error types?**

**Explanation:** Custom error types extend the base Error class to add context-specific information. They help categorize errors and enable different handling for different error types. Custom errors make code more maintainable and enable better error handling strategies.

```typescript
// Custom error for validation failures
class ValidationError extends Error {
  constructor(message: string, public field: string) {
    super(message);
    this.name = "ValidationError";
  }
}

// Custom error for API failures
class ApiError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
    this.name = "ApiError";
  }
}

// Using custom errors
function validateEmail(email: string): void {
  if (!email.includes("@")) {
    throw new ValidationError("Invalid email format", "email");
  }
}

try {
  validateEmail("invalid");
} catch (error) {
  if (error instanceof ValidationError) {
    console.error(`Field ${error.field}: ${error.message}`);
  }
}

// Why custom errors are useful:
// 1. Categorize errors
// 2. Add context-specific info
// 3. Enable type-safe error handling
// 4. Make debugging easier
```

---

25. **How do you validate TypeScript types at runtime?**

**Explanation:** TypeScript types only exist at compile-time and disappear at runtime. To validate at runtime, you must use runtime checks (typeof, instanceof, property checks). Type guards are functions that check types at runtime and narrow types for TypeScript. Essential for validating external data.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// Type guard - validates and narrows type
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    typeof obj.id === 'number' &&
    'name' in obj &&
    typeof obj.name === 'string' &&
    'email' in obj &&
    typeof obj.email === 'string'
  );
}

// Using the type guard
const data: unknown = JSON.parse(apiResponse);
if (isUser(data)) {
  console.log(data.id);   // TypeScript knows it's User
} else {
  console.error("Invalid user data");
}

// Why runtime validation is critical:
// 1. External data can be wrong
// 2. TypeScript types disappear at runtime
// 3. APIs might change unexpectedly
// 4. Prevents bugs from invalid data
```

---

### Async & Promises

26. **How do you type Promises?**

**Explanation:** Promises are typed with `Promise<T>` where T is what the promise resolves to. You specify the resolved type in angle brackets. This tells TypeScript (and users) what value to expect when the promise settles. Proper Promise typing enables better type checking for async code.

```typescript
interface User {
  id: number;
  name: string;
}

// Promise that resolves to string
const promise1: Promise<string> = Promise.resolve("hello");

// Promise that resolves to User
const promise2: Promise<User> = fetch('/api/user')
  .then(r => r.json());

// Function returning Promise
function fetchUser(id: number): Promise<User> {
  return fetch(`/api/users/${id}`)
    .then(r => r.json());
}

// Why Promise types are important:
// 1. Know what you're getting from async code
// 2. Enable better error handling
// 3. Type-safe promise chaining
```

---

27. **What is async/await in TypeScript?**

**Explanation:** Async/await is syntactic sugar for working with Promises. `async` functions automatically return Promises, and `await` pauses execution until the promise settles. The return type annotation applies to what the promise resolves to, not the Promise itself. Async/await makes asynchronous code look synchronous.

```typescript
interface User {
  id: number;
  name: string;
}

// Async function returns Promise<User>
async function fetchData(url: string): Promise<User> {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Fetch error:", error);
    throw error;
  }
}

// Using the async function
const user = await fetchData('/api/user');
console.log(user.name);

// Why async/await is powerful:
// 1. Readable - looks like synchronous code
// 2. Error handling - can use try/catch
// 3. Type-safe - return type is clear
// 4. Easier to reason about
```

---

### Advanced Types

28. **What are literal types?**

**Explanation:** Literal types represent specific exact values rather than general types. `"success"` is more specific than `string`. You combine literal types with unions to create exhaustive lists of valid values. They prevent typos and enable autocomplete for valid values.

```typescript
type Direction = "up" | "down" | "left" | "right";
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type LogLevel = "debug" | "info" | "warn" | "error";

let direction: Direction = "up"; // OK
// direction = "invalid"; // Error

// Why literal types are useful:
// 1. Exhaustiveness - compiler ensures all cases handled
// 2. Autocomplete - IDE shows exact valid values
// 3. Documentation - self-explanatory
// 4. Prevents typos - catches misspellings
```

---

29. **Explain intersection types**

**Explanation:** Intersection types combine multiple types with `&`. The result must satisfy all types simultaneously. Different from unions (which are "or"), intersections are "and"—the value must match all types. Useful for combining unrelated types or interfaces.

```typescript
interface Nameable {
  name: string;
}

interface Identifiable {
  id: number;
}

// Intersection - must have both properties
type NamedEntity = Nameable & Identifiable;

const entity: NamedEntity = {
  name: "John",
  id: 1
};

// Intersection of types
type Point = { x: number } & { y: number };
const point: Point = { x: 1, y: 2 };

// Why intersections are useful:
// 1. Combine unrelated types
// 2. Extend types without inheritance
// 3. Mixin patterns
// 4. Flexible composition
```

---

30. **What are discriminated unions?**

**Explanation:** Discriminated unions use a common property (the discriminant) to distinguish between union members. TypeScript uses this property to narrow the type automatically. They're excellent for state machines and handling multiple cases safely. Makes code more maintainable than regular unions.

```typescript
// Discriminated union - uses 'status' property to distinguish
type Result = 
  | { status: 'success'; data: User }
  | { status: 'error'; error: string }
  | { status: 'loading' };

// TypeScript narrows automatically
function handleResult(result: Result) {
  if (result.status === 'success') {
    console.log(result.data.name); // data available
  } else if (result.status === 'error') {
    console.log(result.error); // error available
  } else {
    console.log("Loading...");
  }
}

// Why discriminated unions are powerful:
// 1. Type-safe exhaustiveness checking
// 2. Automatic type narrowing
// 3. Clear state representation
// 4. Prevents impossible states
```

---

### Utility Types

31. **What is Partial<T>?**

**Explanation:** `Partial<T>` makes all properties of type T optional. It's useful for functions that accept partial updates where all fields are optional. Creates a version of the type where nothing is required. Useful for update/patch operations.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// All properties become optional
type PartialUser = Partial<User>;

// Equivalent to:
// {
//   id?: number;
//   name?: string;
//   email?: string;
// }

// Useful for update functions
function updateUser(id: number, updates: Partial<User>): void {
  // Can update just name, just email, or both
}

updateUser(1, { name: "Jane" });

// Why Partial is useful:
// 1. Update operations - not all fields required
// 2. Configuration objects - sensible defaults
// 3. Incremental building
```

---

32. **What is Required<T>?**

**Explanation:** `Required<T>` makes all properties required, removing optionality. Opposite of Partial. Useful when you need to guarantee all fields are present. Converts optional properties to required.

```typescript
interface User {
  id?: number;
  name?: string;
  email?: string;
}

// All properties become required
type RequiredUser = Required<User>;

// Now all properties must be provided
const user: RequiredUser = {
  id: 1,
  name: "John",
  email: "john@example.com"
};

// Why Required is useful:
// 1. Ensure all fields present
// 2. Strict initialization
// 3. Converting optional to required in specific contexts
```

---

### React & Frontend Integration

33. **How do you type React components?**

**Explanation:** React components in TypeScript are typed with generic Props interfaces. Functional components use `React.FC<Props>` (though this is debated), while arrow functions just have `Props` type annotation. Event handlers have specific types like `React.ChangeEventHandler`. Proper typing catches bugs and enables IDE autocomplete.

```typescript
import React, { FC } from 'react';

interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
}

// Functional component with props type
const Button: FC<ButtonProps> = ({ label, onClick, disabled }) => {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
};

// Event handler types
interface InputProps {
  value: string;
  onChange: (value: string) => void;
}

const Input: FC<InputProps> = ({ value, onChange }) => {
  const handleChange: React.ChangeEventHandler<HTMLInputElement> = (e) => {
    onChange(e.target.value);
  };
  
  return <input value={value} onChange={handleChange} />;
};

// Why React typing matters:
// 1. Props validation - ensures correct usage
// 2. IDE autocomplete - know available props
// 3. Catch prop mismatches at compile-time
// 4. Self-documenting - clear what component needs
```

---

34. **What are React Hooks in TypeScript?**

**Explanation:** React hooks have specific TypeScript types. `useState` is generic—specify the state type. `useEffect` doesn't require types. `useContext` needs generic for context type. `useRef` needs generic for ref type. Proper hook typing prevents many bugs and enables type safety in functional components.

```typescript
interface User {
  id: number;
  name: string;
}

// useState with type
const [user, setUser] = useState<User | null>(null);
const [count, setCount] = useState<number>(0);

// useEffect doesn't need types
useEffect(() => {
  // Effect code
}, [user]);

// useContext with type
interface Theme {
  primary: string;
  secondary: string;
}

const ThemeContext = React.createContext<Theme>({
  primary: "blue",
  secondary: "gray"
});

const theme = useContext(ThemeContext); // theme is Theme type

// useRef with type
const inputRef = useRef<HTMLInputElement>(null);

// Why hook typing is important:
// 1. Type safety for state
// 2. Prevents type confusion
// 3. IDE autocomplete
// 4. Catch bugs early
```

---

### Configuration & Compilation

35. **What is tsconfig.json?**

**Explanation:** `tsconfig.json` configures TypeScript compilation. It specifies what files to compile, where to output, which ES version to target, and how strict type checking should be. Every TypeScript project should have one. Key options include `target` (ES version), `module` (module system), and `strict` (type checking strictness).

```json
{
  "compilerOptions": {
    "target": "ES2020",              // Output JavaScript version
    "module": "commonjs",            // Module system
    "lib": ["ES2020", "DOM"],        // Available types
    "strict": true,                  // Enable all strict checks
    "esModuleInterop": true,         // CommonJS/ES modules interop
    "skipLibCheck": true,            // Skip type checking .d.ts files
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",              // Output directory
    "rootDir": "./src",              // Source directory
    "declaration": true,             // Generate .d.ts files
    "sourceMap": true                // Generate source maps
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

36. **Explain strict mode**

**Explanation:** Strict mode enables all strict type checking flags. It includes: noImplicitAny (error on implicit any), strictNullChecks (treat null/undefined as errors), strictFunctionTypes (stricter function type checking), and others. Strict mode is recommended for new projects as it catches many bugs.

```json
{
  "compilerOptions": {
    "strict": true  // Enables all strict flags
  }
}
```

With strict mode:
- Can't use implicit `any`
- Must handle `null` and `undefined`
- Function types are stricter
- Properties must be initialized
- `this` is checked

---

### Best Practices & Patterns

37. **What is the discriminated union pattern?**

**Explanation:** Discriminated unions use a literal type property to distinguish union members. TypeScript automatically narrows when you check this property. It's a pattern for representing state machines and complex business logic safely. Makes error handling exhaustive.

```typescript
type LoadingState = { status: 'loading' };
type SuccessState = { status: 'success'; data: User };
type ErrorState = { status: 'error'; error: string };

type UserState = LoadingState | SuccessState | ErrorState;

function renderState(state: UserState) {
  switch (state.status) {
    case 'loading':
      return <div>Loading...</div>;
    case 'success':
      return <div>{state.data.name}</div>;
    case 'error':
      return <div>Error: {state.error}</div>;
  }
}

// Why this pattern is powerful:
// 1. Exhaustive handling - compiler ensures all cases
// 2. Type safety - can't access wrong state properties
// 3. Clear intent - explicitly shows possible states
// 4. Prevents bugs - impossible states can't occur
```

---

38. **What is the builder pattern in TypeScript?**

**Explanation:** The builder pattern creates complex objects step by step. In TypeScript, builders enable fluent interfaces and type-safe construction. They're useful when objects have many optional properties. Builders make code more readable than long constructor parameter lists.

```typescript
class UserBuilder {
  private user: Partial<User> = {};
  
  setId(id: number): this {
    this.user.id = id;
    return this; // Return this for chaining
  }
  
  setName(name: string): this {
    this.user.name = name;
    return this;
  }
  
  setEmail(email: string): this {
    this.user.email = email;
    return this;
  }
  
  build(): User {
    if (!this.user.id || !this.user.name || !this.user.email) {
      throw new Error("Missing required fields");
    }
    return this.user as User;
  }
}

// Fluent API
const user = new UserBuilder()
  .setId(1)
  .setName("John")
  .setEmail("john@example.com")
  .build();

// Why builders are useful:
// 1. Fluent interface - readable
// 2. Optional properties - not all required upfront
// 3. Validation - ensures valid objects
// 4. Prevents constructor bloat
```

---

## VERBAL/CONCEPTUAL QUESTIONS

### Fundamentals

1. **What is TypeScript and why use it?**

**Explanation:** TypeScript is a superset of JavaScript that adds static typing. It compiles to JavaScript, so it runs everywhere JavaScript does. The key advantage is catching errors at compile-time instead of runtime. TypeScript enables better IDE support (autocomplete, refactoring), self-documenting code through types, and prevents entire classes of bugs. Ideal for large projects or teams.

**Benefits:**
- Catch errors early - before code runs
- Better IDE support - autocomplete, refactoring
- Self-documenting - types explain intent
- Easier refactoring - compiler catches mistakes
- Scales well - large projects benefit most

---

2. **How does TypeScript compilation work?**

**Explanation:** The TypeScript compiler (tsc) has two main jobs: type checking and transpilation. First, it parses TypeScript, builds an AST (abstract syntax tree), and performs type checking—comparing actual types against declared types. Then it transpiles to JavaScript by removing types and converting modern syntax to target ES version. The output is plain JavaScript that any runtime can execute.

**Process:**
1. Parse TypeScript → AST
2. Type checking → Find errors
3. Transpile → Remove types, convert syntax
4. Output JavaScript

---

3. **What is the difference between TypeScript and JavaScript?**

**Explanation:** 
- JavaScript is dynamically typed - types determined at runtime, errors found when code runs
- TypeScript adds static typing - types checked at compile-time, errors found before code runs

JavaScript is simpler initially but harder to maintain as projects grow. TypeScript adds complexity upfront but prevents bugs and enables better tooling. They're complementary—TypeScript is a superset, so all JavaScript is valid TypeScript.

---

4. **What are the benefits of static typing?**

**Explanation:**
- Early error detection - typos and type mismatches caught before running
- IDE support - autocomplete knows available methods
- Self-documenting - types show intent without reading implementation
- Easier refactoring - compiler finds all usages and errors
- Better performance - compiler can optimize knowing exact types
- Catch impossible states - can't create invalid data

---

5. **What is structural vs nominal typing?**

**Explanation:** TypeScript uses structural typing - types are compatible if they have the same structure, regardless of declared type. JavaScript uses duck typing at runtime. Nominal typing (used in Java/C++) requires explicit type relationship declarations.

```typescript
// Structural typing - compatible if structure matches
interface Point { x: number; y: number }
interface Location { x: number; y: number }

const point: Point = { x: 1, y: 2 };
const location: Location = point; // OK - same structure

// Nominal typing (not TypeScript default) would require explicit inheritance
```

---

6. **What is type inference and when is it limited?**

**Explanation:** Type inference means TypeScript figures out types without explicit annotations. It's limited when:
- Context is ambiguous (does `[]` mean `any[]` or `string[]`?)
- Types are complex (nested generics)
- Edge cases aren't obvious (type widening)

Explicit annotations help in these cases and improve readability.

---

### Design Patterns & Architecture

7. **What are design patterns in TypeScript?**

**Explanation:** Common patterns implemented in TypeScript include:
- Singleton - one instance only
- Factory - create objects without specifying exact class
- Observer - notify multiple listeners of changes
- Builder - construct complex objects step by step
- Strategy - multiple algorithms, pick one
- Decorator - add functionality to objects dynamically

TypeScript's type system enables safer implementations of these patterns.

---

8. **What is dependency injection?**

**Explanation:** DI is a pattern where objects receive their dependencies rather than creating them. Makes code more testable (inject mocks), flexible (swap implementations), and loosely coupled. Common in frameworks like NestJS.

```typescript
// Without DI - tightly coupled
class Service {
  private db = new Database(); // Creates own dependency
}

// With DI - loosely coupled
class Service {
  constructor(private db: Database) {} // Receives dependency
}
```

**Benefits:**
- Testability - inject mocks in tests
- Flexibility - swap implementations easily
- Loose coupling - depends on abstraction not concrete class

---

9. **What is the difference between composition and inheritance?**

**Explanation:** 
- Inheritance: "is-a" relationship, Dog extends Animal
- Composition: "has-a" relationship, Car has Engine

TypeScript (like JavaScript) has single inheritance, so composition is often better. Composition is more flexible, avoids fragile base class problem, and enables runtime changes.

```typescript
// Composition - flexible, prefer this
class Car {
  constructor(private engine: Engine) {}
}

// Inheritance - less flexible
class Dog extends Animal {}
```

---

10. **What is SOLID principles in TypeScript?**

**Explanation:** SOLID are design principles for maintainable code:
- **S**ingle Responsibility - class does one thing
- **O**pen/Closed - open for extension, closed for modification
- **L**iskov Substitution - subclass can replace parent
- **I**nterface Segregation - specific interfaces, not monolithic ones
- **D**ependency Inversion - depend on abstractions, not concretions

TypeScript's type system enables these principles naturally.

---

### Performance & Optimization

11. **How do you optimize TypeScript build performance?**

**Explanation:**
- Use incremental compilation - `"incremental": true` in tsconfig
- Skip type checking for certain files
- Use project references - separate large projects
- Cache compiler output
- Parallelize builds

These techniques significantly speed up development.

---

12. **What is Project References?**

**Explanation:** Project references allow splitting large TypeScript projects into smaller ones with proper dependency declarations. Each reference can be built independently. Enables faster builds and better organization for monorepos.

---

13. **What is Watch Mode?**

**Explanation:** Watch mode automatically recompiles when files change. Enables fast development - save file, see errors immediately. Most development servers use watch mode internally.

---

14. **What are source maps?**

**Explanation:** Source maps map compiled JavaScript back to original TypeScript line-by-line. Enable debugging compiled code - breakpoints in DevTools go to TypeScript source. Essential for debugging production code that's been transpiled and minified.

---

### Testing & Quality

15. **How do you test TypeScript code?**

**Explanation:** TypeScript benefits from same testing approaches as JavaScript (Jest, Mocha) plus type testing. Write unit tests as normal; types are verified by compiler. Can also test types using type assertion libraries.

```typescript
describe('add', () => {
  it('should add numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
});
```

---

16. **How do you test type definitions?**

**Explanation:** Type definitions can be tested with libraries like `dtslint` or conditional type tests. Ensure types are correct and catch regressions. Important for library authors.

---

17. **What is contract-based testing?**

**Explanation:** Both API provider and consumer define contract (interface). Each tests against contract. Prevents breaking changes and ensures compatibility. Especially useful in microservices.

---

### Integration & Tools

18. **How do you integrate TypeScript with build tools?**

**Explanation:** Build tools (Webpack, Vite, esbuild) have TypeScript loaders that compile on the fly. Most integrate seamlessly - just configure loader and tsconfig. Modern tooling makes TypeScript transparent.

---

19. **What is tree-shaking in TypeScript?**

**Explanation:** Tree-shaking removes unused code from bundles. Works by analyzing import/export relationships. TypeScript's ES6 modules (unlike CommonJS) enable static analysis for tree-shaking. Reduces bundle size.

---

20. **What is code splitting?**

**Explanation:** Code splitting divides bundle into chunks loaded on-demand (lazy loading). Reduces initial bundle size. Works with dynamic imports. TypeScript supports this transparently.

```typescript
const Component = React.lazy(() => import('./Component'));
```

---

### Best Practices

21. **When should you use any?**

**Explanation:** Use `any` as a last resort. It defeats the purpose of TypeScript. Acceptable when:
- Working with untyped legacy code
- External untyped libraries (temporary)
- Truly don't know type
- Extreme edge cases

Use `unknown` instead when possible - safer.

---

22. **When to use unknown vs any?**

**Explanation:** 
- `unknown` - safe, requires type checking before use
- `any` - unsafe, bypasses type checking

`unknown` is better 99% of the time. Use it for external data, APIs, user input.

---

23. **How do you avoid over-typing?**

**Explanation:** Too many explicit types can be verbose and brittle. Let TypeScript infer when obvious. Explicit types when:
- Function parameters/returns (public API)
- Complex logic (clarity)
- Ambiguous situations

Balance expressiveness with reducing noise.

---

24. **What is type widening?**

**Explanation:** Type widening is TypeScript inferring broader types. `let x = "hello"` infers `string` not `"hello"`. Use `as const` to prevent widening. Important for understanding inferred types.

```typescript
const x = "hello";  // Type: "hello" (literal)
let y = "hello";    // Type: string (widened)

const z = "hello" as const; // Type: "hello" (prevented widening)
```

---

25. **What are const assertions?**

**Explanation:** `as const` prevents type widening and makes properties readonly. Useful for literal types and tuples. Creates immutable type views.

```typescript
const colors = ["red", "green", "blue"] as const;
// Type: readonly ["red", "green", "blue"] not string[]
```

---

### Security & Maintenance

26. **How do you prevent common security issues with TypeScript?**

**Explanation:**
- Input validation - never trust external data
- Type guards - validate at runtime
- Avoid `any` - maintains type safety
- Use strict mode - catches more errors
- Principle of least privilege - minimal permissions/access

TypeScript helps prevent type-based vulnerabilities.

---

27. **What are circular dependencies?**

**Explanation:** When module A imports B and B imports A. Causes issues in bundling and understanding dependencies. Avoid by:
- Reorganize code - shared module
- Use interfaces - separate from implementation
- Reverse dependency - one imports other

---

28. **How do you handle sensitive data in types?**

**Explanation:** Never expose sensitive data in types:
- Omit sensitive properties from APIs
- Use Pick/Omit to create safe versions
- Don't include passwords, tokens in types

```typescript
type UserPublic = Omit<User, 'password'>;
```

---

### Career & Growth

29. **How do you learn TypeScript effectively?**

**Explanation:**
- Start with fundamentals (types, interfaces, generics)
- Build projects incrementally
- Read source code of typed libraries
- Understand patterns over syntax
- Practice in real code, not just exercises

---

30. **What resources are available for learning TypeScript?**

**Explanation:**
- Official TypeScript Handbook
- Type challenges on GitHub
- Video courses (FrontendMasters, Egghead)
- Open source projects
- Community (TypeScript Discord, Reddit)

---

**This comprehensive TypeScript guide covers everything needed for interviews! 🚀**
