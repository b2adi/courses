# TypeScript Coding Practice - 25 Exercises with Solutions

Complete collection of TypeScript practice problems to master interview concepts.

---

## 🟢 EASY LEVEL (Start Here)

### 1. Type a User Object
**Problem:** Create a User interface with required and optional fields.

```typescript
// Solution
interface User {
  id: number;
  name: string;
  email: string;
  phone?: string;      // Optional
  createdAt: Date;
  isActive: boolean;
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
  createdAt: new Date(),
  isActive: true
};
```

**Explanation:** Interfaces define contracts. The `?` makes a field optional. This is fundamental—you'll type objects in almost every TypeScript project.

---

### 2. Create a Union Type for Status
**Problem:** Create a type that allows only specific status values.

```typescript
// Solution
type Status = "pending" | "approved" | "rejected" | "archived";

function processStatus(status: Status): string {
  switch(status) {
    case "pending": return "Processing...";
    case "approved": return "Approved!";
    case "rejected": return "Rejected.";
    case "archived": return "Archived.";
  }
}

// TypeScript ensures all cases handled
```

**Explanation:** Union types restrict values to specific options. This pattern prevents typos and enables autocomplete. Combined with exhaustiveness checking, it's powerful for state machines.

---

### 3. Generic Function for Array Operations
**Problem:** Write a generic function that works with any array type.

```typescript
// Solution
function getFirst<T>(arr: T[]): T | undefined {
  return arr[0];
}

function getLast<T>(arr: T[]): T | undefined {
  return arr[arr.length - 1];
}

// Works with any type
const firstNum = getFirst([1, 2, 3]);      // Type: number
const firstStr = getFirst(["a", "b"]);      // Type: string
const firstUser = getFirst(users);          // Type: User
```

**Explanation:** Generics let one function work with multiple types while maintaining type safety. `T` is a placeholder that TypeScript fills in based on usage. Generics are essential for reusable code.

---

### 4. Type Narrowing in Functions
**Problem:** Handle a value that could be string or number.

```typescript
// Solution
function process(value: string | number): void {
  // Type narrowing with typeof
  if (typeof value === 'string') {
    console.log(value.toUpperCase());      // Safe - value is string
  } else {
    console.log(value.toFixed(2));         // Safe - value is number
  }
}

// With type guard
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function process2(value: unknown): void {
  if (isString(value)) {
    console.log(value.toUpperCase());      // Narrowed to string
  }
}
```

**Explanation:** With union types, you must narrow before using type-specific operations. Type guards (functions returning `is`) enable reusable narrowing logic. This prevents runtime errors.

---

### 5. Readonly Properties
**Problem:** Create an interface where properties cannot be modified.

```typescript
// Solution
interface Config {
  readonly apiUrl: string;
  readonly port: number;
  readonly timeout: number;
}

const config: Config = {
  apiUrl: "http://localhost",
  port: 3000,
  timeout: 5000
};

// config.port = 8000;  // Error - readonly

// Make entire object readonly
const readonlyUser: Readonly<User> = user;
// readonlyUser.name = "Jane";  // Error
```

**Explanation:** Readonly prevents accidental modifications. Useful for configuration, immutable data structures, and making intent clear. The `Readonly<T>` utility type makes all properties readonly.

---

### 6. Optional Chaining
**Problem:** Safely access nested properties that might not exist.

```typescript
// Solution
interface Address {
  street?: string;
  city?: string;
  country?: string;
}

interface User {
  name: string;
  address?: Address;
}

const user: User = { name: "John" };

// Optional chaining - returns undefined if property doesn't exist
const city = user.address?.city;           // undefined
const street = user.address?.street;       // undefined

// Works with functions too
const json = someObject?.toJSON?.();       // Calls if exists
```

**Explanation:** Optional chaining (`?.`) safely accesses nested properties. Returns `undefined` instead of throwing error if property doesn't exist. Essential for working with incomplete or optional objects.

---

### 7. Nullish Coalescing
**Problem:** Use default value when value is null or undefined.

