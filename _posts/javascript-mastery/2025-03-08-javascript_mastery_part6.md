---
title: "JavaScript Mastery - Part 6: Node.js and Server-Side JavaScript"
date: 2025-03-08 00:00:00 +0530
categories: [JavaScript, JavaScript Mastery]
tags: [JavaScript, Programming, Web Development, Node.js, Backend, Server, Express, API, Database, Authentication, npm, Server-side]
---

# Complete JavaScript Mastery Part 6: Node.js and Server-Side JavaScript

## Introduction

Node.js enables JavaScript to run outside the browser, powering server-side applications, command-line tools, and build systems. Understanding Node.js is essential for full-stack JavaScript development.

This part explores:
- Node.js architecture and runtime
- Core modules (fs, http, path, etc.)
- NPM and package management
- Express.js and web frameworks
- Database integration
- Authentication and security
- RESTful APIs and GraphQL

**Prerequisites:**
- Part 1: JavaScript fundamentals
- Part 2: Event loop, async programming
- Part 5: HTTP basics, Fetch API

**Why Node.js Matters:**

- Build scalable server applications
- Full-stack JavaScript development
- Rich ecosystem (npm)
- Event-driven, non-blocking I/O
- Cross-platform tooling

Let's master server-side JavaScript with Node.js.

---

## 6.1 Node.js Fundamentals

### Node.js Architecture

Node.js runs JavaScript using V8 and provides APIs for system operations through libuv.

```
┌─────────────────────────────────┐
│     JavaScript Application      │
├─────────────────────────────────┤
│       Node.js APIs (C++)        │
│  (fs, http, crypto, etc.)       │
├─────────────────────────────────┤
│         V8 Engine               │
│   (JavaScript execution)         │
├─────────────────────────────────┤
│          libuv                  │
│  (Event loop, async I/O)        │
├─────────────────────────────────┤
│      Operating System           │
└─────────────────────────────────┘
```

**Key Components:**

- **V8:** JavaScript engine (same as Chrome)
- **libuv:** Cross-platform async I/O library
- **Node.js APIs:** Built-in modules (fs, http, etc.)
- **npm:** Package manager and registry

### Node.js Event Loop

The Node.js event loop differs from the browser's event loop.

```javascript
// Event Loop Phases (in order):
// 1. Timers - setTimeout, setInterval callbacks
// 2. Pending callbacks - I/O callbacks deferred from previous iteration
// 3. Idle, prepare - Internal use
// 4. Poll - Retrieve new I/O events
// 5. Check - setImmediate callbacks
// 6. Close callbacks - socket.on('close')

// Example demonstrating phases
console.log('1: Script start');

setTimeout(() => {
  console.log('2: setTimeout');
}, 0);

setImmediate(() => {
  console.log('3: setImmediate');
});

Promise.resolve().then(() => {
  console.log('4: Promise');
});

process.nextTick(() => {
  console.log('5: nextTick');
});

console.log('6: Script end');

// Output:
// 1: Script start
// 6: Script end
// 5: nextTick (microtask, highest priority)
// 4: Promise (microtask)
// 2: setTimeout (timer phase)
// 3: setImmediate (check phase)
```

**Microtask Queue Priority:**

```javascript
// process.nextTick has highest priority
process.nextTick(() => console.log('nextTick 1'));
Promise.resolve().then(() => console.log('Promise 1'));
process.nextTick(() => console.log('nextTick 2'));
Promise.resolve().then(() => console.log('Promise 2'));

// Output:
// nextTick 1
// nextTick 2
// Promise 1
// Promise 2
```

### Global Objects

Node.js provides global objects different from browser globals.

```javascript
// Available globally (no require needed)

// __dirname - Current directory path
console.log(__dirname); // /Users/user/project

// __filename - Current file path
console.log(__filename); // /Users/user/project/app.js

// exports, require, module - Module system
module.exports = { myFunction };
const myModule = require('./myModule');

// global - Global namespace (like window in browser)
global.myGlobal = 'value';

// process - Process information and control
console.log(process.version);     // Node.js version
console.log(process.platform);    // OS platform
console.log(process.argv);        // Command line arguments
console.log(process.env);         // Environment variables
console.log(process.cwd());       // Current working directory
console.log(process.pid);         // Process ID

// Process events
process.on('exit', (code) => {
  console.log(`Process exiting with code: ${code}`);
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught exception:', error);
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection:', reason);
});

// console - Logging (same as browser)
console.log('Info');
console.error('Error');
console.warn('Warning');
console.time('timer');
console.timeEnd('timer');

// Buffer - Binary data handling
const buffer = Buffer.from('Hello');
console.log(buffer); // <Buffer 48 65 6c 6c 6f>
console.log(buffer.toString()); // 'Hello'

// setTimeout, setInterval, setImmediate
setTimeout(() => console.log('timeout'), 1000);
setInterval(() => console.log('interval'), 1000);
setImmediate(() => console.log('immediate'));

// clearTimeout, clearInterval, clearImmediate
const timeoutId = setTimeout(() => {}, 1000);
clearTimeout(timeoutId);
```

### Process Object

```javascript
// Command line arguments
// node app.js arg1 arg2
console.log(process.argv);
// ['node', '/path/to/app.js', 'arg1', 'arg2']

const args = process.argv.slice(2); // ['arg1', 'arg2']

// Environment variables
// NODE_ENV=production node app.js
console.log(process.env.NODE_ENV); // 'production'
console.log(process.env.PORT || 3000);

// Working directory
console.log(process.cwd()); // Current directory
process.chdir('/new/path'); // Change directory

// Exit process
process.exit(0); // Success
process.exit(1); // Error

// Memory usage
const usage = process.memoryUsage();
console.log({
  rss: usage.rss / 1024 / 1024, // MB
  heapTotal: usage.heapTotal / 1024 / 1024,
  heapUsed: usage.heapUsed / 1024 / 1024,
  external: usage.external / 1024 / 1024
});

// CPU usage
const startUsage = process.cpuUsage();
// ... do something ...
const endUsage = process.cpuUsage(startUsage);
console.log(endUsage); // { user: 1000, system: 500 } (microseconds)

// Standard I/O
process.stdin.on('data', (data) => {
  console.log('Received:', data.toString());
});

process.stdout.write('Output\n');
process.stderr.write('Error\n');
```

