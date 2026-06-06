# Event Emitters & Streams in Node.js

## Table of Contents
1. [Event Emitters](#event-emitters)
2. [Readable Streams](#readable-streams)
3. [Writable Streams](#writable-streams)
4. [Transform Streams](#transform-streams)
5. [Pipe Operations](#pipe-operations)

---

## Event Emitters

Event Emitters are the foundation of Node.js event-driven architecture.

### Creating Event Emitters
```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

// Listen for event
myEmitter.on('event', (arg) => {
  console.log('Event fired:', arg);
});

// Emit event
myEmitter.emit('event', 'some data');
```

### Common Methods

#### on() - Multiple Listeners
```javascript
emitter.on('data', (chunk) => console.log('Listener 1:', chunk));
emitter.on('data', (chunk) => console.log('Listener 2:', chunk));

emitter.emit('data', 'test');
// Output:
// Listener 1: test
// Listener 2: test
```

#### once() - Single Listener
```javascript
emitter.once('connect', () => {
  console.log('Connected once');
});

emitter.emit('connect'); // Output: Connected once
emitter.emit('connect'); // No output
```

#### off() - Remove Listener
```javascript
const handler = (data) => console.log(data);

emitter.on('message', handler);
emitter.emit('message', 'Hello'); // Output: Hello

emitter.off('message', handler);
emitter.emit('message', 'World'); // No output
```

#### removeAllListeners()
```javascript
emitter.removeAllListeners('event');
// or remove all events
emitter.removeAllListeners();
```

### Listener Management
```javascript
emitter.listenerCount('event');  // Get listener count
emitter.listeners('event');      // Get all listeners
emitter.eventNames();            // Get all event names
```

---

## Readable Streams

Streams allow processing large amounts of data in chunks.

### Basic Readable Stream
```javascript
const fs = require('fs');

const readStream = fs.createReadStream('large-file.txt', {
  encoding: 'utf-8',
  highWaterMark: 16 * 1024  // 16KB chunks
});

readStream.on('data', (chunk) => {
  console.log('Chunk received:', chunk.length, 'bytes');
});

readStream.on('end', () => {
  console.log('Stream ended');
});

readStream.on('error', (error) => {
  console.error('Stream error:', error);
});
```

### Stream Events
```javascript
readStream.on('readable', () => {
  let chunk;
  while ((chunk = readStream.read()) !== null) {
    console.log('Read chunk:', chunk);
  }
});

readStream.on('pause', () => console.log('Paused'));
readStream.on('resume', () => console.log('Resumed'));
```

### Controlling Backpressure
```javascript
const readStream = fs.createReadStream('file.txt');

readStream.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
  
  // Pause stream if buffer is getting full
  readStream.pause();
  
  // Process data...
  setTimeout(() => {
    readStream.resume();
  }, 1000);
});
```

### Creating Custom Readable Stream
```javascript
const { Readable } = require('stream');

class CountStream extends Readable {
  constructor(options) {
    super(options);
    this.count = 1;
  }

  _read() {
    if (this.count > 5) {
      this.push(null); // Signal end
    } else {
      this.push(`Count: ${this.count}\n`);
      this.count++;
    }
  }
}

const stream = new CountStream();
stream.pipe(process.stdout);
```

---

## Writable Streams

### Basic Writable Stream
```javascript
const fs = require('fs');

const writeStream = fs.createWriteStream('output.txt');

writeStream.write('Hello, ');
writeStream.write('World!');
writeStream.write('\nEnd of file');

writeStream.end();

writeStream.on('finish', () => {
  console.log('All data written');
});

writeStream.on('error', (error) => {
  console.error('Write error:', error);
});
```

### Handling Backpressure in Write
```javascript
const writeStream = fs.createWriteStream('output.txt');

function write(data) {
  const canContinue = writeStream.write(data);
  
  if (!canContinue) {
    console.log('Buffer full, pausing...');
    writeStream.once('drain', () => {
      console.log('Buffer drained, resuming...');
    });
    return false;
  }
  return true;
}

write('Some data\n');
write('More data\n');

writeStream.end();
```

### Creating Custom Writable Stream
```javascript
const { Writable } = require('stream');

class LoggerStream extends Writable {
  _write(chunk, encoding, callback) {
    console.log(`[LOG] ${chunk.toString().trim()}`);
    callback();
  }
}

const logger = new LoggerStream();
logger.write('Important message');
logger.end();
```

---

## Transform Streams

Transform Streams modify data as it passes through.

### Basic Transform Stream
```javascript
const { Transform } = require('stream');

const uppercaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

process.stdin.pipe(uppercaseTransform).pipe(process.stdout);
```

### CSV to JSON Transform
```javascript
const { Transform } = require('stream');

class CSVToJSON extends Transform {
  constructor(options) {
    super(options);
    this.headers = null;
  }

  _transform(chunk, encoding, callback) {
    const lines = chunk.toString().split('\n');
    
    if (!this.headers) {
      this.headers = lines[0].split(',');
      lines.shift();
    }

    lines.forEach(line => {
      if (line.trim()) {
        const values = line.split(',');
        const obj = {};
        this.headers.forEach((header, i) => {
          obj[header] = values[i];
        });
        this.push(JSON.stringify(obj) + '\n');
      }
    });

    callback();
  }
}

const csvToJson = new CSVToJSON();
process.stdin.pipe(csvToJson).pipe(process.stdout);
```

### Compression Transform
```javascript
const fs = require('fs');
const zlib = require('zlib');

fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'))
  .on('finish', () => console.log('File compressed'));
```

---

## Pipe Operations

### Basic Pipe
```javascript
const fs = require('fs');

// Simple copy
fs.createReadStream('source.txt')
  .pipe(fs.createWriteStream('destination.txt'));
```

### Chaining Multiple Pipes
```javascript
const fs = require('fs');
const zlib = require('zlib');

fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'))
  .on('finish', () => console.log('Compressed'));
```

### Error Handling in Pipes
```javascript
const fs = require('fs');

fs.createReadStream('file.txt')
  .on('error', (error) => console.error('Read error:', error))
  .pipe(fs.createWriteStream('output.txt'))
  .on('error', (error) => console.error('Write error:', error))
  .on('finish', () => console.log('Done'));
```

### Piping to Multiple Destinations
```javascript
const fs = require('fs');

const source = fs.createReadStream('input.txt');
const dest1 = fs.createWriteStream('copy1.txt');
const dest2 = fs.createWriteStream('copy2.txt');

source.pipe(dest1);
source.pipe(dest2);
```

### Pump for Better Error Handling
```javascript
const fs = require('fs');
const { pipeline } = require('stream');

pipeline(
  fs.createReadStream('input.txt'),
  fs.createWriteStream('output.txt'),
  (err) => {
    if (err) console.error('Pipeline failed:', err);
    else console.log('Pipeline succeeded');
  }
);
```

---

## Performance Comparison

### Without Streams (Memory Inefficient)
```javascript
// ❌ Loads entire file into memory
const fs = require('fs');
const data = fs.readFileSync('large-file.txt');
process.stdout.write(data);
```

### With Streams (Memory Efficient)
```javascript
// ✅ Processes in chunks
const fs = require('fs');
fs.createReadStream('large-file.txt').pipe(process.stdout);
```

---

## Best Practices

1. ✅ Always handle 'error' events on streams
2. ✅ Use `pipeline()` instead of `.pipe()` for better error handling
3. ✅ Respect backpressure by checking return value of `write()`
4. ✅ Use highWaterMark to tune memory usage
5. ✅ Close streams with `.end()` or `.destroy()`
6. ✅ Use Transform streams to modify data efficiently

