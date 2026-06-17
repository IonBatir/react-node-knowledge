# JavaScript Internals: Functions & Execution Context

This file details JavaScript functions, lexical scopes, execution contexts, closures, `this` binding, currying, and advanced recursion.

---

## 1. Types of Functions

JavaScript functions are "first-class citizens" (they can be stored in variables, passed as arguments, and returned from other functions).

1.  **Function Declaration**: Hoisted completely.
    ```javascript
    function declared() { return "I am fully hoisted"; }
    ```
2.  **Function Expression**: Hoisted as `undefined` (if declared with `var`) or subject to TDZ (if `let`/`const`).
    ```javascript
    const expressed = function() { return "I am an expression"; };
    ```
3.  **Arrow Function**: Concise syntax, lexically bound `this`.
    ```javascript
    const arrow = () => "I am an arrow function";
    ```
4.  **Generator Function**: Returns a generator object; can pause/resume execution using `yield`.
    ```javascript
    function* generator() { yield 1; }
    ```
5.  **Async Function**: Returns a `Promise` implicitly; allows the use of `await`.
    ```javascript
    async function asyncFunc() { return "Resolved"; }
    ```

---

## 2. Immediately-Invoked Function Expressions (IIFE)

An IIFE is a function that is executed immediately after it is created.

```javascript
(function() {
  const privateVar = "I am isolated";
  console.log("IIFE executed!");
})();
```

### Why we need it:
Before ES6 block scoping (`let`/`const`), any variable declared in a script was function-scoped or global. Developers wrapped code in an IIFE to create a **private scope**, avoiding global pollution and naming conflicts.
*   *Modern usage*: Mostly obsolete due to ES Modules and block-scoping, but still useful in immediate initializations (like top-level async functions in older Node runtimes).

---

## 3. Arrow Functions vs Plain Functions

Arrow functions are not just syntactic sugar; they have several fundamental design differences:

| Feature | Plain Functions | Arrow Functions |
| :--- | :--- | :--- |
| **`this` Binding** | **Dynamic**: Evaluated when called (caller-dependent) | **Lexical**: Inherited from the enclosing scope |
| **`arguments` Object**| Yes | No (inherits from outer function scope) |
| **Constructor (`new`)**| Yes (can construct objects) | No (throws `TypeError`) |
| **`prototype` Property**| Yes | No (`undefined`) |
| **Duplicate Parameters**| Allowed (in non-strict mode) | Never allowed |

```javascript
const obj = {
  name: "Alice",
  regularFn() { console.log(this.name); },
  arrowFn: () => console.log(this.name) // inherits this (global window / undefined)
};

obj.regularFn(); // "Alice"
obj.arrowFn();   // undefined (or window.name in browser)
```

---

## 4. Function Constructors

When called with the `new` operator, a plain function acts as a constructor.

1.  A new empty object `{}` is created.
2.  The object's internal prototype (`[[Prototype]]`) is set to the constructor function's `prototype`.
3.  The function is executed with `this` bound to the newly created object.
4.  If the function doesn't return an object explicitly, it returns `this` implicitly.

```javascript
function Cat(name) {
  this.name = name;
  // returns this implicitly
}
const myCat = new Cat("Whiskers");
console.log(myCat.name); // "Whiskers"
```

---

## 5. "Array-Like" `arguments` Object

In non-arrow functions, the `arguments` local variable is an **Array-like object** containing the values of the arguments passed to the function.

*   **Characteristics**: It has a `.length` property and index access (e.g. `arguments[0]`), but lacks array methods (`map`, `filter`, `forEach`).
*   **Conversion**: Convert it to a real array using `Array.from(arguments)` or `[...arguments]`.
*   **Modern Alternative**: **Rest Parameters (`...args`)** are actual arrays and should be preferred in all modern code.

```javascript
function oldArgs() {
  console.log(arguments.map); // undefined (not a real array)
  const argsArray = Array.from(arguments);
  console.log(argsArray.map(x => x * 2)); // works!
}

// Modern Rest Parameters
const modernArgs = (...args) => {
  console.log(args.map(x => x * 2)); // works!
};
```