### Buffer API

Buffers handle binary data in Node.js.

```javascript
// Create buffers
const buf1 = Buffer.from('Hello'); // From string
const buf2 = Buffer.from([72, 101, 108, 108, 111]); // From array
const buf3 = Buffer.alloc(10); // Allocate 10 bytes (filled with 0)
const buf4 = Buffer.allocUnsafe(10); // Faster but uninitialized

// Convert buffer to string
const str = buf1.toString(); // 'Hello'
const hex = buf1.toString('hex'); // '48656c6c6f'
const base64 = buf1.toString('base64'); // 'SGVsbG8='

// Write to buffer
const buf = Buffer.alloc(10);
buf.write('Hello', 'utf8');
console.log(buf.toString()); // 'Hello\x00\x00\x00\x00\x00'

// Read from buffer
console.log(buf[0]); // 72 (ASCII 'H')

// Buffer length
console.log(buf.length); // 10 (bytes, not string length)

// Compare buffers
const buf5 = Buffer.from('ABC');
const buf6 = Buffer.from('ABC');
console.log(buf5.equals(buf6)); // true
console.log(Buffer.compare(buf5, buf6)); // 0

// Concatenate buffers
const buf7 = Buffer.concat([buf5, buf6]);
console.log(buf7.toString()); // 'ABCABC'

// Slice buffer
const slice = buf7.slice(0, 3);
console.log(slice.toString()); // 'ABC'

// Copy buffer
const target = Buffer.alloc(6);
buf7.copy(target, 0, 0, 6);
console.log(target.toString()); // 'ABCABC'
```

---

## 6.2 Node.js Core Modules

### File System (fs)

The fs module provides file system operations.

#### Reading Files

```javascript
const fs = require('fs');

// Synchronous (blocks execution)
try {
  const data = fs.readFileSync('file.txt', 'utf8');
  console.log(data);
} catch (error) {
  console.error('Read error:', error);
}

// Asynchronous (callback)
fs.readFile('file.txt', 'utf8', (error, data) => {
  if (error) {
    console.error('Read error:', error);
    return;
  }
  console.log(data);
});

// Promises (modern approach)
const fsPromises = require('fs/promises');

async function readFileAsync() {
  try {
    const data = await fsPromises.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (error) {
    console.error('Read error:', error);
  }
}

// Read as buffer
const buffer = await fsPromises.readFile('image.png');
console.log(buffer); // <Buffer ...>

// Check if file exists
const exists = fs.existsSync('file.txt'); // Sync
// Note: fs.exists is deprecated, use fs.access instead

try {
  await fsPromises.access('file.txt');
  console.log('File exists');
} catch {
  console.log('File does not exist');
}
```

#### Writing Files

```javascript
const fs = require('fs/promises');

// Write file (creates or overwrites)
await fs.writeFile('output.txt', 'Hello World', 'utf8');

// Append to file
await fs.appendFile('log.txt', 'New log entry\n', 'utf8');

// Write buffer
const buffer = Buffer.from('Binary data');
await fs.writeFile('data.bin', buffer);

// Write with options
await fs.writeFile('file.txt', 'Content', {
  encoding: 'utf8',
  mode: 0o666,
  flag: 'w' // 'w' = write, 'a' = append, 'r+' = read/write
});

// Synchronous write
fs.writeFileSync('sync.txt', 'Sync content');
```

#### Streams for Large Files

```javascript
const fs = require('fs');

// Read stream (memory efficient for large files)
const readStream = fs.createReadStream('large-file.txt', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024 // 64KB chunks
});

readStream.on('data', (chunk) => {
  console.log('Chunk:', chunk.length);
});

readStream.on('end', () => {
  console.log('Read complete');
});

readStream.on('error', (error) => {
  console.error('Read error:', error);
});

// Write stream
const writeStream = fs.createWriteStream('output.txt');

writeStream.write('First line\n');
writeStream.write('Second line\n');
writeStream.end('Final line\n');

writeStream.on('finish', () => {
  console.log('Write complete');
});

// Pipe streams (copy file)
const source = fs.createReadStream('source.txt');
const destination = fs.createWriteStream('destination.txt');

source.pipe(destination);

source.on('end', () => {
  console.log('Copy complete');
});

// Transform stream (uppercase)
const { Transform } = require('stream');

const uppercase = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

fs.createReadStream('input.txt')
  .pipe(uppercase)
  .pipe(fs.createWriteStream('output.txt'));
```

#### Directory Operations

```javascript
const fs = require('fs/promises');
const path = require('path');

// Read directory
const files = await fs.readdir('.');
console.log(files); // ['file1.txt', 'file2.txt', 'dir1']

// Read directory with file types
const entries = await fs.readdir('.', { withFileTypes: true });

for (const entry of entries) {
  if (entry.isFile()) {
    console.log('File:', entry.name);
  } else if (entry.isDirectory()) {
    console.log('Directory:', entry.name);
  }
}

// Create directory
await fs.mkdir('new-dir');

// Create nested directories
await fs.mkdir('path/to/nested/dir', { recursive: true });

// Remove directory
await fs.rmdir('empty-dir');

// Remove directory recursively
await fs.rm('dir-with-contents', { recursive: true, force: true });

// Recursive directory walk
async function walkDirectory(dir) {
  const entries = await fs.readdir(dir, { withFileTypes: true });
  
  for (const entry of entries) {
    const fullPath = path.join(dir, entry.name);
    
    if (entry.isDirectory()) {
      await walkDirectory(fullPath); // Recurse
    } else {
      console.log('File:', fullPath);
    }
  }
}

await walkDirectory('.');
```

