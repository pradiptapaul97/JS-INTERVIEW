# 🚀 Node.js Advanced Interview Guide: Routing, Middleware, and Caching

A curated, comprehensive guide covering essential web server architectures for technical interviews: Routing, Middleware design, and Caching strategies, complete with deep-dive explanations, comparisons, and robust code examples.

---

## 📋 Table of Contents

- [Routing in Node.js](#routing-in-nodejs)
  - [Definition and Mechanics](#definition-and-mechanics)
  - [Types of Routing](#types-of-routing)
  - [🧪 Raw Node.js Route Handler Example](#-raw-nodejs-route-handler-example)
- [Middleware Architecture](#middleware-architecture)
  - [Definition and Mechanics](#definition-and-mechanics-1)
  - [Types of Middleware](#types-of-middleware)
  - [🧪 Custom Middleware Runner Example](#-custom-middleware-runner-example)
- [Caching Strategies and Types](#caching-strategies-and-types)
  - [Definition and Mechanics](#definition-and-mechanics-2)
  - [Types of Caching](#types-of-caching)
  - [Cache Eviction Policies](#cache-eviction-policies)
  - [🧪 In-Memory TTL Cache Example](#-in-memory-ttl-cache-example)

---

## 🌐 Routing in Node.js

### Definition and Mechanics

**Routing** is the mechanism by which a web server maps an incoming HTTP request—defined by its HTTP method (e.g., `GET`, `POST`, `PUT`, `DELETE`) and its URI path (e.g., `/users`, `/products`)—to a specific handler function. 

In a raw Node.js environment (using the native `http` module), routing is handled by parsing the request URL using modules like `url` or `URL` and executing conditional logic (e.g., `switch` statements or dictionary lookups) to dispatch the request to the correct function.

---

### Types of Routing

| Routing Type | Description | Example / Use Case |
| :--- | :--- | :--- |
| **Static Routing** | Direct, exact matches of standard URI paths. | `GET /about` or `GET /contact` |
| **Dynamic / Parameterized Routing** | Matches paths containing placeholders or variable segments, capturing those values from the URI. | `GET /users/:id` (where `:id` matches `42`) |
| **Query-string / Search Routing** | Modifies the behavior of a route handler based on key-value pairs appended to the URL. | `GET /products?category=shoes&sort=price` |
| **Regex / Wildcard Routing** | Matches routes using regular expressions or wildcards to capture groups of paths. | `GET /static/*` (to serve assets dynamically) |

---

### 🧪 Raw Node.js Route Handler Example

The following code illustrates how to build a robust, dynamic route dispatcher from scratch using only Node.js standard libraries (`http` and `url`), featuring static paths, search query extraction, and parameterized dynamic matching:

```javascript
const http = require('http');
const url = require('url');

// Simple registry for static routes
const routes = {
  'GET': {},
  'POST': {}
};

// Route Registration Helper
const router = {
  get: (path, handler) => routes['GET'][path] = handler,
  post: (path, handler) => routes['POST'][path] = handler
};

// 1. Static Route Registration
router.get('/', (req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'Welcome to the API Home!' }));
});

router.get('/about', (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('About Us Page');
});

// 2. Dynamic Route Handler Function
const handleDynamicRoutes = (req, res, parsedUrl) => {
  const path = parsedUrl.pathname;
  const method = req.method;

  // Pattern match for /users/:id (matching integers only)
  const userMatch = path.match(/^\/users\/(\d+)$/);
  if (userMatch && method === 'GET') {
    const userId = userMatch[1];
    
    // 3. Query Parameter Handling
    const queryParams = parsedUrl.query;
    const detailLevel = queryParams.detail || 'basic';

    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      userId: userId,
      role: 'User',
      detailLevel: detailLevel,
      source: 'Dynamic Router'
    }));
    return true; // Return true indicating route was handled
  }

  return false; // Not a dynamic match
};

// Create Raw Server and route incoming requests
const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const method = req.method;

  console.log(`[Request] ${method} ${path}`);

  // Dispatch Static Route
  if (routes[method] && routes[method][path]) {
    return routes[method][path](req, res);
  }

  // Dispatch Dynamic Route
  if (handleDynamicRoutes(req, res, parsedUrl)) {
    return;
  }

  // Fallback 404 Route
  res.writeHead(404, { 'Content-Type': 'text/plain' });
  res.end('Route Not Found');
});

server.listen(3000, () => {
  console.log('🚀 Raw HTTP Router running on http://localhost:3000');
});
```

---

## ⚙️ Middleware Architecture

### Definition and Mechanics

**Middleware** functions are pipeline components that execute sequentially within the application's request-response cycle. They have access to the request object (`req`), the response object (`res`), and the next middleware function in the queue, commonly named `next`.

Middleware functions can:
- Execute any code.
- Make changes to the request and response objects (e.g., adding user auth data to `req.user`).
- End the request-response cycle (preventing further handlers from running).
- Call the next middleware in the stack by invoking `next()`.

```
Incoming Request ──► [ Middleware 1 ] ──► [ Middleware 2 ] ──► [ Final Handler ] ──► Response Out
                         (Logger)           (Auth Guard)           (Controller)
```

---

### Types of Middleware

1. **Application-level Middleware:** Bound directly to the application instance (e.g., Express `app.use()`). Runs on every incoming request.
2. **Router-level Middleware:** Bound to a specific router instance (e.g., Express `router.use()`). Runs only for requests routed through that specific router.
3. **Error-handling Middleware:** Specialized middleware designed for centralized error processing. Identified by taking four arguments instead of three: `(err, req, res, next)`.
4. **Built-in Middleware:** Pre-packaged modules provided directly by frameworks (e.g., `express.json()` for JSON parsing, `express.static()` for serving files).
5. **Third-party Middleware:** Middleware installed via package managers to add capabilities (e.g., `cors()`, `helmet()` for security headers, `morgan()` for request logging).

---

### 🧪 Custom Middleware Runner Example

Understanding how frameworks compose and chain middleware using recursion and call stacks is a highly sought-after interview skill. Here is an implementation of a pure JavaScript middleware runner:

```javascript
class MiddlewareRunner {
  constructor() {
    this.stack = [];
  }

  // Register middleware function
  use(fn) {
    this.stack.push(fn);
  }

  // Run the middleware stack sequentially
  run(req, res, finalHandler) {
    let index = 0;

    // The 'next' callback function handles sequential iteration
    const next = (err) => {
      // If an error is passed down, break chain and send error response
      if (err) {
        console.error(`[Error Middleware Triggered]: ${err.message}`);
        res.writeHead(500, { 'Content-Type': 'text/plain' });
        return res.end(`Internal Server Error: ${err.message}`);
      }

      if (index < this.stack.length) {
        const middleware = this.stack[index++];
        try {
          // Invoke the middleware, passing req, res, and the next pointer recursively
          middleware(req, res, next);
        } catch (error) {
          next(error); // Capture runtime errors and send to error handler
        }
      } else {
        // Execute the final handler once the entire middleware stack has run
        finalHandler(req, res);
      }
    };

    next(); // Initiate stack execution
  }
}

// --- Usage Demonstration ---
const app = new MiddlewareRunner();

// Middleware 1: Logging
app.use((req, res, next) => {
  req.receivedAt = Date.now();
  console.log(`[Middleware 1] Logged request for: ${req.url}`);
  next(); // Move to next middleware
});

// Middleware 2: Authentication simulation
app.use((req, res, next) => {
  const token = req.headers['authorization'];
  
  if (token === 'secret-token-123') {
    req.user = { id: 101, username: 'pradipta', role: 'admin' };
    next(); // Authorized, proceed
  } else {
    // End request-response cycle early
    res.writeHead(401, { 'Content-Type': 'text/plain' });
    res.end('Unauthorized: Missing or invalid token.');
  }
});

// Final Controller / Handler
const getDashboard = (req, res) => {
  const duration = Date.now() - req.receivedAt;
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    status: 'success',
    user: req.user,
    processedInMs: duration
  }));
};

// --- Mocking Server Requests ---

// Scenario A: Valid Token (Access Granted)
console.log('--- Executing Request A (Valid Auth) ---');
const reqA = { url: '/dashboard', headers: { 'authorization': 'secret-token-123' } };
const resA = {
  writeHead: (code, headers) => console.log(`[Res Header] Status: ${code}`, headers),
  end: (body) => console.log(`[Res Body]:`, body)
};
app.run(reqA, resA, getDashboard);

// Scenario B: Invalid Token (Blocked by Middleware 2)
console.log('\n--- Executing Request B (Invalid Auth) ---');
const reqB = { url: '/dashboard', headers: { 'authorization': 'wrong-token' } };
const resB = {
  writeHead: (code, headers) => console.log(`[Res Header] Status: ${code}`, headers),
  end: (body) => console.log(`[Res Body]:`, body)
};
app.run(reqB, resB, getDashboard);
```

---

## 💾 Caching Strategies and Types

### Definition and Mechanics

**Caching** is the strategy of storing copies of computed data in a high-speed, temporary storage layer (the cache) so that subsequent requests for the same resource can be served instantly. 

Caching improves server performance by:
- Eliminating duplicate database queries.
- Saving CPU-intensive calculation cycles.
- Decreasing network latency and page-load times.

---

### Types of Caching

| Cache Type | Location | Pros | Cons | Best Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **In-Memory Cache** | Server Process Memory Heap (RAM) | Microsecond speed, no network overhead, simple setup. | Resets on restart, consumes V8 heap, not shared across multi-core processes/clusters. | Storing configuration objects or infrequently modified small lookups. |
| **Distributed Cache** | Dedicated external server (e.g., Redis, Memcached) | Survives server restarts, scales across clusters/microservices, supports terabytes of memory. | Network latency (milliseconds), requires configuring additional infrastructure. | Storing user sessions, complex database query results, or hot API responses. |
| **HTTP / Browser Cache** | Client browser or CDNs/Reverse Proxies | Zero server load, responses are cached at the edge close to user. | Eviction control is tricky, stale content risk. | Static assets (Images, CSS, JS) and general public endpoints. |

---

### Cache Eviction Policies

Since memory is finite, caches must evict older or less relevant items to make room for new records. Standard eviction policies include:

- **TTL (Time-To-Live):** Data is automatically marked as expired and removed after a specified duration (e.g., 5 minutes).
- **LRU (Least Recently Used):** Tracks access history and discards the items that haven't been accessed for the longest period of time when limit is reached.
- **LFU (Least Frequently Used):** Tracks hit counts and discards the items with the lowest access frequency first.

---

### 🧪 In-Memory TTL Cache Example

Here is a highly educational, lightweight in-memory cache class written in ES6+ JavaScript featuring dynamic key expiry (TTL) and memory-safety handling:

```javascript
class TTLMemoryCache {
  constructor() {
    this.store = new Map();
  }

  /**
   * Storing data with optional Time-To-Live expiration
   * @param {string} key - Cache Identifier
   * @param {*} value - Data to cache
   * @param {number} [ttlMs] - Lifetime in milliseconds
   */
  set(key, value, ttlMs = null) {
    // If key already exists and has an active timer, clear it to prevent duplicate closures
    if (this.store.has(key)) {
      clearTimeout(this.store.get(key).timer);
    }

    let timer = null;
    if (ttlMs) {
      timer = setTimeout(() => {
        this.delete(key);
        console.log(`[Cache Eviction] Key "${key}" expired after ${ttlMs}ms.`);
      }, ttlMs);

      // Prevent the timer from holding the Node.js process open if no other work is running
      if (typeof timer.unref === 'function') {
        timer.unref();
      }
    }

    this.store.set(key, {
      value: value,
      timer: timer
    });
  }

  // Retrieve cached value
  get(key) {
    if (!this.store.has(key)) {
      return null;
    }
    return this.store.get(key).value;
  }

  // Explicitly delete key and clear its expiration timer
  delete(key) {
    if (this.store.has(key)) {
      const entry = this.store.get(key);
      if (entry.timer) {
        clearTimeout(entry.timer);
      }
      this.store.delete(key);
      return true;
    }
    return false;
  }

  // Clear all cached items and their active timers
  clear() {
    for (const [key, entry] of this.store.entries()) {
      if (entry.timer) {
        clearTimeout(entry.timer);
      }
    }
    this.store.clear();
    console.log('[Cache Cleared] All entries and timers deleted.');
  }
}

// --- Usage Demonstration ---
const cache = new TTLMemoryCache();

// Set user profiles with different lifetimes
cache.set('session:1001', { username: 'alice' }, 1000); // 1s expiry
cache.set('session:1002', { username: 'bob' }, 3000);   // 3s expiry

console.log('Get 1001 immediately:', cache.get('session:1001')); // { username: 'alice' }

// Verify expiry after 1.5 seconds
setTimeout(() => {
  console.log('\n--- 1.5 Seconds Checked ---');
  console.log('Get 1001 (1s TTL):', cache.get('session:1001')); // null (expired)
  console.log('Get 1002 (3s TTL):', cache.get('session:1002')); // { username: 'bob' } (still fresh)
}, 1500);

// Verify final expiry after 3.5 seconds
setTimeout(() => {
  console.log('\n--- 3.5 Seconds Checked ---');
  console.log('Get 1002 (3s TTL):', cache.get('session:1002')); // null (expired)
}, 3500);
```