```typescript
// Solution
interface User {
  name: string;
  nickname?: string;
}

const user: User = { name: "John" };

// Nullish coalescing - uses default if null/undefined
const displayName = user.nickname ?? user.name;  // "John"

// Unlike ||, works with falsy values like 0 or ""
const count = savedCount ?? 0;    // 0 even if savedCount is 0
const message = text ?? "default"; // "default" only if null/undefined

// Different from || which treats all falsy values as missing
const wrongCount = savedCount || 0;  // 0 even if savedCount is 0 (problem!)
```

**Explanation:** Nullish coalescing (`??`) only uses default for null/undefined, not other falsy values. Important distinction from `||`. Better for handling missing data.

---

### 8. Discriminated Union (Tagged Union)
**Problem:** Create a type representing different states.

```typescript
// Solution
type Result =
  | { status: 'success'; data: User }
  | { status: 'error'; error: string }
  | { status: 'loading' };

function handleResult(result: Result): void {
  // TypeScript narrows automatically based on status
  switch(result.status) {
    case 'success':
      console.log(result.data.name);  // data available
      break;
    case 'error':
      console.error(result.error);    // error available
      break;
    case 'loading':
      console.log("Loading...");
      break;
  }
}
```

**Explanation:** Discriminated unions use a common property (discriminant) to narrow types automatically. Excellent for representing different states. Compiler ensures all cases handled. Pattern frequently asked in interviews.

---

### 9. Pick Utility Type
**Problem:** Select specific properties from an interface.

```typescript
// Solution
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  phone?: string;
}

// Pick only specific properties
type UserPreview = Pick<User, 'id' | 'name' | 'email'>;

// Equivalent to:
// {
//   id: number;
//   name: string;
//   email: string;
// }

const preview: UserPreview = {
  id: 1,
  name: "John",
  email: "john@example.com"
};

// password not available - good for API responses
```

**Explanation:** `Pick<T, K>` creates new type with only specified properties. Useful for API responses, hiding sensitive data, or creating views of types. Frequently used in real projects.

---

### 10. Omit Utility Type
**Problem:** Create a type by excluding certain properties.

```typescript
// Solution
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  phone?: string;
}

// Omit specific properties
type UserPublic = Omit<User, 'password'>;

// Equivalent to Pick<User, 'id' | 'name' | 'email' | 'phone'>

const publicUser: UserPublic = {
  id: 1,
  name: "John",
  email: "john@example.com"
  // password not needed
};

// Hide sensitive data before sending to client
```

**Explanation:** `Omit<T, K>` is opposite of Pick—removes specified properties. Useful for excluding sensitive properties. Common pattern in API typing.

---

## 🟡 MEDIUM LEVEL

### 11. Keyof Operator with Generics
**Problem:** Create a function that safely accesses object properties.

```typescript
// Solution
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: "John", email: "john@example.com" };

const id = getProperty(user, "id");      // OK - "id" is valid
const name = getProperty(user, "name");  // OK - "name" is valid
// getProperty(user, "phone");  // Error - "phone" not in user

// Type-safe property access
```

**Explanation:** `keyof T` produces union of all keys of T. `K extends keyof T` ensures K is a valid key. This is very powerful for creating type-safe utilities. Important pattern in interviews.

---

### 12. Typeof Operator
**Problem:** Extract type from a variable or expression.

```typescript
// Solution
const user = { id: 1, name: "John", age: 30 };
type UserType = typeof user;
// Result: { id: number; name: string; age: number }

// Works with any expression
const numbers = [1, 2, 3];
type NumberArrayType = typeof numbers;  // number[]

// Extract from function
function greet(name: string): string {
  return `Hello, ${name}`;
}
type GreetType = typeof greet;
// Result: (name: string) => string

// Useful for avoiding duplicate type definitions
```

**Explanation:** `typeof` in TypeScript extracts compile-time type from values. Different from JavaScript's `typeof`. Useful for creating types from existing values without redefining.

---

### 13. Mapped Types
**Problem:** Transform an interface by making all properties optional.