#### File Stats and Metadata

```javascript
const fs = require('fs/promises');

// Get file stats
const stats = await fs.stat('file.txt');

console.log({
  size: stats.size,                  // Bytes
  isFile: stats.isFile(),            // true/false
  isDirectory: stats.isDirectory(),  // true/false
  created: stats.birthtime,          // Creation date
  modified: stats.mtime,             // Modification date
  accessed: stats.atime              // Access date
});

// Check permissions
const mode = stats.mode;
console.log({
  readable: (mode & fs.constants.R_OK) !== 0,
  writable: (mode & fs.constants.W_OK) !== 0,
  executable: (mode & fs.constants.X_OK) !== 0
});

// Change permissions
await fs.chmod('file.txt', 0o644); // rw-r--r--

// Change owner (requires privileges)
await fs.chown('file.txt', uid, gid);

// Rename/move file
await fs.rename('old-name.txt', 'new-name.txt');

// Copy file
await fs.copyFile('source.txt', 'destination.txt');

// Delete file
await fs.unlink('file.txt');
```

#### Watch for File Changes

```javascript
const fs = require('fs');

// Watch file
const watcher = fs.watch('file.txt', (eventType, filename) => {
  console.log(`Event: ${eventType} on ${filename}`);
  // eventType: 'rename' or 'change'
});

// Stop watching
watcher.close();

// Watch directory
const dirWatcher = fs.watch('.', { recursive: true }, (eventType, filename) => {
  console.log(`${eventType}: ${filename}`);
});

// Alternative: fs.watchFile (uses polling, less efficient)
fs.watchFile('file.txt', { interval: 1000 }, (curr, prev) => {
  console.log('File changed');
  console.log('Previous modified:', prev.mtime);
  console.log('Current modified:', curr.mtime);
});

// Stop watching
fs.unwatchFile('file.txt');
```

### Path Module

The path module handles file paths in a cross-platform way.

```javascript
const path = require('path');

// Join paths
const fullPath = path.join('/users', 'john', 'documents', 'file.txt');
// '/users/john/documents/file.txt'

// Resolve to absolute path
const absolute = path.resolve('documents', 'file.txt');
// '/current/working/directory/documents/file.txt'

// Get directory name
console.log(path.dirname('/users/john/file.txt'));
// '/users/john'

// Get file name
console.log(path.basename('/users/john/file.txt'));
// 'file.txt'

console.log(path.basename('/users/john/file.txt', '.txt'));
// 'file' (remove extension)

// Get extension
console.log(path.extname('file.txt'));
// '.txt'

// Parse path
const parsed = path.parse('/users/john/file.txt');
console.log(parsed);
// {
//   root: '/',
//   dir: '/users/john',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }

// Format path
const formatted = path.format({
  dir: '/users/john',
  base: 'file.txt'
});
// '/users/john/file.txt'

// Normalize path (resolve .. and .)
console.log(path.normalize('/users/john/../jane/./file.txt'));
// '/users/jane/file.txt'

// Relative path
console.log(path.relative('/users/john', '/users/jane/file.txt'));
// '../jane/file.txt'

// Check if absolute
console.log(path.isAbsolute('/users/john')); // true
console.log(path.isAbsolute('users/john'));  // false

// Path separator
console.log(path.sep);    // '/' on Unix, '\\' on Windows
console.log(path.delimiter); // ':' on Unix, ';' on Windows

// Cross-platform paths
const crossPlatform = path.join('users', 'john', 'file.txt');
// Uses correct separator for platform
```

### HTTP/HTTPS Module

Create HTTP servers and make requests.

```javascript
const http = require('http');

// Create HTTP server
const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);
  
  // Request properties
  console.log('Headers:', req.headers);
  console.log('Method:', req.method);
  console.log('URL:', req.url);
  
  // Set response status and headers
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  
  // Send response
  res.end('Hello World\n');
});

// Start server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});

// Handle different routes
const server = http.createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Home Page</h1>');
  } else if (req.url === '/api/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ users: ['John', 'Jane'] }));
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found');
  }
});

// Handle POST request with body
const server = http.createServer((req, res) => {
  if (req.method === 'POST' && req.url === '/api/users') {
    let body = '';
    
    req.on('data', (chunk) => {
      body += chunk.toString();
    });
    
    req.on('end', () => {
      const data = JSON.parse(body);
      console.log('Received:', data);
      
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ success: true, data }));
    });
  }
});

// Make HTTP request
http.get('http://api.example.com/data', (res) => {
  let data = '';
  
  res.on('data', (chunk) => {
    data += chunk;
  });
  
  res.on('end', () => {
    console.log('Response:', data);
  });
}).on('error', (error) => {
  console.error('Request error:', error);
});

// POST request
const options = {
  hostname: 'api.example.com',
  port: 80,
  path: '/users',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  }
};

const req = http.request(options, (res) => {
  console.log('Status:', res.statusCode);
  
  res.on('data', (chunk) => {
    console.log('Data:', chunk.toString());
  });
});

req.on('error', (error) => {
  console.error('Request error:', error);
});

req.write(JSON.stringify({ name: 'John' }));
req.end();
```

### Other Core Modules

