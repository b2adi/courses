# Advanced Node.js Topics

## Table of Contents
1. [Clustering](#clustering)
2. [Child Processes](#child-processes)
3. [Worker Threads](#worker-threads)
4. [Memory Management](#memory-management)
5. [Performance Optimization](#performance-optimization)

---

## Clustering

Clustering allows you to utilize multiple CPU cores.

### Basic Cluster Setup
```javascript
const cluster = require('cluster');
const os = require('os');
const http = require('http');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;

  console.log(`Master process ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // Handle worker exit
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Restart worker
  });
} else {
  // Worker processes
  const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Handled by worker ${process.pid}\n`);
  });

  server.listen(3000);
  console.log(`Worker ${process.pid} started`);
}
```

### Load Balancing with Cluster
```javascript
const cluster = require('cluster');
const os = require('os');
const http = require('http');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;

  // Fork workers equal to number of CPUs
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // Master handles signals
  process.on('SIGTERM', () => {
    console.log('Graceful shutdown');
    for (const id in cluster.workers) {
      cluster.workers[id].kill();
    }
    process.exit(0);
  });

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} exited, starting new worker`);
    cluster.fork();
  });
} else {
  // Each worker runs the app
  const app = require('./app'); // Express app
  app.listen(3000);
}
```

### Sharing State Between Workers
```javascript
const cluster = require('cluster');
const net = require('net');

if (cluster.isMaster) {
  const workers = [];
  
  for (let i = 0; i < 2; i++) {
    workers.push(cluster.fork());
  }

  // Master receives messages from workers
  cluster.on('message', (worker, message) => {
    console.log(`Master received: ${message}`);
    
    // Broadcast to other workers
    workers.forEach(w => {
      if (w.process.pid !== worker.process.pid) {
        w.send(message);
      }
    });
  });
} else {
  // Worker sends message to master
  process.send(`Worker ${process.pid} ready`);

  // Worker receives message from master
  process.on('message', (message) => {
    console.log(`Worker ${process.pid} received: ${message}`);
  });
}
```

---

## Child Processes

Execute external programs or run heavy computations separately.

### spawn() - Stream-based
```javascript
const { spawn } = require('child_process');

const ls = spawn('ls', ['-la']);