```typescript
// Solution
interface User {
  id: number;
  name: string;
  email: string;
}

// Make all properties optional
type PartialUser = {
  [K in keyof User]?: User[K];
};

// Or use built-in Partial
type PartialUser2 = Partial<User>;

// Make all properties readonly
type ReadonlyUser = {
  readonly [K in keyof User]: User[K];
};

// Or use built-in Readonly
type ReadonlyUser2 = Readonly<User>;

const partial: PartialUser = { name: "John" };  // OK - missing fields
const readonly: ReadonlyUser = { id: 1, name: "John", email: "john@example.com" };
// readonly.id = 2;  // Error - readonly
```

**Explanation:** Mapped types iterate over properties and create new types. Pattern: `[K in keyof T]` transforms each property. Powerful for creating type variants without duplication.

---

### 14. Conditional Types
**Problem:** Create a type that returns different types based on conditions.

```typescript
// Solution
// Simple conditional
type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">;  // true
type B = IsString<number>;   // false

// Practical - flatten arrays
type Flatten<T> = T extends Array<infer U> ? U : T;

type Str = Flatten<string[]>;     // string
type Num = Flatten<number>;       // number

// Extract promise type
type Awaited<T> = T extends Promise<infer U> ? U : T;

type PromiseString = Awaited<Promise<string>>;  // string
type Regular = Awaited<number>;                 // number
```

**Explanation:** Conditional types enable type-level logic. `T extends X ? Y : Z` checks if T is assignable to X. The `infer` keyword extracts types from complex types. Essential for advanced patterns.

---

### 15. Custom Type Guard
**Problem:** Create a function that narrows types safely.

```typescript
// Solution
interface User {
  id: number;
  name: string;
  type: 'user';
}

interface Admin {
  id: number;
  name: string;
  type: 'admin';
  permissions: string[];
}

type Person = User | Admin;

// Type guard function - returns boolean
function isAdmin(person: Person): person is Admin {
  return person.type === 'admin';
}

function processPerson(person: Person): void {
  if (isAdmin(person)) {
    console.log(person.permissions);  // person narrowed to Admin
  } else {
    console.log(person.name);         // person narrowed to User
  }
}
```

**Explanation:** Type guard is function returning `value is Type`. Enables type narrowing outside switch statements. Useful for custom logic. Frequently used with complex unions.

---

### 16. Generic Constraints
**Problem:** Restrict generic parameter to specific types.

```typescript
// Solution
// Constraint: must have length property
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength("hello");     // OK - string has length
getLength([1, 2, 3]);   // OK - array has length
// getLength(42);       // Error - number has no length

// Constraint: must be key of object
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Constraint: must extend base class
class Animal { move() {} }
class Dog extends Animal { bark() {} }

function create<T extends Animal>(ctor: new () => T): T {
  return new ctor();
}

create(Dog);     // OK
// create(String);  // Error - String doesn't extend Animal
```

**Explanation:** Constraints limit what types can be used. `T extends X` means T must be assignable to X. Enables assumptions about the type. Important for type-safe generics.

---

### 17. Record Type
**Problem:** Create object with specific keys all mapping to same value type.

```typescript
// Solution
type Color = 'red' | 'green' | 'blue';

// Record ensures all keys provided
const colorHex: Record<Color, string> = {
  red: '#FF0000',
  green: '#00FF00',
  blue: '#0000FF'
};

// Common use: status codes
type StatusCode = 'success' | 'error' | 'pending';
const messages: Record<StatusCode, string> = {
  success: 'Operation successful',
  error: 'Operation failed',
  pending: 'Please wait...'
};

// With generic
type Roles = 'admin' | 'user' | 'guest';
type Permissions = Record<Roles, string[]>;

const rolePermissions: Permissions = {
  admin: ['read', 'write', 'delete'],
  user: ['read', 'write'],
  guest: ['read']
};
```

**Explanation:** `Record<K, V>` creates object with all keys from union K mapping to type V. Ensures all keys provided. Useful for configuration maps and enums.

---

### 18. Extract and Exclude Utility Types
**Problem:** Select or remove types from union.