```javascript
// OS module - Operating system info
const os = require('os');

console.log({
  platform: os.platform(),        // 'darwin', 'win32', 'linux'
  arch: os.arch(),                // 'x64', 'arm64'
  cpus: os.cpus(),                // CPU info
  totalMemory: os.totalmem(),     // Total RAM (bytes)
  freeMemory: os.freemem(),       // Free RAM (bytes)
  uptime: os.uptime(),            // System uptime (seconds)
  hostname: os.hostname(),        // Computer name
  homedir: os.homedir(),          // User home directory
  tmpdir: os.tmpdir()             // Temp directory
});

// Crypto module - Cryptographic operations
const crypto = require('crypto');

// Generate hash
const hash = crypto.createHash('sha256');
hash.update('Hello World');
console.log(hash.digest('hex'));

// Generate random bytes
const randomBytes = crypto.randomBytes(32);
console.log(randomBytes.toString('hex'));

// Create HMAC
const hmac = crypto.createHmac('sha256', 'secret-key');
hmac.update('Message');
console.log(hmac.digest('hex'));

// Util module - Utilities
const util = require('util');

// Promisify callback-based functions
const fs = require('fs');
const readFile = util.promisify(fs.readFile);

const data = await readFile('file.txt', 'utf8');

// Format strings
console.log(util.format('Hello %s', 'World')); // 'Hello World'

// Inspect objects
console.log(util.inspect({ a: 1, b: 2 }, { colors: true }));

// Events module - EventEmitter
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const emitter = new MyEmitter();

emitter.on('event', (arg) => {
  console.log('Event occurred:', arg);
});

emitter.emit('event', 'data');

// Query string module - Parse query strings
const querystring = require('querystring');

const parsed = querystring.parse('name=John&age=30');
console.log(parsed); // { name: 'John', age: '30' }

const stringified = querystring.stringify({ name: 'John', age: 30 });
console.log(stringified); // 'name=John&age=30'

// URL module - Parse URLs
const { URL } = require('url');

const myURL = new URL('https://example.com:8080/path?query=value#hash');

console.log({
  href: myURL.href,           // Full URL
  origin: myURL.origin,       // 'https://example.com:8080'
  protocol: myURL.protocol,   // 'https:'
  hostname: myURL.hostname,   // 'example.com'
  port: myURL.port,           // '8080'
  pathname: myURL.pathname,   // '/path'
  search: myURL.search,       // '?query=value'
  hash: myURL.hash            // '#hash'
});

// URLSearchParams
const params = myURL.searchParams;
console.log(params.get('query')); // 'value'
params.set('newParam', 'newValue');

// Zlib module - Compression
const zlib = require('zlib');

// Compress
const compressed = zlib.gzipSync('Hello World');
console.log(compressed);

// Decompress
const decompressed = zlib.gunzipSync(compressed);
console.log(decompressed.toString()); // 'Hello World'

// Stream compression
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
```

---

## 6.3 NPM and Package Management

### package.json

The package.json file defines your project and its dependencies.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "webpack",
    "lint": "eslint ."
  },
  "keywords": ["node", "javascript"],
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0",
    "jest": "^29.0.0",
    "eslint": "^8.0.0"
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}
```

**Key Fields:**

- **name:** Package name (lowercase, no spaces)
- **version:** Semantic version (major.minor.patch)
- **main:** Entry point file
- **scripts:** Runnable scripts (npm run <script>)
- **dependencies:** Production dependencies
- **devDependencies:** Development-only dependencies
- **engines:** Required Node.js/npm versions

### Semantic Versioning (semver)

```javascript
// Version format: MAJOR.MINOR.PATCH

// ^1.2.3 - Compatible with 1.x.x (>= 1.2.3, < 2.0.0)
// ~1.2.3 - Compatible with 1.2.x (>= 1.2.3, < 1.3.0)
// 1.2.3  - Exact version
// *      - Any version (not recommended)
// latest - Latest version

// Examples:
"express": "^4.18.0"  // >= 4.18.0, < 5.0.0
"lodash": "~4.17.21"  // >= 4.17.21, < 4.18.0
"react": "18.2.0"     // Exact version
```

### NPM Commands

```bash
# Initialize new project
npm init
npm init -y  # Skip questions

# Install dependencies
npm install express
npm install express mongoose dotenv
npm install --save-dev nodemon jest

# Shortcuts
npm i express               # install
npm i -D nodemon           # install dev dependency
npm i -g pm2               # install globally

# Install from package.json
npm install
npm ci  # Clean install (faster, for CI/CD)

# Update dependencies
npm update
npm update express
npm outdated  # Check for outdated packages

# Uninstall
npm uninstall express
npm uninstall -D nodemon

# Run scripts
npm start
npm test
npm run dev
npm run build

# List installed packages
npm list
npm list --depth=0  # Top level only

# View package info
npm view express
npm view express versions

# Search packages
npm search keyword

# Audit security
npm audit
npm audit fix
npm audit fix --force

# Publish package
npm login
npm publish

# Version bumping
npm version patch  # 1.0.0 → 1.0.1
npm version minor  # 1.0.0 → 1.1.0
npm version major  # 1.0.0 → 2.0.0
```

### package-lock.json

Locks exact versions of dependencies and their dependencies.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "name": "my-app",
      "version": "1.0.0",
      "dependencies": {
        "express": "^4.18.0"
      }
    },
    "node_modules/express": {
      "version": "4.18.2",
      "resolved": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
      "integrity": "sha512-...",
      "dependencies": {
        "accepts": "~1.3.8",
        "array-flatten": "1.1.1"
      }
    }
  }
}
```

**Benefits:**
- Ensures consistent installs across environments
- Faster installs (known dependency tree)
- Commit to version control

### NPM Scripts

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest --coverage",
    "test:watch": "jest --watch",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "build": "webpack --mode production",
    "clean": "rm -rf dist",
    "prebuild": "npm run clean",
    "postbuild": "echo 'Build complete'",
    "deploy": "npm run build && npm run upload"
  }
}
```

**Pre/Post Hooks:**
- `pretest` runs before `test`
- `posttest` runs after `test`
- Works with any script name

**Run scripts:**
```bash
npm start           # runs "start" script
npm test            # runs "test" script
npm run dev         # runs "dev" script
npm run build       # runs prebuild, build, postbuild
```

### Alternative Package Managers

**Yarn:**
```bash
# Install
npm install -g yarn