---

## 6. Recursion: Definition & Examples

Recursion is a programming pattern where a function calls itself to solve a smaller instance of the same problem.

### Key Requirements:
1.  **Base Case**: The termination condition that stops recursion. Without it, the execution leads to a Stack Overflow.
2.  **Recursive Step**: The block where the function invokes itself, working closer to the base case.

### Example: Factorial
```javascript
function factorial(n) {
  if (n <= 1) return 1; // Base case
  return n * factorial(n - 1); // Recursive step
}
```

---

## 7. Recursive Traversal

Recursion is particularly suited for walking nested structures (trees, graphs, JSON payloads) where the nesting depth is dynamic.

### Example: Deep sum of numbers in a nested object
```javascript
const companySalary = {
  sales: [{ name: "John", salary: 1000 }, { name: "Alice", salary: 1600 }],
  development: {
    sites: [{ name: "Peter", salary: 2000 }, { name: "Alex", salary: 1800 }],
    internals: [{ name: "Jack", salary: 1300 }]
  }
};

function sumSalaries(department) {
  if (Array.isArray(department)) {
    // Base Case: Sum salaries of workers in this array
    return department.reduce((prev, current) => prev + current.salary, 0);
  } else {
    // Recursive Step: Iterate through sub-departments
    let sum = 0;
    for (const subdep of Object.values(department)) {
      sum += sumSalaries(subdep);
    }
    return sum;
  }
}
console.log(sumSalaries(companySalary)); // 7700
```

---

## 8. The `new Function` feature

The `new Function` constructor allows compiling a function dynamically from string statements.

`let func = new Function([arg1, arg2, ...argN], functionBodyString);`

```javascript
const sum = new Function("a", "b", "return a + b");
console.log(sum(2, 3)); // 5
```

### Unique Scope Behavior:
Unlike ordinary functions, functions created using `new Function` **do not have access to their enclosing lexical scope**. Instead, their outer reference points directly to the **global scope**.

```javascript
let value = "global";

function test() {
  let value = "local";
  const f = new Function("console.log(value)");
  f();
}

test(); // Prints "global" (does not see local variable "value")
```

---

## 9. Currying

Currying is a functional programming technique that transforms a function of multiple arguments, like `f(a, b, c)`, into a sequence of nested unary functions, like `f(a)(b)(c)`.

```javascript
// Regular sum
const sum = (a, b) => a + b;

// Curried sum
const curriedSum = a => b => a + b;
console.log(curriedSum(2)(3)); // 5

// Partial application usage
const addTen = curriedSum(10);
console.log(addTen(5)); // 15
```

---

## 10. Tag Functions (Tagged Templates)

A tagged template is a function call that takes a template literal.
The first argument is an array of strings, and the subsequent arguments are the interpolated expressions.

```javascript
function tag(strings, ...values) {
  console.log(strings); // [ "Hello ", "!" ]
  console.log(values);  // [ "World" ]
  return "custom output";
}

const entity = "World";
const result = tag`Hello ${entity}!`;
```

---

## 11. "this" Definition

In JavaScript, `this` represents the execution context of a function call. It is not static; it is **dynamic** and evaluated at call time based on **how** the function is invoked.

---

## 12. Four Rules of `this` Binding

### 1. Default Binding (Global / Undefined)
If called as a standalone function in non-strict mode, `this` points to the global object. In strict mode, `this` is `undefined`.
```javascript
function showThis() { return this; }
showThis(); // window (non-strict) / undefined (strict)
```

### 2. Implicit Binding (Object Context)
When a function is called as a method of an object, `this` points to the object preceding the dot.
```javascript
const obj = {
  val: 42,
  getVal() { return this.val; }
};
obj.getVal(); // 42 (this points to obj)
```

