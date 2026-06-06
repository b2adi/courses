# Node.js Fundamentals

## Table of Contents
1. [What is Node.js?](#what-is-nodejs)
2. [Event-Driven Architecture](#event-driven-architecture)
3. [CommonJS vs ES Modules](#commonjs-vs-es-modules)
4. [File System Operations](#file-system-operations)
5. [NPM & Package Management](#npm--package-management)

---

## What is Node.js?

Node.js is a **JavaScript runtime** built on Chrome's V8 engine that allows you to run JavaScript outside the browser.

### Key Characteristics:
- **Single-threaded, event-driven, non-blocking I/O model**
- **Asynchronous** by default
- **Lightweight and efficient**
- Perfect for **data-intensive, real-time applications**

### Example:
```javascript
// Simple Node.js server
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello, Node.js!\n');
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

## Event-Driven Architecture

Node.js uses an **event emitter pattern** at its core.

### How it Works:
1. **Event Loop** - continuously checks for events
2. **Event Queue** - stores pending events
3. **Callback Execution** - executes callbacks when events occur

### Example: Event Emitter
```javascript
const EventEmitter = require('events');

const emitter = new EventEmitter();

// Register listeners
emitter.on('greet', (name) => {
  console.log(`Hello, ${name}!`);
});

// Emit event
emitter.emit('greet', 'Node.js'); // Output: Hello, Node.js!
```

### Key Methods:
- `on(eventName, callback)` - Listen for an event
- `once(eventName, callback)` - Listen once, then remove
- `emit(eventName, data)` - Trigger an event
- `off(eventName, callback)` - Remove listener
- `listenerCount(eventName)` - Count listeners

---

## CommonJS vs ES Modules

### CommonJS (Traditional)
```javascript
// Export
module.exports = {
  add: (a, b) => a + b
};

// Import
const math = require('./math');
console.log(math.add(2, 3)); // 5
```

### ES Modules (Modern)
```javascript
// Export
export const add = (a, b) => a + b;

// Import
import { add } from './math.js';
console.log(add(2, 3)); // 5
```

### Key Differences:

| Feature | CommonJS | ES Modules |
|---------|----------|-----------|
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Loading | Synchronous | Asynchronous |
| Tree Shaking | ❌ No | ✅ Yes |
| Usage | Default in older Node | Default in modern Node |

### Enable ES Modules in Node.js:
```json
{
  "type": "module"
}
```

---

## File System Operations

### Reading Files

#### Synchronous (Blocking)
```javascript
const fs = require('fs');

const content = fs.readFileSync('file.txt', 'utf-8');
console.log(content);
```

#### Asynchronous (Non-blocking)
```javascript
const fs = require('fs');

fs.readFile('file.txt', 'utf-8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

#### Using Promises
```javascript
const fs = require('fs').promises;

const content = await fs.readFile('file.txt', 'utf-8');
console.log(content);
```

### Writing Files
```javascript
const fs = require('fs').promises;

// Write to file
await fs.writeFile('output.txt', 'Hello, Node.js!');

// Append to file
await fs.appendFile('output.txt', '\nMore content');
```

### Directory Operations
```javascript
const fs = require('fs').promises;

// Create directory
await fs.mkdir('newFolder', { recursive: true });

// List files
const files = await fs.readdir('newFolder');
console.log(files);

// Remove directory
await fs.rmdir('newFolder');
```

### File Stats
```javascript
const fs = require('fs').promises;

const stats = await fs.stat('file.txt');
console.log(stats.size);      // File size
console.log(stats.isFile());  // Is it a file?
console.log(stats.isDirectory()); // Is it a directory?
```

---

## NPM & Package Management

### Basic Commands
```bash
# Initialize a new project
npm init

# Install package locally
npm install express

# Install package globally
npm install -g nodemon

# Install from package.json
npm install

# Update packages
npm update

# Uninstall package
npm uninstall express

# List installed packages
npm list
```

### package.json Structure
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My awesome app",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0"
  }
}
```

### Semantic Versioning
- `^4.18.0` - Up to next major version (4.x.x)
- `~4.18.0` - Up to next minor version (4.18.x)
- `4.18.0` - Exact version

---

## Best Practices

1. ✅ Always use `async/await` over callbacks for readability
2. ✅ Handle errors properly in asynchronous operations
3. ✅ Use environment variables for configuration
4. ✅ Avoid synchronous operations in loops
5. ✅ Use `.gitignore` to exclude `node_modules`
6. ✅ Keep `package-lock.json` in version control

