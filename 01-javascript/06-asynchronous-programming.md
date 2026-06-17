# JavaScript Internals: Asynchronous Programming

This file covers JavaScript's asynchronous execution architecture: blocking execution, the Event Loop, task scheduling, Promises, custom polyfills, generators, and async iterators.

---

## 1. Blocking Code

JavaScript is a **single-threaded** language, meaning it has a single call stack and can only execute one command at a time.
*   **Blocking Code**: A synchronous CPU-bound task (e.g., a massive `for` loop, complex cryptography, or synchronous file reads) that occupies the call stack. While this task is running, the main thread cannot process user clicks, render frames, or handle network responses. The UI freezes.

```javascript
// Example of Blocking Code
console.log("Start");
const endTime = Date.now() + 3000;
while (Date.now() < endTime) {
  // Loop blocks the call stack for 3 seconds
}
console.log("End"); // Runs only after 3 seconds; page was completely frozen during this time
```

---

## 2. `setTimeout` and `setInterval`

Since JS is single-threaded, we need a way to schedule execution without blocking the call stack. Timer functions are provided by the host environment (Browser Web APIs or Node.js runtime).

*   **`setTimeout(callback, delay, ...args)`**: Schedules a single execution of the callback after `delay` milliseconds.
*   **`setInterval(callback, delay, ...args)`**: Repeatedly executes the callback every `delay` milliseconds.
*   **Why we need them**: To perform non-blocking actions, such as polling a server, animating UI components, or deferring operations to keep the main thread responsive.

### Clearing Timers:
Both functions return a unique ID that can be passed to cancel the scheduled executions.
```javascript
const timerId = setTimeout(() => console.log("Hi"), 5000);
clearTimeout(timerId); // Prevents the function from executing

const intervalId = setInterval(() => console.log("Tick"), 1000);
clearInterval(intervalId); // Stops the loop
```

---

## 3. The Event Loop Concept

The **Event Loop** is the orchestrator that enables JavaScript's non-blocking, concurrent execution model.

```
┌───────────────────────────────────────────────┐
│              Call Stack                       │  (Runs synchronous code)
└──────┬────────────────────────────────────────┘
       │ (Async APIs delegated to host)
       ▼
┌───────────────────────────────────────────────┐
│        Host APIs (Web APIs / Node C++)        │  (Timers, Fetch, File I/O)
└──────┬────────────────────────────────────────┘
       │ (When complete, callbacks pushed to queues)
       ▼
 ┌────────────────────────┐      ┌────────────────────────┐
 │   Microtask Queue      │      │   Macrotask Queue      │
 ├────────────────────────┤      ├────────────────────────┤
 │ - Promises             │      │ - setTimeout / interval│
 │ - queueMicrotask       │      │ - Event listener click │
 └───────────┬────────────┘      └───────────┬────────────┘
             │                               │
             └───────────────┬───────────────┘
                             ▼
                 [   Event Loop   ]
                 Checks if Call Stack is empty. 
                 1. Pulls & runs all Microtasks.
                 2. Pulls & runs ONE Macrotask.
                 3. Repaints UI (if needed).
```

---

## 4. Zero Delay (`setTimeout(() => {}, 0)`)

Passing `0` (or leaving it blank) to `setTimeout` does not run the callback immediately.
1.  The callback is registered in the Web API.
2.  The callback is pushed immediately into the **Macrotask Queue**.
3.  The callback will **only execute** once the current synchronous call stack is completely empty, and any waiting microtasks have run.

```javascript
console.log("A");
setTimeout(() => console.log("B"), 0);
console.log("C");
// Outputs: A -> C -> B
```

---

## 5. Macrotasks vs Microtasks

The Event Loop divides callbacks into two distinct queues:

| Feature | Microtasks | Macrotasks (Tasks) |
| :--- | :--- | :--- |
| **Sources** | `Promise.then/catch/finally`, `queueMicrotask()`, `MutationObserver` | `setTimeout`, `setInterval`, `setImmediate` (Node), I/O events, UI clicks |
| **Priority** | Higher (runs immediately after the current script) | Lower (runs one task per loop cycle) |
| **Queue Execution**| The entire queue is drained until empty (even if new microtasks are added recursively) | Executed one at a time. The loop checks microtasks and rendering between each macrotask |