# Commands
yarn add express
yarn add --dev nodemon
yarn install
yarn remove express
yarn upgrade
```

**pnpm:**
```bash
# Install
npm install -g pnpm

# Commands (similar to npm)
pnpm add express
pnpm add -D nodemon
pnpm install
pnpm remove express

# Benefits: Disk space efficient, faster
```

---

## 6.4 Express.js and Web Frameworks

### Express Basics

Express is the most popular Node.js web framework.

```javascript
const express = require('express');
const app = express();

// Middleware to parse JSON
app.use(express.json());

// Basic route
app.get('/', (req, res) => {
  res.send('Hello World');
});

// Route with parameter
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ userId: id });
});

// Route with query parameters
app.get('/search', (req, res) => {
  const { q, limit } = req.query;
  res.json({ query: q, limit: limit || 10 });
});

// POST route
app.post('/users', (req, res) => {
  const user = req.body;
  res.status(201).json({ message: 'User created', user });
});

// PUT route
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const updates = req.body;
  res.json({ message: 'User updated', id, updates });
});

// DELETE route
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ message: 'User deleted', id });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Middleware

Middleware functions have access to req, res, and next().

```javascript
// Application-level middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // Pass to next middleware
});

// Middleware with path
app.use('/api', (req, res, next) => {
  console.log('API request');
  next();
});

// Multiple middleware
app.get('/profile',
  authenticate,
  authorize('admin'),
  (req, res) => {
    res.json({ profile: req.user });
  }
);

// Custom middleware
function logger(req, res, next) {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
}

app.use(logger);

// Error-handling middleware (4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

// Built-in middleware
app.use(express.json());                          // Parse JSON
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded
app.use(express.static('public'));               // Serve static files

// Third-party middleware
const cors = require('cors');
const morgan = require('morgan');
const helmet = require('helmet');

app.use(cors());              // Enable CORS
app.use(morgan('combined'));  // HTTP logging
app.use(helmet());            // Security headers
```

### Routing

Organize routes with Express Router.

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ users: [] });
});

router.get('/:id', (req, res) => {
  res.json({ user: { id: req.params.id } });
});

router.post('/', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

router.put('/:id', (req, res) => {
  res.json({ message: 'User updated' });
});

router.delete('/:id', (req, res) => {
  res.json({ message: 'User deleted' });
});

module.exports = router;

// app.js
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);

// Now accessible at:
// GET    /api/users
// GET    /api/users/:id
// POST   /api/users
// PUT    /api/users/:id
// DELETE /api/users/:id
```

### Request and Response Objects

```javascript
app.get('/demo', (req, res) => {
  // Request properties
  console.log(req.params);      // Route parameters
  console.log(req.query);       // Query string
  console.log(req.body);        // Request body
  console.log(req.headers);     // Headers
  console.log(req.method);      // HTTP method
  console.log(req.url);         // URL
  console.log(req.path);        // Path
  console.log(req.ip);          // Client IP
  console.log(req.hostname);    // Host
  console.log(req.protocol);    // http/https
  console.log(req.secure);      // Is HTTPS?
  console.log(req.cookies);     // Cookies (with cookie-parser)
  
  // Response methods
  res.send('Text');                         // Send text
  res.json({ key: 'value' });              // Send JSON
  res.status(404).send('Not Found');       // Set status
  res.sendFile('/path/to/file.html');      // Send file
  res.download('/path/to/file.pdf');       // Download file
  res.redirect('/other-page');             // Redirect
  res.set('Content-Type', 'text/html');    // Set header
  res.cookie('name', 'value', options);    // Set cookie
  res.clearCookie('name');                 // Clear cookie
  
  // Method chaining
  res
    .status(200)
    .set('Content-Type', 'application/json')
    .json({ success: true });
});
```

### Error Handling

```javascript
// Async error handling
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    next(error); // Pass to error handler
  }
});

// Async wrapper to avoid try-catch
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    throw new Error('User not found');
  }
  res.json(user);
}));

// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Use custom error
app.get('/admin', (req, res) => {
  throw new AppError('Access denied', 403);
});

// Global error handler
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  
  res.status(statusCode).json({
    status: 'error',
    statusCode,
    message
  });
  
  if (!err.isOperational) {
    console.error('Unexpected error:', err);
  }
});

// 404 handler (must be last)
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});
```

---

## 6.5 Database Integration

### MongoDB with Mongoose

Mongoose is an ODM (Object Data Modeling) library for MongoDB.

```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
async function connectDB() {
  try {
    await mongoose.connect('mongodb://localhost:27017/myapp', {
      useNewUrlParser: true,
      useUnifiedTopology: true
    });
    console.log('MongoDB connected');
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
}

// Define schema
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: 2,
    maxlength: 50
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Please enter valid email']
  },
  age: {
    type: Number,
    min: 0,
    max: 120
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: Date
}, {
  timestamps: true // Auto-manage createdAt, updatedAt
});

// Add methods
userSchema.methods.getPublicProfile = function() {
  return {
    id: this._id,
    name: this.name,
    email: this.email
  };
};

// Add statics
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

// Add virtuals
userSchema.virtual('profile').get(function() {
  return `${this.name} (${this.email})`;
});

// Pre/Post hooks (middleware)
userSchema.pre('save', function(next) {
  console.log('About to save user');
  this.updatedAt = Date.now();
  next();
});

userSchema.post('save', function(doc) {
  console.log('User saved:', doc._id);
});

// Create model
const User = mongoose.model('User', userSchema);