```typescript
// Solution
type A = 'a' | 'b' | 'c' | 'd';
type B = 'a' | 'c';

// Extract - keep only types in B
type Common = Extract<A, B>;  // 'a' | 'c'

// Exclude - remove types in B
type Remaining = Exclude<A, B>;  // 'b' | 'd'

// Practical example
type Methods = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
type SafeMethods = 'GET' | 'HEAD' | 'OPTIONS';

type UnsafeMethods = Exclude<Methods, SafeMethods>;
// Result: 'POST' | 'PUT' | 'DELETE' | 'PATCH'
```

**Explanation:** `Extract<T, U>` keeps types in both, `Exclude<T, U>` removes types in U. Useful for set operations on union types.

---

### 19. Partial and Required Types
**Problem:** Make all properties optional or required.

```typescript
// Solution
interface Config {
  host?: string;
  port?: number;
  timeout?: number;
}

// Make all required
type RequiredConfig = Required<Config>;
// Now all properties must be provided

const config: RequiredConfig = {
  host: 'localhost',
  port: 3000,
  timeout: 5000
};

// Make all optional
interface StrictConfig {
  host: string;
  port: number;
}

type OptionalConfig = Partial<StrictConfig>;
const partial: OptionalConfig = { host: 'localhost' };  // OK
```

**Explanation:** `Partial<T>` makes all properties optional, `Required<T>` makes all required. Opposite transformations. Useful for different contexts (init vs complete).

---

### 20. Intersection Types
**Problem:** Combine multiple types requiring all properties.

```typescript
// Solution
interface Named {
  name: string;
}

interface Identifiable {
  id: number;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

// Intersection - must have all properties
type Entity = Named & Identifiable & Timestamped;

const user: Entity = {
  id: 1,
  name: "John",
  createdAt: new Date(),
  updatedAt: new Date()
};

// Useful for mixins
type Admin = User & { role: 'admin'; permissions: string[] };

// Object intersection
type Result<T> = { status: 'success'; data: T } & { timestamp: Date };
```

**Explanation:** Intersection types combine multiple types with `&`. Result must satisfy all types. Different from unions (|) which are "or". Used for mixins and combining concerns.

---

## 🔴 HARD LEVEL

### 21. Deep Partial Type
**Problem:** Make all properties optional recursively.

```typescript
// Solution
type DeepPartial<T> = T extends object 
  ? {
      [P in keyof T]?: DeepPartial<T[P]>;
    }
  : T;

interface Config {
  database: {
    host: string;
    port: number;
    credentials: {
      user: string;
      password: string;
    }
  }
}

type PartialConfig = DeepPartial<Config>;

const config: PartialConfig = {
  database: {
    credentials: {
      user: "admin"
      // other fields optional
    }
  }
};
```

**Explanation:** Recursive type that applies Partial to nested objects. Uses conditional types and recursion. Advanced pattern for handling complex nested structures.

---

### 22. Type-Safe Event Emitter
**Problem:** Create event emitter where event handlers are type-safe.

```typescript
// Solution
type Events = {
  'user:created': { id: number; name: string };
  'user:deleted': { id: number };
  'error': { code: number; message: string };
};

class EventEmitter {
  private handlers: Map<string, Function[]> = new Map();
  
  on<E extends keyof Events>(
    event: E,
    handler: (data: Events[E]) => void
  ): void {
    if (!this.handlers.has(event as string)) {
      this.handlers.set(event as string, []);
    }
    this.handlers.get(event as string)!.push(handler);
  }
  
  emit<E extends keyof Events>(event: E, data: Events[E]): void {
    this.handlers.get(event as string)?.forEach(h => h(data));
  }
}

const emitter = new EventEmitter();

emitter.on('user:created', (data) => {
  console.log(data.id, data.name);  // Types available
  // console.log(data.code);  // Error - code not in user:created
});

emitter.emit('user:created', { id: 1, name: 'John' });
// emitter.emit('user:created', { id: 1 });  // Error - missing name
```

**Explanation:** Type-safe event system using mapped types and keyof. Each event has specific data type. Prevents using wrong data for events. Advanced pattern showing practical uses of generics.