### Execution Cycle Walkthrough:
```javascript
console.log("Start");

setTimeout(() => console.log("Timeout (Macrotask)"), 0);

Promise.resolve().then(() => console.log("Promise (Microtask)"));

console.log("End");

// Output:
// 1. "Start" (Sync)
// 2. "End" (Sync)
// 3. "Promise (Microtask)" (Microtask Queue drained)
// 4. "Timeout (Macrotask)" (Macrotask runs)
```

---

## 6. Fake Multithreading and Concurrency

JavaScript is single-threaded, meaning there is no *parallel* execution of JavaScript code on separate threads (excluding Web Workers). 
*   **Concurrency**: The illusion of multi-tasking is achieved by delegating blocking asynchronous operations (like network requests, file reading, timers) to the **host environment** (which operates on C++ or system-level threads).
*   Once the host environment finishes the operation, it places the callback into the event loop queue, where it is executed on the single JS thread.

---

## 7. Promises vs Callbacks

*   **Callbacks**: Simple functions passed to another function to be executed when an operation finishes.
    *   **Cons**: Leads to **Callback Hell** (deeply nested pyramids of doom) and **Inversion of Control** (trusting a third-party library to call your callback correctly).
*   **Promises**: An object representing the eventual completion (or failure) of an asynchronous operation.
    *   **Pros**: Flattens nested structures via chaining (`.then()`), supports structured error propagation (`.catch()`), and maintains control flow.

---

## 8. States of Promises

A Promise operates in one of three states:

```
                  ┌───────────────┐
                  │    Pending    │ (Initial state)
                  └──────┬────────┘
                         │
         ┌───────────────┴───────────────┐
         ▼                               ▼
┌─────────────────┐             ┌─────────────────┐
│    Fulfilled    │             │    Rejected     │ (Final/Settled states)
│ (with a value)  │             │ (with a reason) │
└─────────────────┘             └─────────────────┘
```
*   **Immutability**: Once a promise transitions from `Pending` to either `Fulfilled` or `Rejected`, it becomes **settled** and its state/value can never change.

---

## 9. Promise Chaining & Error Propagation

Chaining works because every call to `.then()`, `.catch()`, or `.finally()` returns a **new Promise**.

### Error Bubble Down (Falling through):
If a promise chain encounters an error, it skips all subsequent `.then()` blocks until it finds a `.catch()` block.

```javascript
Promise.resolve("Step 1")
  .then(val => {
    console.log(val);
    throw new Error("Failed at Step 2");
  })
  .then(val => console.log("Step 3 won't run"))
  .catch(err => console.error(`Caught: ${err.message}`)); // Catches error from Step 2
```

---

## 10. Custom Promise Implementation

Here is a simplified polyfill demonstrating how a Promise handles status transitions, callbacks, and asynchronous resolution:

```javascript
class MyPromise {
  constructor(executor) {
    this.state = "pending";
    this.value = undefined;
    this.successCallbacks = [];
    this.failureCallbacks = [];

    const resolve = (value) => {
      if (this.state !== "pending") return;
      this.state = "fulfilled";
      this.value = value;
      // Execute callbacks asynchronously (mimicking microtask behavior)
      queueMicrotask(() => {
        this.successCallbacks.forEach(cb => cb(this.value));
      });
    };

    const reject = (reason) => {
      if (this.state !== "pending") return;
      this.state = "rejected";
      this.value = reason;
      queueMicrotask(() => {
        this.failureCallbacks.forEach(cb => cb(this.value));
      });
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : val => val;
    onRejected = typeof onRejected === "function" ? onRejected : err => { throw err; };

    return new MyPromise((resolve, reject) => {
      const handleCallback = () => {
        try {
          if (this.state === "fulfilled") {
            const result = onFulfilled(this.value);
            result instanceof MyPromise ? result.then(resolve, reject) : resolve(result);
          } else {
            const result = onRejected(this.value);
            result instanceof MyPromise ? result.then(resolve, reject) : resolve(result);
          }
        } catch (err) {
          reject(err);
        }
      };

      if (this.state === "pending") {
        this.successCallbacks.push(handleCallback);
        this.failureCallbacks.push(handleCallback);
      } else {
        queueMicrotask(handleCallback);
      }
    });
  }
}
```