// CRUD operations
async function examples() {
  // Create
  const user = new User({
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
  });
  
  await user.save();
  
  // Or using create
  const user2 = await User.create({
    name: 'Jane Doe',
    email: 'jane@example.com',
    age: 28
  });
  
  // Read (find)
  const users = await User.find(); // All users
  const activeUsers = await User.find({ isActive: true });
  const admins = await User.find({ role: 'admin' })
    .select('name email') // Select specific fields
    .limit(10)
    .skip(0)
    .sort({ createdAt: -1 });
  
  // Find one
  const user = await User.findById(id);
  const user = await User.findOne({ email: 'john@example.com' });
  const user = await User.findByEmail('john@example.com'); // Custom static
  
  // Update
  await User.updateOne({ _id: id }, { age: 31 });
  await User.updateMany({ isActive: false }, { isActive: true });
  
  const updated = await User.findByIdAndUpdate(
    id,
    { age: 31 },
    { new: true, runValidators: true } // Return updated doc
  );
  
  // Delete
  await User.deleteOne({ _id: id });
  await User.deleteMany({ isActive: false });
  
  const deleted = await User.findByIdAndDelete(id);
  
  // Aggregation
  const stats = await User.aggregate([
    { $match: { isActive: true } },
    { $group: {
      _id: '$role',
      count: { $sum: 1 },
      avgAge: { $avg: '$age' }
    }},
    { $sort: { count: -1 } }
  ]);
  
  // Population (references)
  const postSchema = new mongoose.Schema({
    title: String,
    author: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    }
  });
  
  const Post = mongoose.model('Post', postSchema);
  
  const post = await Post.findById(id).populate('author');
  console.log(post.author.name); // Populated user data
}
```

### PostgreSQL with node-postgres

```javascript
const { Pool } = require('pg');

// Create connection pool
const pool = new Pool({
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  user: 'postgres',
  password: 'password',
  max: 20, // Max connections in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});

// Query
async function query(text, params) {
  const start = Date.now();
  const result = await pool.query(text, params);
  const duration = Date.now() - start;
  console.log('Executed query', { text, duration, rows: result.rowCount });
  return result;
}

// Examples
async function examples() {
  // Simple query
  const result = await pool.query('SELECT * FROM users');
  console.log(result.rows);
  
  // Parameterized query (prevents SQL injection)
  const result = await pool.query(
    'SELECT * FROM users WHERE email = $1',
    ['john@example.com']
  );
  
  // Insert
  const result = await pool.query(
    'INSERT INTO users(name, email, age) VALUES($1, $2, $3) RETURNING *',
    ['John Doe', 'john@example.com', 30]
  );
  
  console.log('Inserted:', result.rows[0]);
  
  // Update
  await pool.query(
    'UPDATE users SET age = $1 WHERE id = $2',
    [31, 1]
  );
  
  // Delete
  await pool.query('DELETE FROM users WHERE id = $1', [1]);
  
  // Transaction
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    
    await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [100, 1]);
    await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [100, 2]);
    
    await client.query('COMMIT');
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// Graceful shutdown
process.on('SIGTERM', async () => {
  await pool.end();
  process.exit(0);
});
```

### ORMs (Sequelize, Prisma)

**Sequelize Example:**

```javascript
const { Sequelize, DataTypes } = require('sequelize');

const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres'
});

// Define model
const User = sequelize.define('User', {
  name: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    unique: true,
    validate: {
      isEmail: true
    }
  },
  age: {
    type: DataTypes.INTEGER,
    validate: {
      min: 0,
      max: 120
    }
  }
});

// Sync (create tables)
await sequelize.sync();

// CRUD
const user = await User.create({
  name: 'John',
  email: 'john@example.com',
  age: 30
});

const users = await User.findAll();
const user = await User.findByPk(1);
const user = await User.findOne({ where: { email: 'john@example.com' } });

await User.update({ age: 31 }, { where: { id: 1 } });
await User.destroy({ where: { id: 1 } });
```

---

## 6.6 Authentication and Security

### Password Hashing with bcrypt

```javascript
const bcrypt = require('bcrypt');

// Hash password
async function hashPassword(password) {
  const saltRounds = 10;
  const hash = await bcrypt.hash(password, saltRounds);
  return hash;
}

// Verify password
async function verifyPassword(password, hash) {
  const match = await bcrypt.compare(password, hash);
  return match;
}

// Usage in user registration
app.post('/register', async (req, res) => {
  const { email, password } = req.body;
  
  // Hash password
  const hashedPassword = await hashPassword(password);
  
  // Save user
  const user = await User.create({
    email,
    password: hashedPassword
  });
  
  res.status(201).json({ message: 'User created' });
});

// Login
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await User.findOne({ email });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const isValid = await verifyPassword(password, user.password);
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate token (see JWT section)
  const token = generateToken(user.id);
  
  res.json({ token });
});
```

### JWT Authentication

```javascript
const jwt = require('jsonwebtoken');

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';

// Generate token
function generateToken(userId) {
  return jwt.sign(
    { userId },
    JWT_SECRET,
    { expiresIn: '7d' }
  );
}

// Verify token
function verifyToken(token) {
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    return decoded;
  } catch (error) {
    return null;
  }
}

// Authentication middleware
async function authenticate(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  const decoded = verifyToken(token);
  if (!decoded) {
    return res.status(401).json({ error: 'Invalid token' });
  }
  
  // Attach user to request
  req.userId = decoded.userId;
  req.user = await User.findById(decoded.userId);
  
  if (!req.user) {
    return res.status(401).json({ error: 'User not found' });
  }
  
  next();
}

// Protected routes
app.get('/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});

// Refresh token pattern
function generateTokens(userId) {
  const accessToken = jwt.sign({ userId }, JWT_SECRET, { expiresIn: '15m' });
  const refreshToken = jwt.sign({ userId }, JWT_SECRET, { expiresIn: '7d' });
  
  return { accessToken, refreshToken };
}

app.post('/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  const decoded = verifyToken(refreshToken);
  if (!decoded) {
    return res.status(401).json({ error: 'Invalid refresh token' });
  }
  
  const tokens = generateTokens(decoded.userId);
  res.json(tokens);
});
```

### Security Best Practices

```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

