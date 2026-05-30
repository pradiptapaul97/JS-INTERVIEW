# 🚀 JavaScript Interview Guide

A comprehensive, curated guide covering core JavaScript concepts, advanced mechanics, asynchronous programming, and common interview topics with clear explanations and code examples.

---

## 📋 Table of Contents

- [Function Types](#-function-types)
  - [Function Statement (Declaration)](#function-statement-declaration)
  - [Function Expression](#function-expression)
  - [Anonymous Function](#anonymous-function)
  - [Named Function](#named-function)
  - [First Class Function](#first-class-function)
  - [Higher Order Function](#higher-order-function)
  - [Arrow Function](#arrow-function)
- [Block and Scope](#-block-and-scope)
  - [Shadowing](#shadowing)
- [Currying](#-currying)
- [Closure](#-closure)
- [Prototypes and Inheritance](#-prototypes-and-inheritance)
- [Hoisting](#-hoisting)
- [Temporal Dead Zone (TDZ)](#-temporal-dead-zone-tdz)
- [Chaining Techniques](#-chaining-techniques)
  - [Method Chaining](#method-chaining)
  - [Prototype Chaining](#prototype-chaining)
  - [Callback Chaining (Callback Hell)](#callback-chaining-callback-hell)
  - [Promise Chaining](#promise-chaining)
- [Callback Functions in JavaScript](#-callback-functions-in-javascript)
  - [Difference Between process.nextTick() and setImmediate()](#difference-between-processnexttick-and-setimmediate)
  - [Issues with Callbacks](#issues-with-callbacks)
- [Map vs Filter vs Reduce](#-map-vs-filter-vs-reduce)
- [Event Loop in JavaScript](#-event-loop-in-javascript)
  - [Why The Event Loop Exists](#why-the-event-loop-exists)
  - [Queue Priority & Execution Order](#queue-priority--execution-order)
- [Promises](#-promises)
  - [Promise.all](#1-promiseall)
  - [Promise.allSettled](#2-promiseallsettled)
  - [Promise.race](#3-promiserace)
  - [Race Conditions](#4-race-conditions)
- [Async / Await in JavaScript](#-async--await-in-javascript)
  - [Understanding How Async/Await Works](#understanding-how-asyncawait-works)
  - [Difference between Async/Await and Promise Chaining](#difference-between-asyncawait-and-promise-chaining)
  - [Test Cases: Sequential Execution with Await](#test-cases-sequential-execution-with-await)

---

## 🛠️ Function Types

### Function Statement (Declaration)
A normal named function declared using the `function` keyword.

> [!NOTE]
> **Features:**
> - Fully hoisted (can be called before its declaration in the code).

```javascript
function a() {
  console.log("Function Statement");
}
a();
```

### Function Expression
A function stored inside a variable.

> [!NOTE]
> **Features:**
> - The variable is hoisted, but the function value itself is not.
> - Often used for passing callbacks.

```javascript
let b = function() {
  console.log("Function Expression");
};
b();
```

### Anonymous Function
A function without a name. Typically used in expressions or as arguments.

```javascript
const greet = function () {
  console.log("Anonymous Function");
};
```

### Named Function
A function expression where the function has its own name. The name is accessible only *inside* the function's own scope (useful for recursion or stack traces).

```javascript
const greet = function sayHello() {
  console.log("Hello");
  // 'sayHello' is only defined inside this scope
};
greet();
```

### First Class Function
A programming language is said to have **First-Class Functions** when functions in that language are treated like any other variable. They can be:
1. Passed as arguments to other functions.
2. Returned by other functions.
3. Assigned as values to variables.

```javascript
function sayHello() {
  return "Hello, ";
}

function greet(fn, name) {
  console.log(fn() + name);
}

// Passing 'sayHello' as an argument (First-Class behavior)
greet(sayHello, "Pradipta"); // Outputs: Hello, Pradipta
```

### Higher Order Function
A **Higher Order Function (HOF)** is a function that either accepts other functions as arguments (callbacks) or returns a function as its result. This is made possible because functions in JavaScript are First-Class Functions.

#### Example (Taking a Function as an Argument)
Using the Array `.map()` method:
```javascript
const numbers = [1, 2, 3];

// .map() is a HOF; the function passed inside is the callback.
const doubled = numbers.map((num) => num * 2);

console.log(doubled); // Outputs: [2, 4, 6]
```

### Arrow Function
Introduced in ES6, arrow functions are a concise alternative to traditional function expressions. They are anonymous and best suited for non-method functions.

> [!IMPORTANT]
> **Key Features:**
> - Concise syntax for writing functions.
> - **No `this` binding:** They do not bind their own `this` keyword; they inherit it from the parent scope (lexical `this`).
> - Cannot be used as constructors (calling them with `new` throws an error).

```javascript
// Arrow Function equivalent
const addArrow = (a, b) => a + b;

console.log(addArrow(5, 5)); // Outputs: 10
```

---

## 📦 Block and Scope

A block is a section of code enclosed in curly braces `{ }`.

```javascript
{
  var a = 10;
  let b = 20;
  const c = 30;
  console.log(a); // 10
  console.log(b); // 20
  console.log(c); // 30
}
console.log(a); // 10 (var is functionally/globally scoped, not block scoped)
console.log(b); // Uncaught ReferenceError: b is not defined
```

### Why are blocks used?
Blocks help to:
- Group multiple statements together.
- Create local scopes for variables declared with `let` and `const`.
- Control execution flow in loops, conditionals, and functions.

### Shadowing
Shadowing occurs when a variable declared inside a block or function scope has the exact same name as a variable in an outer scope. The inner variable "shadows" (hides) the outer variable.

```javascript
var a = 100;
let b = 200;
{
  var a = 10; // Shadows and overwrites the outer var 'a'
  let b = 20; // Shadows outer 'b' within this block only
  console.log(a); // 10
  console.log(b); // 20
}
console.log(a); // 10 (var 'a' was modified globally)
console.log(b); // 200 (let 'b' was shadowed block-wise, outer 'b' remains unaffected)
```

---

## ⛓️ Currying

Currying is a technique where a function with multiple arguments is transformed into a sequence of nested functions, each taking a single argument at a time.

```javascript
function add(a) {
  return function (b) {
    return a + b;
  };
}
console.log(add(2)(3)); // Outputs: 5
```

---

## 🔒 Closure

A **closure** is the combination of a function bundled together with references to its surrounding state (its **lexical environment**). It allows an inner function to remember and access variables from its outer scope even after the outer function has finished executing.

```javascript
function createCounter(init) {
  let count = init;
  return function () {
    count++;
    return count;
  };
}
const counter = createCounter(5);
console.log(counter()); // 6
console.log(counter()); // 7
```

---

## 🧬 Prototypes and Inheritance

- A **prototype** is an object from which other objects inherit properties and methods.
- JavaScript uses prototypal inheritance under the hood.
- Every JavaScript object has a hidden link (internal `[[Prototype]]` property, accessible via `__proto__` or `Object.getPrototypeOf()`) to another object called its prototype.

> [!TIP]
> **Do primitives (numbers, strings, booleans) have prototypes?**
>
> While primitives are not objects, when you try to access a property or method on them (like `.length` on a string or `.toFixed()` on a number), JavaScript temporarily wraps them in their corresponding wrapper objects (`String`, `Number`, `Boolean`). These wrapper objects are part of the prototype chain, allowing primitives to access methods. This process is called **autoboxing**.

---

## 🪂 Hoisting

Hoisting is JavaScript's default behavior of moving variable and function declarations to the top of their enclosing scope before code execution.
- **Function Declarations:** Fully hoisted (both definition and assignment).
- **`var` Declarations:** Hoisted but initialized to `undefined`.
- **`let` and `const` Declarations:** Hoisted but NOT initialized. They remain in the Temporal Dead Zone (TDZ).

---

## ⏳ Temporal Dead Zone (TDZ)

The **Temporal Dead Zone (TDZ)** is the time period between when a variable is hoisted (at the start of the execution context) and when it is actually initialized with a value in the code.

> [!WARNING]
> Accessing a variable in its TDZ throws a **ReferenceError**.
> TDZ applies to `let` and `const`.

---

## 🔗 Chaining Techniques

### Method Chaining
Method chaining is a pattern where multiple methods are called one after another on the same object in a single statement. It works when each method returns `this` (the object itself).

```javascript
const calculator = {
  value: 0,
  add(num) {
    this.value += num;
    return this; // Enables chaining
  },
  multiply(num) {
    this.value *= num;
    return this; // Enables chaining
  }
};
console.log(calculator.add(2).multiply(3).value); // Outputs: 6
```

### Prototype Chaining
Prototype chaining is the mechanism of inheritance where an object can access properties and methods belonging to other objects up the prototype chain.

```javascript
const person = {
  greet() { console.log("Hello"); }
};
const user = Object.create(person);
user.greet(); // Outputs: "Hello" (accessed via prototype chain)
```

### Callback Chaining (Callback Hell)
Executing asynchronous operations sequentially by nesting callbacks inside other callbacks. This can quickly lead to unreadable, deeply nested code (often called **Callback Hell** or the **Pyramid of Doom**).

```javascript
setTimeout(() => {
  console.log("First");
  setTimeout(() => {
    console.log("Second");
  }, 1000);
}, 1000);
```

### Promise Chaining
A cleaner technique where multiple asynchronous operations are handled sequentially using `.then()` methods. Each `.then()` returns a new Promise, preventing the nesting typical of callback hell.

```javascript
fetch("https://api.example.com")
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.log(err));
```

---

## 📞 Callback Functions in JavaScript

A **callback** is a function passed as an argument to another function, intended to be executed later (either synchronously or asynchronously). Common use cases include:
- Asynchronous operations
- Event handling
- Timers (`setTimeout`, `setInterval`)
- Functional array methods (`.map()`, `.filter()`, etc.)

### Difference Between `process.nextTick()` and `setImmediate()` (Node.js)
These are Node.js-specific asynchronous scheduling methods with differing execution times:
1. **`process.nextTick()`**: Schedules a callback to run immediately after the current operation completes, *before* the event loop continues (draining the microtask queue).
2. **`setImmediate()`**: Schedules a callback to execute in the next iteration of the event loop (the check phase).

### Issues with Callbacks

#### 1. Callback Hell (Pyramid of Doom)
Multiple nested asynchronous functions make the code deeply indented, hard to read, maintain, and debug.
```javascript
asyncOp1(() => {
  asyncOp2(() => {
    asyncOp3(() => {
      asyncOp4(() => {
        console.log("Deeply nested!");
      });
    });
  });
});
```

#### 2. Inversion of Control (IOC)
By passing a callback to a third-party library or external function, you hand over control of when, how, and if your code is executed.
> [!CAUTION]
> **Associated Risks of IOC:**
> - The callback might be called too many times.
> - The callback might be called too few times (or never).
> - The callback might be called with the wrong arguments.

---

## 🗺️ Map vs Filter vs Reduce

| Method | Purpose | Use Case | Example |
| :--- | :--- | :--- | :--- |
| **`map()`** | **Transformation** | Creates a new array of the same length by transforming every element. | Extracting usernames from user objects. |
| **`filter()`** | **Selection** | Creates a new array with elements that pass a conditional test. | Getting active users from a list of all users. |
| **`reduce()`** | **Aggregation** | Condenses all elements in an array down to a single output value. | Calculating the total sum or compiling a tally. |

---

## 🔄 Event Loop in JavaScript

The **Event Loop** is a core engine mechanism that enables non-blocking, asynchronous execution in JavaScript, despite it being single-threaded. It continuously checks the Call Stack and queues to coordinate execution.

```
┌──────────────────────────────────────────┐
│                Call Stack                │
└────────────────────┬─────────────────────┘
                     │ (If stack is empty)
                     ▼
┌──────────────────────────────────────────┐
│             Microtask Queue              │  ◄── Promises, process.nextTick()
└────────────────────┬─────────────────────┘
                     │ (When microtasks are empty)
                     ▼
┌──────────────────────────────────────────┐
│              Callback Queue              │  ◄── setTimeout, setImmediate
└──────────────────────────────────────────┘
```

### Why The Event Loop Exists
JavaScript is single-threaded, meaning it can only execute one task at a time. The Event Loop helps offload heavy operations (like timers, network requests, or database queries) to the browser APIs or the OS and brings back their callbacks when done, keeping the interface responsive.

### Queue Priority & Execution Order
When the Call Stack is empty, the Event Loop processes queues in the following order:
1. **Call Stack**: Currently executing code.
2. **`process.nextTick()`** (Node.js specific): Run immediately after the current operation.
3. **Microtask Queue**: Promises (`.then`/`.catch`/`await`), `queueMicrotask()`.
4. **Callback Queue (MacroTask)**: `setTimeout`, `setInterval`, domestic DOM events.
5. **`setImmediate()`** (Node.js specific): Executed in the check phase of the loop.

---

## 🤝 Promises

A **Promise** is an object representing the eventual completion or failure of an asynchronous operation. It acts as a placeholder for a value that is not yet available.

### Core Methods:

#### 1. `Promise.all`
`Promise.all()` is a helper method that takes an iterable of promises (usually an array) and returns a single Promise. This returned promise resolves when **all** of the input promises have resolved, or rejects immediately if **any** of the input promises reject.

##### How It Works in Parallel (Concurrent Execution)
In JavaScript, asynchronous tasks (such as network requests or timers) are offloaded to the browser APIs or Node.js background threads. When you invoke `Promise.all([p1, p2, p3])`, all these promises are initiated **simultaneously** at the exact same moment. They run concurrently in the background and do not block one another.

##### 🧪 Scenario: The Slowest Promise Determines the Time
Suppose you have three promises:
1. **Promise 1 (`p1`)**: Resolves in **3 seconds**
2. **Promise 2 (`p2`)**: Resolves in **1 second**
3. **Promise 3 (`p3`)**: Resolves in **2 seconds**

If you run them using `Promise.all([p1, p2, p3])`, the **total execution time will be 3 seconds**.

###### Code Example
```javascript
const p1 = new Promise((resolve) => setTimeout(() => resolve("P1 (3s)"), 3000));
const p2 = new Promise((resolve) => setTimeout(() => resolve("P2 (1s)"), 1000));
const p3 = new Promise((resolve) => setTimeout(() => resolve("P3 (2s)"), 2000));

const start = Date.now();

Promise.all([p1, p2, p3])
  .then((results) => {
    console.log("Results:", results); // ["P1 (3s)", "P2 (1s)", "P3 (2s)"]
    console.log(`Total time: ${(Date.now() - start) / 1000}s`); 
    // Total time: ~3s (e.g. 3.002s)
  })
  .catch((error) => {
    console.error("One of the promises failed:", error);
  });
```

###### ⏳ Why does it take 3 seconds instead of 6 seconds (3 + 1 + 2)?
Since these promises run in parallel, their timers tick down concurrently in the background:
- **`t = 0s`**: All three promises are kicked off.
- **`t = 1s`**: `p2` finishes and resolves. The other two are still running in the background.
- **`t = 2s`**: `p3` finishes and resolves. `p1` is still running in the background.
- **`t = 3s`**: `p1` (the slowest promise) finally resolves.
- **Completion**: Since all promises have now resolved, `Promise.all` completes and returns the aggregated results array.

```
Timeline of Parallel Execution:
t = 0s
 ├─► p2 (1s) ───────────► Resolved (at t = 1s)
 ├─► p3 (2s) ───────────────────────────► Resolved (at t = 2s)
 └─► p1 (3s) ───────────────────────────────────────────────────► Resolved (at t = 3s)
                                                                ▲
                                                       Promise.all Resolves
```

> [!TIP]
> **Summary Rule:** The total time taken by `Promise.all` for successfully resolving concurrent operations is always equal to the duration of the **slowest (maximum duration) promise** in the array.

##### ⚠️ Error Handling: What if one or multiple promises reject (error)?
`Promise.all` employs a **"fail-fast" (short-circuiting)** behavior when it comes to errors:

1. **Immediate Rejection:** As soon as **any** single promise rejects, the entire `Promise.all` rejects **immediately** with that promise's error. It does not wait for the remaining promises to finish or settle.
2. **What happens to other running promises?** The other promises in the array that are still pending will continue executing in the background (JavaScript does not cancel them automatically), but their results or subsequent rejections will be completely ignored.
3. **If multiple promises reject:** Only the error of the **first promise that rejects (chronologically)** will be returned and caught. Any subsequent failures from other promises are discarded.

###### 🧪 Example: Failure Scenario
Let's see what happens when `p2` rejects after 1 second:
```javascript
const p1 = new Promise((resolve) => setTimeout(() => resolve("P1 (3s)"), 3000));
// p2 rejects after 1 second
const p2 = new Promise((_, reject) => setTimeout(() => reject(new Error("P2 Failed! (1s)")), 1000));
const p3 = new Promise((resolve) => setTimeout(() => resolve("P3 (2s)"), 2000));

const start = Date.now();

Promise.all([p1, p2, p3])
  .then((results) => {
    console.log("This will NOT run because p2 rejected");
  })
  .catch((error) => {
    console.error("Caught error:", error.message); // Caught error: P2 Failed! (1s)
    console.log(`Rejected after: ${(Date.now() - start) / 1000}s`); 
    // Rejected after: ~1s
  });
```

###### ⏳ Why did it reject in 1 second instead of 3 seconds?
Because `Promise.all` is "fail-fast":
- **`t = 0s`**: All three promises are kicked off.
- **`t = 1s`**: `p2` rejects. `Promise.all` immediately terminates and throws the rejection to the `.catch` block. It doesn't wait for `p3` (at 2s) or `p1` (at 3s) to resolve.

> [!WARNING]
> Since the other promises keep running in the background, be careful of side effects! If those background promises write to a database or alter state, those actions will still complete even though `Promise.all` threw an error at the 1-second mark.


#### 2. `Promise.allSettled`

##### 🛑 The Negatives of `Promise.all` (Why we need `Promise.allSettled`)
While `Promise.all` is powerful, its "fail-fast" behavior introduces critical downsides in real-world applications:
1. **Total Loss of Data:** If you are fetching data from 5 different APIs and 4 succeed but 1 fails, `Promise.all` immediately rejects. You lose access to the 4 successful API responses, forcing you to treat the entire operation as a complete failure.
2. **Ignorant Execution:** You have no way of knowing which tasks succeeded and which failed because the `.catch()` block only receives the error from the *first* promise that rejected.
3. **No Partial Successes:** There is no straightforward way to collect the "good" data and handle/log the "bad" data individually.

##### 💡 How `Promise.allSettled` Solves This
`Promise.allSettled()` was introduced in ES2020 to overcome these limitations:

- **Resilient Execution:** It **never rejects**. The returned promise always resolves, regardless of how many individual promises fail.
- **Complete Insight:** It waits for **every** promise to settle (either resolve or reject).
- **Standardized Output Structure:** It returns an array of objects, with one object per promise, describing its final outcome:
  - For successful promises: `{ status: "fulfilled", value: resultValue }`
  - For rejected promises: `{ status: "rejected", reason: errorReason }`

##### 🧪 Example: Collecting Successes & Handling Failures
Let's see how `Promise.allSettled` behaves in a scenario where one promise fails:

```javascript
const p1 = new Promise((resolve) => setTimeout(() => resolve("Data from API 1"), 3000));
const p2 = new Promise((_, reject) => setTimeout(() => reject(new Error("API 2 Down!")), 1000));
const p3 = new Promise((resolve) => setTimeout(() => resolve("Data from API 3"), 2000));

const start = Date.now();

Promise.allSettled([p1, p2, p3])
  .then((results) => {
    console.log(`Completed in: ${(Date.now() - start) / 1000}s`); 
    // Completed in: ~3s (Waits for all of them!)

    console.log("Full Results:", results);
    /* Output:
    [
      { status: "fulfilled", value: "Data from API 1" },
      { status: "rejected", reason: Error: API 2 Down! },
      { status: "fulfilled", value: "Data from API 3" }
    ]
    */

    // We can easily filter out the successful results:
    const successfulResponses = results
      .filter(res => res.status === "fulfilled")
      .map(res => res.value);

    console.log("Successful API Data:", successfulResponses);
    // ["Data from API 1", "Data from API 3"]

    // And log the errors individually:
    results
      .filter(res => res.status === "rejected")
      .forEach(err => console.error("Logged Error:", err.reason.message));
      // Logged Error: API 2 Down!
  });
```

> [!TIP]
> **When to use which?**
> - Use **`Promise.all`** when your operations are **interdependent** (e.g., you need all datasets or none at all; if one fails, the rest are useless).
> - Use **`Promise.allSettled`** when your operations are **independent** (e.g., loading different dashboard widgets, where one widget failing shouldn't break the entire dashboard).


#### 3. `Promise.race`
`Promise.race()` takes an iterable of promises and returns a single Promise. This returned promise settles (either resolves or rejects) **as soon as the very first promise in the input array settles**. 

> [!IMPORTANT]
> **The Golden Rule of `Promise.race`:** Whoever crosses the finish line first **wins**, regardless of whether they succeed (resolve) or fail (reject).

##### 🧪 Scenario 1: Success Wins (Fastest Promise Resolves)
If the fastest promise resolves successfully, `Promise.race` resolves with that value.

###### Code Example
```javascript
const p1 = new Promise((resolve) => setTimeout(() => resolve("Fast Success (1s)"), 1000));
const p2 = new Promise((_, reject) => setTimeout(() => reject(new Error("Slow Error (2s)")), 2000));

Promise.race([p1, p2])
  .then((value) => {
    console.log("Resolved with:", value); // "Resolved with: Fast Success (1s)"
  })
  .catch((error) => {
    console.error("This catch block will NOT execute");
  });
```

```
Timeline (Success Wins):
t = 0s
 ├─► p1 (1s) ───────────► Resolved (at t = 1s) ◄── Winner (Resolves Promise.race)
 └─► p2 (2s) ───────────────────────────► Rejected (at t = 2s)
```

##### 🧪 Scenario 2: Failure Wins (Fastest Promise Rejects)
If the fastest promise rejects/throws an error, `Promise.race` rejects immediately with that error.

###### Code Example
```javascript
const p1 = new Promise((resolve) => setTimeout(() => resolve("Slow Success (2s)"), 2000));
const p2 = new Promise((_, reject) => setTimeout(() => reject(new Error("Fast Error (1s)")), 1000));

Promise.race([p1, p2])
  .then((value) => {
    console.log("This then block will NOT execute");
  })
  .catch((error) => {
    console.error("Caught error:", error.message); // "Caught error: Fast Error (1s)"
  });
```

```
Timeline (Failure Wins):
t = 0s
 ├─► p2 (1s) ───────────► Rejected (at t = 1s) ◄── Winner (Rejects Promise.race)
 └─► p1 (2s) ───────────────────────────► Resolved (at t = 2s)
```

---

#### 4. Race Conditions
A race condition in programming is an undesirable situation that occurs when a system's substantive behavior is dependent on the sequence or timing of uncontrollable events (like network latency). In JavaScript, it often occurs when two concurrent async requests are made, and you incorrectly assume the order in which they will return.


---

## ⚡ Async / Await in JavaScript

Introduced in ES8 (ES2017), `async/await` is modern syntactic sugar built on top of Promises. It allows us to write asynchronous code that looks and behaves like synchronous code, avoiding nested `.then()` chains.

### Key Concepts:
- **`async` Keyword**: Declared before a function to ensure it always returns a Promise. If the function returns a non-promise value, JavaScript automatically wraps it in a resolved Promise.
- **`await` Keyword**: Pauses code execution inside the `async` function until the promise settles. It then returns the resolved value.

### Difference between Async/Await and Promise Chaining

| Feature | Async/Await | Promise Chaining (`.then/.catch`) |
| :--- | :--- | :--- |
| **Readability** | **High**: Flat structure, resembles synchronous code. | **Medium/Low**: Nested `.then()` chains can become verbose. |
| **Error Handling** | Uses standard **`try...catch`** blocks. | Uses **`.catch()`** attached at the end of the chain. |
| **Debugging** | **Easy**: Line-by-line debugging works natively. | **Complex**: Debugger has to step through asynchronous microtasks. |

---

## 🧪 Test Cases: Sequential Execution with Await

Sequential execution occurs when asynchronous operations are awaited one after another. The `await` keyword pauses execution of the async function until the current promise settles before moving to the next.

```javascript
async function handlePromise() {
  const start = Date.now();
  const val1 = await p1;
  console.log(val1 + " after " + (Date.now() - start)/1000 + "s");
  const val2 = await p2;
  console.log(val2 + " after " + (Date.now() - start)/1000 + "s");
}

handlePromise();
```

### Test Case 1: Slow Promise is Second (5s then 10s)
In this scenario, `p1` resolves in 5 seconds and `p2` in 10 seconds. Since they are run in sequence:
- **`p1`** resolves after 5 seconds.
- **`p2`** resolves after 10 seconds.

**Expected Output:**
```
Promise 1 resolved after 5.003 seconds
Promise 2 resolved after 10.004 seconds
```

### Test Case 2: Slow Promise is First (10s then 5s)
Here, execution blocks for 10 seconds at the first `await`. Even though the second promise would have finished in 5 seconds, it must wait for the first to complete.

```javascript
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Promise 1 resolved");
  }, 10000);
});  
const promise2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Promise 2 resolved");
  }, 5000);
});
```

**Expected Output:**
```
Promise 1 resolved after 10.001 seconds
Promise 2 resolved after 10.002 seconds
```

### Conclusion
The total execution time for sequential `await` operations is the sum of the individual promise delays due to blocking behavior. If promises are initiated concurrently before being awaited, they run in parallel; however, awaiting them sequentially enforces a blocking pipeline.