---

## 11. `Promise.all` Polyfill

`Promise.all` takes an array of promises, waits for all of them to resolve, and returns an array of results. If **any** promise rejects, the entire operation rejects immediately.

```javascript
Promise.myAll = function(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completedCount = 0;
    const array = Array.from(promises);

    if (array.length === 0) {
      return resolve([]);
    }

    array.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completedCount++;
          if (completedCount === array.length) {
            resolve(results);
          }
        })
        .catch(reject); // Rejects immediately on any failure
    });
  });
};
```

---

## 12. `Promise.race` & `Promise.allSettled`

### `Promise.race(iterable)`
Settles as soon as the **first** promise in the iterable settles (either resolves or rejects).
```javascript
Promise.race([
  new Promise(res => setTimeout(() => res("Slow"), 100)),
  new Promise((res, rej) => setTimeout(() => rej("Fast error"), 10))
])
.catch(err => console.log(err)); // "Fast error"
```

### `Promise.allSettled(iterable)`
Waits for all promises to settle and returns an array of objects describing the outcome of each promise. It **never rejects**.
```javascript
Promise.allSettled([
  Promise.resolve(1),
  Promise.reject("error")
]).then(results => {
  console.log(results);
  // [
  //   { status: "fulfilled", value: 1 },
  //   { status: "rejected", reason: "error" }
  // ]
});
```

---

## 13. Iterators and Generators

### Iterators
An object is an **iterator** if it implements a `next()` method returning `{ value, done }`.
```javascript
const makeIterator = (arr) => {
  let index = 0;
  return {
    next() {
      return index < arr.length 
        ? { value: arr[index++], done: false } 
        : { done: true };
    }
  };
};
```

### Generators
Generators are functions that can be entered and exited multiple times, retaining their context across yields. They are declared with `function*` and pause execution using `yield`.

```javascript
function* numberGen() {
  yield 1;
  yield 2;
  return 3;
}

const gen = numberGen();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: true }
```

### Generator Composition (`yield*`)
You can delegate generator execution to another generator using `yield*`.
```javascript
function* generateSequence() {
  yield* [1, 2]; // delegates to the array's iterator
  yield 3;
}
```

### Bi-directional Communication (Passing values to `next()`)
The `yield` keyword is a two-way street. Calling `gen.next(value)` passes `value` back into the generator as the result of the current `yield` expression.

```javascript
function* interactiveGen() {
  const result = yield "Give me a value";
  console.log(`Received: ${result}`);
}

const iterator = interactiveGen();
console.log(iterator.next().value); // "Give me a value"
iterator.next("Hello!");           // Console prints: "Received: Hello!"
```

---

## 14. Async Iterators & Async Generators

Standard iterators are synchronous. If our data source is asynchronous (e.g., streaming chunks from a network), we use **Async Iterators** and **Async Generators**.

### Async Iterators
Implement a method at the `Symbol.asyncIterator` key, returning a promise resolving to `{ value, done }`.
We consume them using the **`for await...of`** loop.

```javascript
const asyncRange = {
  from: 1,
  to: 3,
  [Symbol.asyncIterator]() {
    let current = this.from;
    return {
      async next() {
        await new Promise(resolve => setTimeout(resolve, 500)); // async delay
        if (current <= 3) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
};

(async () => {
  for await (const num of asyncRange) {
    console.log(num); // Prints 1, 2, 3 with a 500ms pause between each
  }
})();
```

### Async Generators
Declared as `async function*`. They allow using `yield` and `await` together.
```javascript
async function* fetchSequence(url) {
  let page = 1;
  while (page <= 3) {
    const data = await fetch(`${url}?page=${page}`).then(res => res.json());
    yield data;
    page++;
  }
}
```

---

## Practical Checkpoint

Predict the output order of this script:

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout 1");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise 1");
  setTimeout(() => {
    console.log("Timeout 2");
  }, 0);
}).then(() => {
  console.log("Promise 2");
});

console.log("End");
```