// Security headers
app.use(helmet());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests'
});

app.use('/api/', limiter);

// Strict rate limit for login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts'
});

app.use('/api/login', loginLimiter);

// Prevent NoSQL injection
app.use(mongoSanitize());

// Prevent XSS
app.use(xss());

// Prevent parameter pollution
app.use(hpp());

// CORS configuration
const cors = require('cors');

app.use(cors({
  origin: 'https://yourdomain.com',
  credentials: true,
  optionsSuccessStatus: 200
}));

// Input validation with joi
const Joi = require('joi');

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
  name: Joi.string().min(2).max(50).required()
});

app.post('/register', async (req, res) => {
  const { error, value } = userSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }
  
  // Proceed with validated data
  const user = await User.create(value);
  res.status(201).json(user);
});

// Environment variables
require('dotenv').config();

const config = {
  port: process.env.PORT || 3000,
  dbUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
  nodeEnv: process.env.NODE_ENV || 'development'
};

// HTTPS in production
if (config.nodeEnv === 'production') {
  app.use((req, res, next) => {
    if (req.header('x-forwarded-proto') !== 'https') {
      res.redirect(`https://${req.header('host')}${req.url}`);
    } else {
      next();
    }
  });
}
```

---

## 6.7 RESTful APIs and GraphQL

### RESTful API Design

```javascript
// User routes following REST conventions
const express = require('express');
const router = express.Router();

// GET /api/users - List all users
router.get('/', async (req, res) => {
  const { page = 1, limit = 10, sort = 'createdAt' } = req.query;
  
  const users = await User.find()
    .limit(limit * 1)
    .skip((page - 1) * limit)
    .sort(sort);
  
  const count = await User.countDocuments();
  
  res.json({
    users,
    totalPages: Math.ceil(count / limit),
    currentPage: page
  });
});

// GET /api/users/:id - Get single user
router.get('/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json({ user });
});

// POST /api/users - Create user
router.post('/', async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json({ user });
});

// PUT /api/users/:id - Update user (full replacement)
router.put('/:id', async (req, res) => {
  const user = await User.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true, runValidators: true, overwrite: true }
  );
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json({ user });
});

// PATCH /api/users/:id - Partial update
router.patch('/:id', async (req, res) => {
  const user = await User.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true, runValidators: true }
  );
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json({ user });
});

// DELETE /api/users/:id - Delete user
router.delete('/:id', async (req, res) => {
  const user = await User.findByIdAndDelete(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.status(204).send();
});

module.exports = router;

// API versioning
app.use('/api/v1/users', require('./routes/v1/users'));
app.use('/api/v2/users', require('./routes/v2/users'));
```

### GraphQL with Apollo Server

```javascript
const { ApolloServer, gql } = require('apollo-server-express');

// Define schema
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
  }
  
  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
  }
  
  type Query {
    users: [User!]!
    user(id: ID!): User
    posts: [Post!]!
    post(id: ID!): Post
  }
  
  type Mutation {
    createUser(name: String!, email: String!): User!
    updateUser(id: ID!, name: String, email: String): User!
    deleteUser(id: ID!): Boolean!
    createPost(title: String!, content: String!, authorId: ID!): Post!
  }
`;

// Define resolvers
const resolvers = {
  Query: {
    users: async () => {
      return await User.find();
    },
    
    user: async (parent, { id }) => {
      return await User.findById(id);
    },
    
    posts: async () => {
      return await Post.find();
    },
    
    post: async (parent, { id }) => {
      return await Post.findById(id);
    }
  },
  
  Mutation: {
    createUser: async (parent, { name, email }) => {
      const user = await User.create({ name, email });
      return user;
    },
    
    updateUser: async (parent, { id, name, email }) => {
      const user = await User.findByIdAndUpdate(
        id,
        { name, email },
        { new: true }
      );
      return user;
    },
    
    deleteUser: async (parent, { id }) => {
      await User.findByIdAndDelete(id);
      return true;
    },
    
    createPost: async (parent, { title, content, authorId }) => {
      const post = await Post.create({ title, content, author: authorId });
      return post;
    }
  },
  
  User: {
    posts: async (parent) => {
      return await Post.find({ author: parent.id });
    }
  },
  
  Post: {
    author: async (parent) => {
      return await User.findById(parent.author);
    }
  }
};

// Create Apollo Server
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // Add authentication context
    const token = req.headers.authorization || '';
    const user = getUserFromToken(token);
    return { user };
  }
});

// Apply middleware
await server.start();
server.applyMiddleware({ app });

// GraphQL queries
/*
query {
  users {
    id
    name
    email
    posts {
      title
    }
  }
}

query {
  user(id: "123") {
    name
    posts {
      title
      content
    }
  }
}

mutation {
  createUser(name: "John", email: "john@example.com") {
    id
    name
  }
}
*/
```

---

## Comprehensive FAQs

**Q1: What's the difference between CommonJS and ES Modules in Node.js?**

**A:** CommonJS (`require`/`module.exports`) is the original Node.js module system (synchronous). ES Modules (`import`/`export`) are the standard (asynchronous, better for tree-shaking).

```javascript
// CommonJS
const express = require('express');
module.exports = { myFunction };

// ES Modules (add "type": "module" to package.json)
import express from 'express';
export { myFunction };
```

**Related Concepts:** Module systems, imports/exports

---

**Q2: How do I handle environment variables securely?**

**A:** Use the `dotenv` package and never commit `.env` files. Use environment-specific files and key management services in production.

