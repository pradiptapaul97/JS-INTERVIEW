# 🚀 Node.js Advanced Interview Guide

A curated, comprehensive guide covering essential web server architectures and high-performance file handling for technical interviews.

---

## 📋 Table of Contents

- [How do you handle large file data efficiently in Node.js?](#how-do-you-handle-large-file-data-efficiently-in-nodejs)
- [What security measures do you implement in a Node.js application?](#what-security-measures-do-you-implement-in-a-nodejs-application)
- [How do you handle concurrent users in a Node.js application?](#how-do-you-handle-concurrent-users-in-a-nodejs-application)
- [Cluster vs. Worker Threads vs. Child Process](#cluster-vs-worker-threads-vs-child-process)
- [What is a Router in Node.js?](#what-is-a-router-in-nodejs)
- [What is Middleware in Node.js?](#what-is-middleware-in-nodejs)
- [How many ways can Class B use Class A's function?](#how-many-ways-can-class-b-use-class-as-function)
- [What will we do if the server gets 502 Bad Gateway?](#what-will-we-do-if-the-server-gets-502-bad-gateway)

---

## How do you handle large file data efficiently in Node.js?

### Answer

When dealing with large files in Node.js, I avoid using `fs.readFile()` because it loads the entire file into memory before processing it. For very large files, this can cause high memory consumption or even crash the application.

Instead, I use **Streams**, which process data in smaller chunks. Streams are memory-efficient because only a portion of the file is kept in memory at any given time.

---

### Bad Approach

```javascript
const fs = require("fs");

fs.readFile("large-file.csv", "utf8", (err, data) => {
  console.log(data);
});
```

#### Problem
```
1 GB file
    ↓
Load entire 1 GB into RAM
    ↓
Process
```

If multiple users upload large files simultaneously, memory usage can become very high.

---

### Good Approach: Streams

```javascript
const fs = require("fs");

const readStream = fs.createReadStream("large-file.csv", {
  encoding: "utf8",
});

readStream.on("data", (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

readStream.on("end", () => {
  console.log("File processing completed");
});
```

#### Flow
```
1 GB file
    ↓
64 KB chunk
    ↓
Process
    ↓
Next 64 KB chunk
```

Memory remains almost constant regardless of file size.

---

### Real-World Example: File Upload

#### Instead of:

```javascript
app.post("/upload", (req, res) => {
  // Read entire file into memory ❌
});
```

#### Use:

```javascript
const fs = require("fs");

app.post("/upload", (req, res) => {
  const writeStream = fs.createWriteStream("./uploads/file.mp4");

  req.pipe(writeStream);

  writeStream.on("finish", () => {
    res.send("Upload completed");
  });
});
```

This streams the uploaded file directly to disk.

---

### Copying Large Files Efficiently

```javascript
const fs = require("fs");

const readStream = fs.createReadStream("source.zip");
const writeStream = fs.createWriteStream("destination.zip");

readStream.pipe(writeStream);
```

#### Benefits:

- **Low memory usage**
- **Faster processing**
- **Built-in backpressure handling**

---

## What security measures do you implement in a Node.js application?

### 1. Input Validation
**Why:** To ensure incoming request parameters match strict formats before processing them, preventing buffer overflows, invalid values, or malicious code executions.
**Implementation:** Use schema validators like **Joi**, **Zod**, or **express-validator**.
```javascript
const { body, validationResult } = require('express-validator');

app.post('/user', 
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 5 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Proceed...
  }
);
```

### 2. Authentication & Authorization
**Why:** Verification of identity (Authentication) and checking permissions (Authorization) prevents unauthenticated or privilege-escalation actions.
**Implementation:** Use industry standards like **JWT (JSON Web Tokens)** or **Session Cookies** for authentication, and **RBAC (Role-Based Access Control)** for authorization. Always practice the principle of least privilege.

### 3. Password Hashing
**Why:** Storing passwords in plain text is a critical security failure. If a database is compromised, all user credentials are leaked.
**Implementation:** Use cryptographic hashing algorithms with automatic salting, such as **bcrypt** or **argon2**.
```javascript
const bcrypt = require('bcrypt');

// Hashing a password during signup
const saltRounds = 12;
const hashedPassword = await bcrypt.hash(plainTextPassword, saltRounds);

// Comparing a password during login
const isMatch = await bcrypt.compare(plainTextPassword, hashedPassword);
```

### 4. Use HTTPS
**Why:** Protects data in transit from eavesdropping and Man-in-the-Middle (MitM) attacks by encrypting communications between the client and server.
**Implementation:** Obtain SSL/TLS certificates (e.g. Let's Encrypt). Configure Node.js or reverse proxies (like Nginx) to redirect all standard HTTP traffic to HTTPS.

### 5. Security Headers with Helmet
**Why:** Browsers use certain HTTP response headers to control cross-site features and enable safety filters.
**Implementation:** Use the **helmet** middleware to automatically configure secure HTTP headers (e.g., `Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security`).
```javascript
const helmet = require('helmet');
app.use(helmet());
```

### 6. Prevent SQL Injection
**Why:** SQL injection occurs when input text is concatenated directly into SQL queries, enabling attackers to execute arbitrary SQL commands.
**Implementation:** Use parameterized queries, prepared statements, or modern Object-Relational Mappers (ORMs) like **Prisma** or **Sequelize**.
```javascript
// ❌ BAD: raw string concatenation
const query = `SELECT * FROM users WHERE email = '${req.body.email}'`;

// ✅ GOOD: Parameterized query (using pg module)
const query = 'SELECT * FROM users WHERE email = $1';
db.query(query, [req.body.email]);
```

### 7. Prevent NoSQL Injection
**Why:** In MongoDB, query operators are defined as objects (e.g., `{ $ne: "" }`). If inputs aren't sanitized, attackers can pass object parameters instead of strings to bypass checks.
**Implementation:** Sanitize query objects using middleware like **express-mongo-sanitize** or force input parameters to match strings.
```javascript
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize()); // Strips out keys starting with "$" or containing "."
```

### 8. Rate Limiting
**Why:** Restricting the rate of incoming requests from an IP address prevents Denial of Service (DoS) attacks, brute-force login attempts, and API scraping.
**Implementation:** Use **express-rate-limit** or Redis-backed rate limiters for distributed environments.
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use(limiter);
```

### 9. CORS Configuration
**Why:** A wildcard Cross-Origin Resource Sharing policy (`Access-Control-Allow-Origin: *`) allows any malicious external website to access your API endpoints.
**Implementation:** Use the **cors** library and restrict access exclusively to trusted whitelist domains.
```javascript
const cors = require('cors');

const corsOptions = {
  origin: ['https://trustedwebsite.com'],
  optionsSuccessStatus: 200
};
app.use(cors(corsOptions));
```

### 10. Secure Environment Variables
**Why:** Hardcoding credentials, API keys, or private salts directly into codebase commits risks public leakages when pushing to repository hosters.
**Implementation:** Store credentials in an external `.env` file loaded with **dotenv** or use dedicated cloud secret vaults (e.g. AWS Secrets Manager). Add `.env` to `.gitignore`.
```javascript
require('dotenv').config();
const dbPassword = process.env.DB_PASSWORD;
```

### 11. Prevent XSS (Cross-Site Scripting)
**Why:** XSS occurs when raw, unescaped user inputs are rendered directly in browsers, allowing attackers to execute malicious JavaScript scripts within users' sessions.
**Implementation:** Sanitize and escape all output strings using tools like **DOMPurify** or **xss-filters**. Configure a strict Content Security Policy (CSP).

### 12. Secure Cookies
**Why:** If session identifier cookies are not properly secured, attackers can steal them via JavaScript (XSS) or send them over unencrypted connections.
**Implementation:** Configure crucial security flags on session cookies:
```javascript
app.use(session({
  cookie: {
    httpOnly: true, // Prevents client-side scripts from reading the cookie
    secure: true,   // Ensures cookie is sent only over HTTPS
    sameSite: 'strict' // Mitigates Cross-Site Request Forgery (CSRF)
  }
}));
```

### 13. Dependency Security
**Why:** Using third-party open-source packages from npm risks importing nested sub-dependencies with known vulnerabilities or malicious code.
**Implementation:** Proactively run **`npm audit`** to scan for vulnerabilities, keep dependencies updated, and integrate automated monitoring tools like **Snyk** or **GitHub Dependabot**.

### 14. Logging & Monitoring
**Why:** Without audit logs, it is impossible to identify, trace, or recover from active breaches.
**Implementation:** Use logging libraries like **winston** or **pino** to output structured logs.
> [!CAUTION]
> **Crucial Rule:** Never write highly sensitive data (e.g. passwords, credit card numbers, authorization tokens, or personally identifiable information) into log outputs.

### 15. Protect Against DoS Attacks
**Why:** Attackers can send massive JSON bodies or keep connections open indefinitely (Slowloris) to consume RAM and sockets, freezing your service.
**Implementation:** Set size limits on incoming body parses and restrict request timeouts.
```javascript
// Restrict JSON payload size
app.use(express.json({ limit: '10kb' }));

// Restrict URL-encoded payload size
app.use(express.urlencoded({ limit: '10kb', extended: true }));
```

---

## How do you handle concurrent users in a Node.js application?

### Answer

Node.js is designed to handle a large number of concurrent users using its **event-driven, non-blocking I/O architecture**.

Instead of creating one thread per request (which consumes massive RAM), Node.js utilizes:
```
Event Loop + Non-Blocking I/O
```
This allows a single OS process to handle thousands of simultaneous connections efficiently.

---

### Example

When 1,000 users request data simultaneously:

```javascript
app.get("/users", async (req, res) => {
  const users = await User.find();
  res.json(users);
});
```

Node.js does **not** block while waiting for the database to return results:
```
Request 1 ──► DB Query (Delegated to OS / Libuv)
Request 2 ──► DB Query (Delegated to OS / Libuv)
Request 3 ──► DB Query (Delegated to OS / Libuv)
...
```
The single-threaded event loop remains completely free to receive and handle other incoming requests in the meantime.

---

### But What If Traffic Increases?

Suppose:
- **Normal Traffic:** 5,000 requests
- **Peak Traffic:** 10,000+ requests

To scale beyond the limits of a single process, I would implement the following strategies:

#### 1. Horizontal Scaling
Distribute incoming request loads across multiple independent server instances.
```
                  Load Balancer (Nginx / AWS ALB)
                                 │
      ┌──────────────────────────┼──────────────────────────┐
      ▼                          ▼                          ▼
   Node 1                     Node 2                     Node 3
```
- **Tools:** Nginx, AWS Application Load Balancer (ALB), Kubernetes.

#### 2. Node.js Cluster Mode
By default, a Node.js process runs on a single CPU core. I would utilize the built-in `cluster` module to spawn worker processes matching the server's CPU core count, allowing the application to utilize all available system power:

```javascript
const cluster = require("cluster");
const os = require("os");

if (cluster.isPrimary) {
  const cpus = os.cpus().length;

  for (let i = 0; i < cpus; i++) {
    cluster.fork();
  }
}
```
This scales the application vertically on multi-core servers.

#### 3. Redis Caching
Instead of querying the primary database for every single read request, cache frequently accessed data to minimize response latency:

##### Instead of:
```
Request ──► Database
```

##### Use:
```
Request ──► Redis Cache ──► Database (only on cache miss)
```
This significantly reduces primary database load and query bottlenecks.

#### 4. Queue Heavy Operations
Move expensive, blocking, or slow-running operations out of the main request-response cycle into background workers:
- **Examples:** Sending emails, generating PDFs, processing/resizing images.
- **Tools:** BullMQ, RabbitMQ, Apache Kafka.

#### 5. Database Optimization
Ensure the database is not the primary bottleneck under load:
- Add appropriate **indexes** to fields queried frequently.
- Optimize database queries to prevent full-table scans.
- Use **connection pooling** to reuse established TCP connections efficiently.
- Setup **Read Replicas** to offload read-heavy traffic from the master database.

#### 6. Prevent Event Loop Blocking
Avoid synchronous CPU-bound operations on the main thread:

##### ❌ Bad Approach (Blocks all concurrent users):
```javascript
// Large calculation loops block the main thread from processing incoming connections!
for (let i = 0; i < 10000000000; i++) {}
```

##### ✅ Good Approach:
Offload heavy computational work to:
- **Worker Threads** (built-in `worker_threads` module)
- **Child Processes**
- **Background Message Queues**

---

### Real-World Example

In a production e-commerce application, a concurrent scaling pipeline is structured as follows:

```
10,000 Users Hit Product API
             │
             ▼
      Cloudflare CDN (Caches static assets & edge HTML)
             │
             ▼
     Load Balancer (Spreads requests across servers)
             │
             ▼
  Multiple Node.js Instances (Running in Cluster Mode)
             │
             ▼
    Redis Cache (Handles hot product lookups)
             │
             ▼
   PostgreSQL Database (Handles master writes & cache misses)
```

This multi-layered architecture can gracefully handle massive traffic spikes with zero downtime.

---

## Cluster vs. Worker Threads vs. Child Process

When scaling Node.js applications or handling performance bottlenecks, developers must choose between three key concurrency modules. Here is a high-level comparison:

| Module | Type | Memory Space | Launch Cost / Overhead | Communication | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`child_process`** | Isolated OS Process | Completely Separate | **High** (New OS resources, independent V8 heap) | Inter-process (IPC / JSON serialization) | Running external scripts (Python, Bash) or OS CLI commands. |
| **`cluster`** | Forked copies of the same process | Completely Separate | **High** (Spawns a new OS process per CPU core) | Inter-process (IPC channels managed by Master) | Distributing incoming HTTP/network connection loads. |
| **`worker_threads`** | Light threads in the same process | **Shared** (via `SharedArrayBuffer` / typed arrays) | **Low** (Leverages process resource pooling) | Thread Messaging (`parentPort` postMessage) | Heavy JavaScript CPU calculations (image resizing, encryption, AI). |

---

### 1. Child Process (`child_process`)

#### 🛑 Which issue brought it into the picture?
Node.js runs in a single process. If we want to execute non-Node.js tasks (such as a Python script for machine learning, a Bash command, or an external system binary) or run fully isolated scripts, doing so synchronously on the main thread would freeze the entire Node.js server.

#### 💡 The problem it solves
It allows Node.js to spawn separate processes on the operating system level, executing shell scripts or command-line programs asynchronously without blocking the main event loop.

#### 🧪 Simple Example: Executing a Terminal Command
```javascript
const { exec } = require("child_process");

// Spawns a shell and runs a system command asynchronously
exec("node -v", (error, stdout, stderr) => {
  if (error) {
    console.error(`Error: ${error.message}`);
    return;
  }
  console.log(`Node Version inside Child: ${stdout.trim()}`);
});
```

---

### 2. Cluster Mode (`cluster`)

#### 🛑 Which issue brought it into the picture?
On multi-core servers (e.g., a system with 8 or 16 CPU cores), a default Node.js process only runs on a single core. This leaves 90% of the server's compute capacity completely unused and wasted.

#### 💡 The problem it solves
It scales Node.js HTTP/network servers vertically. The parent (Master) process forks worker processes (essentially matching the CPU core count). It listens on a single shared TCP port and automatically distributes incoming connection requests across all worker processes using **Round-Robin** scheduling.

#### 🧪 Simple Example: Multi-Core HTTP Server
```javascript
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

if (cluster.isPrimary) {
  console.log(`Master process ${process.pid} is running.`);
  // Fork workers for all cores
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  // Workers share the same port!
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Handled by worker process ${process.pid}\n`);
  }).listen(8000);
}
```

---

### 3. Worker Threads (`worker_threads`)

#### 🛑 Which issue brought it into the picture?
When a CPU-bound computational task is executed directly in Node.js (e.g., computing Fibonacci sequences, hashing passwords with bcrypt, or image transformations), it blocks the single-threaded event loop. While the calculation runs, no other concurrent users can access the server. 

Using `child_process` or `cluster` is highly inefficient here because spawning OS processes has high RAM overhead and passing huge datasets between processes is slow due to serializing/deserializing payloads.

#### 💡 The problem it solves
It enables lightweight, multi-threaded JavaScript execution. Worker threads run **inside the same OS process**, allowing parallel CPU computations to occur with extremely low latency. Crucially, they can **share memory** (via `SharedArrayBuffer` or `ArrayBuffer` transfers), meaning large data can be mutated in place without expensive copy operations.

#### 🧪 Simple Example: Offloading CPU Calculations
```javascript
const { Worker, isMainThread, parentPort, workerData } = require("worker_threads");

if (isMainThread) {
  // Main thread spawns a worker thread passing data
  const worker = new Worker(__filename, { workerData: 40 });
  
  worker.on("message", (result) => {
    console.log(`Result from worker: ${result}`);
  });
} else {
  // Worker Thread handles computational work in parallel
  const computeFibonacci = (n) => {
    if (n < 2) return n;
    return computeFibonacci(n - 1) + computeFibonacci(n - 2);
  };

  const result = computeFibonacci(workerData);
  parentPort.postMessage(result); // Return output to main thread
}
```

---

## What is a Router in Node.js?

### Answer

A **Router** is a design pattern and architectural component in web development that acts as a traffic controller for incoming HTTP requests. It maps an incoming request—specifically its **HTTP method** (e.g., `GET`, `POST`, `PUT`, `DELETE`) and its **URL pathname** (e.g., `/users`, `/orders/12`)—to a specific function, known as a **route handler** or **controller**.

---

### Why is it used in Node.js?

1. **Decouples Request Handling:** It separates the raw network server setup from the business logic, keeping code clean and modular.
2. **Encapsulates URL Logic:** It parses query strings, extracts dynamic parameters (e.g., `/users/:id`), and handles wildcard routes automatically.
3. **Organizes Modular Apps (Express):** Frameworks provide modular routers (like `express.Router()`) to split routes into separate, dedicated files (e.g., `userRouter.js`, `productRouter.js`), acting as mini-applications.

---

### Raw Node.js Routing vs. Express.js Routing

#### 1. The Raw Node.js Approach (Custom Dispatcher)
Without a framework, developers must manually parse the URL and method, using `if/else` or `switch` blocks.

```javascript
const http = require("http");
const url = require("url");

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const method = req.method;

  // Manual route routing
  if (method === "GET" && path === "/users") {
    res.writeHead(200, { "Content-Type": "application/json" });
    return res.end(JSON.stringify([{ name: "Pradipta" }]));
  } 
  
  if (method === "POST" && path === "/users") {
    res.writeHead(201);
    return res.end("User created successfully!");
  }

  // Fallback 404 Route
  res.writeHead(404);
  res.end("Route not found");
});

server.listen(3000);
```

#### 2. The Modular Express.js Approach (Scalable Routing)
In large applications, Express simplifies this by offering a built-in `Router` class to group and modularize endpoints.

```javascript
// routes/userRoutes.js
const express = require("express");
const router = express.Router(); // Create a modular, isolated router instance

// Define isolated user routes
router.get("/", (req, res) => {
  res.json([{ name: "Pradipta" }]);
});

router.post("/", (req, res) => {
  res.status(201).send("User created successfully!");
});

module.exports = router;
```

```javascript
// server.js (Main Application file)
const express = require("express");
const app = express();
const userRouter = require("./routes/userRoutes");

// Mount the modular router onto the '/users' namespace
app.use("/users", userRouter);

app.listen(3000);
```

---

## What is Middleware in Node.js?

### Answer

**Middleware** is a function that executes **in the middle** of the request-response cycle. It acts as an interceptor or filter situated between the incoming raw request and the final route handler (controller).

```
Incoming Request ──► [ Middleware ] ──► [ Route Handler ] ──► Response Out
```

It has the signature `(req, res, next)` and can:
1. Execute any logic (e.g., logging, validation).
2. Mutate the `req` or `res` objects (e.g., adding user details to `req.user`).
3. End the request-response cycle early (e.g., throwing a `401 Unauthorized` if validation fails).
4. Hand off execution to the next function in line by calling `next()`.

---

### The 5 Core Types of Middleware (with Short Examples)

#### 1. Application-level Middleware
Bound directly to the app instance. It runs on **every incoming request** across the application.
```javascript
app.use((req, res, next) => {
  console.log(`[LOG]: ${req.method} ${req.url}`);
  next(); // Passes control to the next middleware
});
```

#### 2. Router-level Middleware
Bound to a specific router instance. It only executes for **routes associated with that router**.
```javascript
const router = express.Router();

router.use((req, res, next) => {
  if (!req.isAdmin) return res.status(403).send("Forbidden");
  next();
});
```

#### 3. Error-handling Middleware
Specialized middleware designed for centralized error catching. It **must** accept **4 arguments** `(err, req, res, next)` instead of 3.
```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send("Something went wrong!");
});
```

#### 4. Built-in Middleware
Standard parser/static handlers packaged directly inside the Express framework.
```javascript
app.use(express.json()); // Parses incoming JSON payloads on req.body
app.use(express.static("public")); // Serves static HTML/CSS files
```

#### 5. Third-party Middleware
Pre-packaged middleware installed via npm to add robust cross-cutting capabilities.
```javascript
const cors = require("cors");
const helmet = require("helmet");

app.use(cors());   // Enables CORS requests
app.use(helmet()); // Sets protective security HTTP headers
```

---

## How many ways can Class B use Class A's function?

### Answer

In JavaScript and Object-Oriented Programming (OOP), there are **seven distinct patterns** that allow Class B to access or execute a function defined in Class A. 

---

### The Main 4 Types

These represent the core structural design patterns of software engineering:

#### 1. Inheritance (`extends`)
* **Concept:** Class B extends Class A (IS-A relationship). Class B inherits all prototype methods from Class A and can invoke them directly using `this` or override them with `super`.
* **🧪 Short Example:**
```javascript
class A {
  greet() { return "Hello from A"; }
}

class B extends A {
  sayHello() {
    return this.greet(); // Accessible via inheritance
  }
}
```

#### 2. Composition (HAS-A Relationship)
* **Concept:** Class B contains and manages an instance of Class A internally. Instead of inheriting A's entire API, B delegates specific work to Class A's instance.
* **🧪 Short Example:**
```javascript
class A {
  greet() { return "Hello from A"; }
}

class B {
  constructor() {
    this.instanceA = new A(); // Composition
  }
  sayHello() {
    return this.instanceA.greet();
  }
}
```

#### 3. Dependency Injection (DI)
* **Concept:** Class B does not create Class A internally. Instead, an instance of Class A is **passed (injected)** into B's constructor or method from the outside, promoting loose coupling and making testing trivial.
* **🧪 Short Example:**
```javascript
class A {
  greet() { return "Hello from A"; }
}

class B {
  constructor(serviceA) {
    this.serviceA = serviceA; // Injected dependency
  }
  sayHello() {
    return this.serviceA.greet();
  }
}

const b = new B(new A()); // Instantiated externally
```

#### 4. Static Methods
* **Concept:** Class A defines helper methods using the `static` keyword. Class B can invoke these functions directly on the class constructor name without instantiating either class.
* **🧪 Short Example:**
```javascript
class A {
  static greet() { return "Hello from A"; }
}

class B {
  sayHello() {
    return A.greet(); // Called directly on the Class namespace
  }
}
```

---

### Other 3 JavaScript-Specific Patterns

These patterns leverage JavaScript's highly dynamic prototype chain and module ecosystem:

#### 5. Prototype Assignment
* **Concept:** Manually linking the prototypes of two constructor functions to share functions without using the modern class `extends` syntax.
```javascript
function A() {}
A.prototype.greet = function() { return "Hello from A"; };

function B() {}
B.prototype = Object.create(A.prototype); // Share prototype chain
B.prototype.constructor = B;
```

#### 6. Mixins
* **Concept:** Copying specific methods from A's prototype directly into B's prototype using object assignment. This achieves multi-class composition (simulating multiple inheritance).
```javascript
const A = {
  greet() { return "Hello from A"; }
};

class B {}
Object.assign(B.prototype, A); // Mixin A's methods into B's prototype
```

#### 7. Utility / Module Import
* **Concept:** Exporting Class A (or a singleton instance of A) from a distinct file module and requiring/importing it inside Class B's file.
```javascript
// fileA.js
class A {
  greet() { return "Hello from A"; }
}
module.exports = new A(); // Exporting Singleton

// fileB.js
const instanceA = require("./fileA");
class B {
  sayHello() {
    return instanceA.greet();
  }
}
```

---

## What will we do if the server gets 502 Bad Gateway?

### Answer

A **502 Bad Gateway** is an HTTP status code indicating that an edge server, reverse proxy, or load balancer (e.g., **Nginx**, **Apache**, **AWS ALB**, or **Cloudflare**) acted as a gateway and received an **invalid or empty response** from the upstream application server (our **Node.js process**).

```
User ──► [ Edge Proxy / Nginx ] ──❌ Connection Refused / Timeout ❌──► [ Upstream Node.js ]
```

---

### Root Causes in a Node.js Environment

1. **The Node.js Process is Dead:** The application crashed due to an uncaught exception, a syntax error, or a V8 out-of-memory (OOM) error, and Nginx is attempting to proxy requests to a port with no active listener.
2. **Event Loop Blocking (Timeout):** A synchronous CPU-intensive calculation is running on the main thread, blocking the event loop. The proxy server health check or request times out before Node.js can reply.
3. **Overloaded Request Queue:** The Node.js application is overwhelmed with concurrent requests, filling up the OS socket queue and leading to dropped connections.
4. **Port / Hostname Misconfiguration:** The proxy configuration is routing traffic to the wrong internal port (e.g., Nginx looks at `127.0.0.1:3000` but Node.js is listening on `:8080`).

---

### Step-by-Step Production Diagnostics & Fixes

When a 502 occurs, I follow this structured production investigation plan:

#### Step 1: Check Upstream Process Health
Verify if the Node.js process is active or crashed:
- **Using PM2:** Run `pm2 list` or `pm2 status` to check if the app is online or constantly rebooting.
- **Using systemctl:** Run `systemctl status node-app` to inspect the system service.
- **Action:** If the process is dead, start it immediately (`pm2 start app.js` or `npm start`).

#### Step 2: Analyze Logs (Proxy vs. Application)
- **Check Reverse Proxy Logs:** Inspect `/var/log/nginx/error.log` to identify the socket connection error:
  - `111: Connection refused` indicates the Node.js process is completely down or listening on the wrong port.
  - `110: Connection timed out` indicates the process is alive but taking too long to respond (Event Loop blocked or slow queries).
- **Check Node.js Application Logs:** Read Winston logs or `pm2 logs` to catch critical stack traces like:
  - `uncaughtException` or `unhandledRejection`.
  - `Fatal Error: JavaScript heap out of memory`.

#### Step 3: Implement Process Auto-Restart (Self-Healing)
Never run a production Node.js script raw with `node index.js`. Use process managers or containerization:
- **Process Manager (PM2):** PM2 automatically restarts the process on a crash instantly.
- **Docker / Kubernetes:** Configure container restart policies (`restart: always` or K8s liveness/readiness probes) to automatically kill and replace unhealthy instances.

#### Step 4: Setup Clustering and Load Balancing
Avoid running a single instance of a Node.js process on a server:
- Run **multiple clustered instances** matching CPU cores using the `cluster` module or `pm2 start app.js -i max`.
- Configure Nginx with an **upstream group** to distribute requests. If one instance crashes, Nginx bypasses it and routes to a healthy sibling instantly with zero downtime:
```nginx
# nginx.conf
upstream node_cluster {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    keepalive 64;
}

server {
    listen 80;
    location / {
        proxy_pass http://node_cluster;
        proxy_next_upstream error timeout http_502; # Failover on 502
    }
}
```

#### Step 5: Protect the Event Loop from Timeouts
- Offload CPU-heavy calculations to **Worker Threads** or **Background Message Queues** (e.g. BullMQ) to ensure the Event Loop remains free to quickly respond to the reverse proxy health checks.
- Adjust reverse proxy read/connect timeouts if dealing with slow streaming requests:
```nginx
proxy_connect_timeout 60s;
proxy_read_timeout 120s;
```