---

### 23. Utility Type Implementation - GetReturnType
**Problem:** Extract return type from any function.

```typescript
// Solution
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getString(): string {
  return "hello";
}

function getNumber(): number {
  return 42;
}

type StringReturn = GetReturnType<typeof getString>;  // string
type NumberReturn = GetReturnType<typeof getNumber>;  // number

// Works with complex returns too
const promiseFunc = async () => ({ id: 1, name: "John" });
type PromiseReturn = GetReturnType<typeof promiseFunc>;
// Promise<{ id: number; name: string }>
```

**Explanation:** Uses `infer` to extract return type from function signature. Pattern is: function returns something, extract that something. Powerful for metaprogramming.

---

### 24. Builder Pattern with Type Safety
**Problem:** Create fluent builder ensuring required fields before building.

```typescript
// Solution
interface User {
  id: number;
  name: string;
  email: string;
  phone?: string;
}

class UserBuilder {
  private data: Partial<User> = {};
  
  setId(id: number): this {
    this.data.id = id;
    return this;
  }
  
  setName(name: string): this {
    this.data.name = name;
    return this;
  }
  
  setEmail(email: string): this {
    this.data.email = email;
    return this;
  }
  
  setPhone(phone: string): this {
    this.data.phone = phone;
    return this;
  }
  
  build(): User {
    // Validate required fields
    if (!this.data.id || !this.data.name || !this.data.email) {
      throw new Error("Missing required fields");
    }
    return this.data as User;
  }
}

// Fluent API
const user = new UserBuilder()
  .setId(1)
  .setName("John")
  .setEmail("john@example.com")
  .build();
```

**Explanation:** Builder pattern creates complex objects step by step. Chainable methods return `this`. Validates before returning. Useful for objects with many optional properties.

---

### 25. Type-Safe API Client
**Problem:** Create client where request/response types are linked.

```typescript
// Solution
type ApiEndpoints = {
  '/users': {
    GET: { response: User[] };
    POST: { body: CreateUserDTO; response: User };
  };
  '/users/:id': {
    GET: { response: User };
    PUT: { body: UpdateUserDTO; response: User };
    DELETE: { response: void };
  };
};

class ApiClient {
  async request<
    E extends keyof ApiEndpoints,
    M extends keyof ApiEndpoints[E]
  >(
    endpoint: E,
    method: M,
    body?: ApiEndpoints[E][M] extends { body: infer B } ? B : undefined
  ): Promise<ApiEndpoints[E][M]['response']> {
    // Implementation
    return fetch(`http://api.com${endpoint as string}`, {
      method: method as string,
      body: JSON.stringify(body)
    }).then(r => r.json());
  }
}

const client = new ApiClient();

// Type-safe calls
const users = await client.request('/users', 'GET');
// users is User[]

const newUser = await client.request('/users', 'POST', {
  // body required, type-checked
  name: 'John',
  email: 'john@example.com'
});

// Error - DELETE shouldn't have body
// const result = await client.request('/users/:id', 'DELETE', { });
```

**Explanation:** Advanced pattern linking routes to request/response types. Each endpoint/method has specific contract. Compiler enforces correct usage. Shows real-world complexity.

---

## 📋 Practice Tips

1. **Understand Before Typing** - Read problem carefully first
2. **Test Your Types** - Hover over variables to see inferred types
3. **Use TypeScript Playground** - Practice online instantly
4. **Type Incrementally** - Start loose, add constraints gradually
5. **Read Error Messages** - They guide you to solutions
6. **Check Utility Types** - Often there's a built-in solution
7. **Practice Explaining** - Say why types work, not just syntax

---

## 🎯 Practice Schedule

**Week 1:** Problems 1-10 (Easy) - Build fundamentals
**Week 2:** Problems 11-17 (Medium) - Advanced concepts
**Week 3:** Problems 18-25 (Hard) - Master patterns
**Week 4:** Mix all levels, focus on speed

---

**Keep practicing! Typing is a skill that improves with practice! 💪**