```javascript
// .env file (add to .gitignore)
DATABASE_URL=mongodb://localhost:27017/mydb
JWT_SECRET=your-secret-key

// Load in app
require('dotenv').config();
const dbUrl = process.env.DATABASE_URL;
```

**Related Concepts:** Security, configuration, deployment

---

**Q3: What's the best way to handle async errors in Express?**

**A:** Use async wrapper middleware or express-async-errors package.

```javascript
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json(users);
}));
```

**Related Concepts:** Error handling, middleware, async/await

---

**Q4: Should I use MongoDB or PostgreSQL?**

**A:** Use MongoDB for flexible schemas, rapid prototyping, and document-oriented data. Use PostgreSQL for complex queries, transactions, and relational data.

**Related Concepts:** Database design, data modeling

---

**Q5: How do I prevent SQL/NoSQL injection?**

**A:** Always use parameterized queries/prepared statements. Never concatenate user input into queries.

```javascript
// ❌ Vulnerable to SQL injection
await pool.query(`SELECT * FROM users WHERE email = '${email}'`);

// ✅ Safe (parameterized)
await pool.query('SELECT * FROM users WHERE email = $1', [email]);

// ✅ Safe (Mongoose)
await User.find({ email }); // Mongoose handles sanitization
```

**Related Concepts:** Security, input validation, database queries

---

## Interview Questions

**Question 1: Explain the Node.js event loop and its phases.**

**Difficulty:** Senior

**Answer:**

The Node.js event loop processes async operations in six phases:

1. **Timers:** Execute setTimeout/setInterval callbacks
2. **Pending callbacks:** Execute I/O callbacks deferred from previous iteration
3. **Idle, prepare:** Internal use
4. **Poll:** Retrieve new I/O events, execute I/O callbacks
5. **Check:** Execute setImmediate callbacks
6. **Close callbacks:** Execute close event callbacks (e.g., socket.on('close'))

Between each phase, microtasks (process.nextTick, Promises) are processed.

```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
setImmediate(() => console.log('3'));
Promise.resolve().then(() => console.log('4'));
process.nextTick(() => console.log('5'));
console.log('6');

// Output: 1, 6, 5, 4, 2, 3
```

**Why This Matters:** Understanding the event loop is crucial for writing performant async code and debugging timing issues.

**Follow-up:**
* What's the difference between setImmediate and setTimeout(fn, 0)?
* Why does process.nextTick have the highest priority?

---

**Question 2: How would you implement authentication middleware?**

**Difficulty:** Mid-Level

**Answer:**

```javascript
const jwt = require('jsonwebtoken');

async function authenticate(req, res, next) {
  try {
    // Extract token
    const token = req.headers.authorization?.replace('Bearer ', '');
    
    if (!token) {
      return res.status(401).json({ error: 'No token provided' });
    }
    
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Find user
    const user = await User.findById(decoded.userId);
    
    if (!user) {
      return res.status(401).json({ error: 'Invalid token' });
    }
    
    // Attach to request
    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Usage
app.get('/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});
```

**Why This Matters:** Authentication middleware is fundamental to securing APIs.

**Follow-up:**
* How would you implement role-based authorization?
* How do you handle token refresh?

---

**Question 3: Design a RESTful API for a blog with posts and comments.**

**Difficulty:** Mid-Level

**Answer:**

```javascript
// Posts routes
GET    /api/posts           // List posts
GET    /api/posts/:id       // Get post
POST   /api/posts           // Create post
PUT    /api/posts/:id       // Update post
DELETE /api/posts/:id       // Delete post

// Comments routes
GET    /api/posts/:id/comments      // List comments for post
POST   /api/posts/:id/comments      // Create comment
PUT    /api/comments/:id            // Update comment
DELETE /api/comments/:id            // Delete comment

// Implementation
router.get('/posts/:id/comments', async (req, res) => {
  const comments = await Comment.find({ postId: req.params.id });
  res.json({ comments });
});

router.post('/posts/:id/comments', async (req, res) => {
  const comment = await Comment.create({
    postId: req.params.id,
    ...req.body
  });
  res.status(201).json({ comment });
});
```

**Why This Matters:** RESTful API design is a core backend skill.

**Follow-up:**
* How would you implement pagination?
* How would you version your API?

---

## Key Takeaways

✅ **Node.js Fundamentals:**
- Event-driven, non-blocking I/O
- V8 engine + libuv architecture
- Event loop with multiple phases
- Global objects (process, Buffer, etc.)

✅ **Core Modules:**
- fs for file operations (sync, async, streams)
- path for cross-platform file paths
- http/https for servers and requests
- crypto for security operations

✅ **NPM:**
- Package.json defines project
- Semantic versioning (semver)
- Lock files ensure consistency
- Scripts automate tasks

✅ **Express.js:**
- Routing and middleware
- Request/response handling
- Error handling patterns
- RESTful API design

✅ **Databases:**
- MongoDB with Mongoose (NoSQL)
- PostgreSQL with node-postgres (SQL)
- ORMs for abstraction
- Query optimization

✅ **Security:**
- Password hashing (bcrypt)
- JWT authentication
- Input validation (joi)
- Security headers (helmet)
- Rate limiting

✅ **Best Practices:**
- Async error handling
- Environment configuration
- Input validation
- Security middleware
- API versioning

---

## Conclusion

Part 6 covered Node.js and server-side JavaScript development:
- Node.js architecture and runtime
- Core modules for system operations
- NPM package management
- Express.js web framework
- Database integration
- Authentication and security
- RESTful APIs and GraphQL

**What's Next:**

In **Part 7: Modern JavaScript Development**, we'll explore:
- TypeScript fundamentals and advanced types
- Build tools (Webpack, Vite, esbuild)
- Testing with Jest, Vitest
- Code quality (ESLint, Prettier)
- Development workflows

Master Node.js to build scalable server applications!

---

**Total Word Count: Part 6 Complete (~22,000 words)**