ls.stdout.on('data', (data) => {
  console.log(`Output: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`Error: ${data}`);
});

ls.on('close', (code) => {
  console.log(`Process exited with code ${code}`);
});
```

### exec() - Buffer-based
```javascript
const { exec } = require('child_process');

exec('ls -la', (error, stdout, stderr) => {
  if (error) {
    console.error(`Error: ${error.message}`);
    return;
  }
  console.log(`Output:\n${stdout}`);
});
```

### execFile() - More secure
```javascript
const { execFile } = require('child_process');

execFile('node', ['script.js', 'arg1'], (error, stdout, stderr) => {
  if (error) throw error;
  console.log(stdout);
});
```

### fork() - Node.js processes
```javascript
const { fork } = require('child_process');

// Parent process
const child = fork('child.js');

child.on('message', (message) => {
  console.log('Parent received:', message);
});

child.send({ hello: 'world' });

// Child process (child.js)
process.on('message', (message) => {
  console.log('Child received:', message);
  process.send({ response: 'hi from child' });
});
```

### Heavy Computation with Fork
```javascript
// main.js
const { fork } = require('child_process');

function calculate(number) {
  return new Promise((resolve) => {
    const child = fork('worker.js');

    child.on('message', (result) => {
      resolve(result);
      child.kill();
    });

    child.send(number);
  });
}

calculate(100000).then(result => {
  console.log('Result:', result);
});

// worker.js
process.on('message', (number) => {
  // Heavy computation
  let sum = 0;
  for (let i = 0; i < number; i++) {
    sum += i;
  }
  process.send(sum);
});
```

---

## Worker Threads

True parallelism within same process.

### Basic Worker Thread
```javascript
const { Worker } = require('worker_threads');

const worker = new Worker('./worker.js');

worker.on('message', (message) => {
  console.log('Received from worker:', message);
});

worker.on('error', (error) => {
  console.error('Worker error:', error);
});

worker.on('exit', (code) => {
  if (code !== 0) {
    console.error(`Worker stopped with exit code ${code}`);
  }
});

worker.postMessage({ number: 100 });
```

### Worker File
```javascript
// worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', (message) => {
  const result = fibonacci(message.number);
  parentPort.postMessage({ result });
});

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}
```

### Worker Pool
```javascript
const { Worker } = require('worker_threads');
const path = require('path');

class WorkerPool {
  constructor(size = 4) {
    this.workers = [];
    this.queue = [];

    for (let i = 0; i < size; i++) {
      const worker = new Worker(path.join(__dirname, 'worker.js'));
      worker.available = true;
      worker.on('message', () => {
        this.handleWorkerMessage(worker);
      });
      this.workers.push(worker);
    }
  }

  run(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };

      const available = this.workers.find(w => w.available);
      if (available) {
        this.execute(task, available);
      } else {
        this.queue.push(task);
      }
    });
  }

  execute(task, worker) {
    worker.available = false;
    worker.once('message', (result) => {
      task.resolve(result);
    });
    worker.postMessage(task.data);
  }

  handleWorkerMessage(worker) {
    worker.available = true;
    if (this.queue.length > 0) {
      const task = this.queue.shift();
      this.execute(task, worker);
    }
  }
}

module.exports = WorkerPool;
```

---

## Memory Management

### Memory Monitoring
```javascript
const formatBytes = (bytes) => {
  return `${Math.round(bytes / 1024 / 1024)} MB`;
};

setInterval(() => {
  const memUsage = process.memoryUsage();
  console.log('Memory Usage:');
  console.log(`  RSS: ${formatBytes(memUsage.rss)}`);           // Resident Set Size
  console.log(`  Heap Total: ${formatBytes(memUsage.heapTotal)}`);
  console.log(`  Heap Used: ${formatBytes(memUsage.heapUsed)}`);
  console.log(`  External: ${formatBytes(memUsage.external)}`);
}, 10000);
```

### Preventing Memory Leaks
```javascript
// ❌ Memory leak - listeners not removed
class EventEmitter {
  on(event, callback) {
    if (!this.listeners) this.listeners = [];
    this.listeners.push(callback);
  }
}

// ✅ Good - removes listeners
emitter.removeAllListeners();
emitter.off('event', handler);
```

### Garbage Collection
```javascript
if (global.gc) {
  global.gc();
  console.log('Garbage collection triggered');
}

// Run with: node --expose-gc app.js
```

### Memory Profiling
```javascript
const v8 = require('v8');
const fs = require('fs');

// Take heap snapshot
const snapshot = v8.writeHeapSnapshot();
console.log(`Heap snapshot written to ${snapshot}`);

// Heap statistics
const heapStats = v8.getHeapStatistics();
console.log('Heap Statistics:');
console.log(`  Total Heap Size: ${heapStats.total_heap_size}`);
console.log(`  Executable Size: ${heapStats.total_heap_execute_size}`);
```

---

## Performance Optimization

### Caching
```javascript
const cache = new Map();

function fibonacci(n) {
  if (cache.has(n)) return cache.get(n);

  if (n <= 1) return n;
  const result = fibonacci(n - 1) + fibonacci(n - 2);

  cache.set(n, result);
  return result;
}
```

### LRU Cache
```javascript
class LRUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return undefined;

    // Move to end (most recently used)
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);

    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }

    this.cache.set(key, value);

    if (this.cache.size > this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
}
```

### Profiling
```javascript
console.time('operation');

// Code to measure
for (let i = 0; i < 1000000; i++) {
  Math.sqrt(i);
}

console.timeEnd('operation');
```

### Streaming Large Files
```javascript
const fs = require('fs');

// ❌ Inefficient - loads entire file
const data = fs.readFileSync('large-file.txt');

// ✅ Efficient - streams data
fs.createReadStream('large-file.txt')
  .pipe(process.stdout);
```

### Async Operations Batching
```javascript
async function processBatch(items, batchSize = 10) {
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    await Promise.all(batch.map(item => processItem(item)));
  }
}
```

---

## Best Practices

1. ✅ Use clustering for CPU-bound tasks
2. ✅ Use worker threads for heavy computations
3. ✅ Monitor memory usage
4. ✅ Implement proper error handling
5. ✅ Avoid memory leaks by cleaning up resources
6. ✅ Use connection pooling for databases
7. ✅ Cache frequently accessed data
8. ✅ Profile your application
9. ✅ Use CDN for static files
10. ✅ Implement graceful shutdown