### 3. Explicit Binding (`call`, `apply`, `bind`)
*   `call(thisArg, arg1, arg2, ...)`: Executes function immediately, passing arguments individually.
*   `apply(thisArg, [arg1, arg2, ...])`: Executes function immediately, passing arguments as an array.
*   `bind(thisArg, arg1, ...)`: Returns a **new function** with `this` permanently bound.
```javascript
function introduce(greeting) {
  return `${greeting}, I am ${this.name}`;
}
const user = { name: "Bob" };

console.log(introduce.call(user, "Hello")); // "Hello, I am Bob"
const bound = introduce.bind(user);
console.log(bound("Hi")); // "Hi, I am Bob"
```

### 4. New Binding (Constructor Call)
When called with `new`, `this` points to the newly instantiated object.

---

## 13. Global Context (`window`, `globalThis`)

*   **`window`**: The global object in standard browser environments.
*   **`global`**: The global object in Node.js.
*   **`globalThis`**: A unified, environment-agnostic keyword (introduced in ES2020) that accesses the global object regardless of execution environment (browser, web worker, or Node.js).

---

## 14. Lexical Environment Definition

Every executing function, block of code, or script has an associated internal object called the **Lexical Environment**.

It consists of two parts:
1.  **Environment Record**: An object storing all local variables, parameters, and declared functions as properties.
2.  **Outer Lexical Environment Reference**: A link pointing to the parent Lexical Environment (the scope where the code was physically written).

```
┌──────────────────────────────────────────────┐
│ Lexical Environment (Local)                  │
├──────────────────────────────────────────────┤
│ Environment Record: { x: 10, y: 20 }         │
│ Outer Reference: ────────────────────────────┼───► [ Lexical Environment (Parent) ]
└──────────────────────────────────────────────┘
```

---

## 15. Scope and Scope Chain

*   **Scope**: The visibility context of identifiers (variables, functions).
*   **Scope Chain**: When the JS engine looks for a variable, it starts at the innermost local Lexical Environment's record. If not found, it follows the **Outer Lexical Environment** link to inspect the parent environment, repeating this process until it reaches the global scope. If it is still not found, it throws a `ReferenceError`.

---

## 16. Variable Shadowing

Variable shadowing occurs when a variable declared within a inner scope has the exact same name as a variable in an outer scope. The inner variable "shadows" (blocks access to) the outer variable within that block.

```javascript
let color = "red"; // Outer variable

if (true) {
  let color = "blue"; // Shadows the outer variable
  console.log(color); // "blue"
}

console.log(color); // "red"
```

---

## 17. Closures

A **closure** is the combination of a function bundled together with references to its surrounding state (its Lexical Environment). In simpler terms, a closure allows a function to **remember and access its outer variables** even when it is executed outside its original scope.

```javascript
function createCounter() {
  let count = 0; // Preserved variable
  
  return function() {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

### Scope Name Conflicts
If an inner closure has variables with the same name as outer variables (shadowing), it will default to accessing the closest variable in the scope chain.
```javascript
const name = "Global";
function outer() {
  const name = "Outer";
  return function inner() {
    const name = "Inner";
    console.log(name); // "Inner" (Name conflict resolved by closest scope)
  };
}
```

---

## 18. Lexical Environment Garbage Collection

Normally, when a function finishes executing, its local Lexical Environment is removed from memory (garbage collected) because its variables are no longer reachable.

However, **if a closure exists and remains reachable**, its outer reference keeps the parent Lexical Environment alive in memory:

```javascript
function makeFunc() {
  let heavyData = new Array(1000000);
  return function() {
    console.log("Alive");
  };
}

let myFunc = makeFunc(); // The heavyData array is kept in memory because myFunc's outer scope points to it.
myFunc = null; // Now the closure is unreachable; memory is freed.
```

### Modern Engine Optimizations
V8 and other modern engines analyze closure code at compilation. If a parent variable is not explicitly referenced inside the inner functions, it is **swept from the Lexical Environment** even if the closure remains alive, avoiding unnecessary memory overhead.

---

## Practical Checkpoint

What does the following print? Think through the closure and lexical environment rules:

```javascript
function makePick(x) {
  return function(y) {
    return x + y;
  };
}

const addFive = makePick(5);
const addTen = makePick(10);

console.log(addFive(2));
console.log(addTen(2));
```
