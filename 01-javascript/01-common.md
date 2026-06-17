# JavaScript Basics: Common Concepts

This file contains a comprehensive walkthrough of the fundamental concepts, grammar, scoping rules, and quirks in JavaScript.

---

## 1. `let` / `const` vs `var`

Understanding the differences between declaration keywords requires looking at three axes: **Scope**, **Hoisting/Initialization**, and **Re-assignment**.

| Feature | `var` | `let` | `const` |
| :--- | :--- | :--- | :--- |
| **Scope** | Function Scope | Block Scope | Block Scope |
| **Hoisting** | Hoisted, initialized as `undefined` | Hoisted, remains uninitialized (TDZ) | Hoisted, remains uninitialized (TDZ) |
| **Re-assignment** | Allowed | Allowed | Forbidden (read-only reference) |
| **Re-declaration**| Allowed | Forbidden | Forbidden |
| **Global Object** | Adds property to `window` / `global` | Does not add to global object | Does not add to global object |

### Code Demonstration

```javascript
// Scope differences
if (true) {
  var globalLike = "I am function-scoped (leaks out of blocks)";
  let blockScoped = "I am block-scoped";
}
console.log(globalLike); // "I am function-scoped (leaks out of blocks)"
// console.log(blockScoped); // ReferenceError: blockScoped is not defined

// Re-assignment vs Re-declaration
var x = 1;
var x = 2; // Perfectly fine

let y = 1;
// let y = 2; // SyntaxError: Identifier 'y' has already been declared

const z = { name: "John" };
z.name = "Jane"; // Allowed! The reference to the object hasn't changed.
// z = { name: "Jane" }; // TypeError: Assignment to constant variable.
```

---

## 2. Variable Hoisting

**Hoisting** is the JavaScript engine's behavior of moving declarations to the top of their containing scope (execution context creation phase) before executing the code line-by-line.

During the compilation phase:
1. **Function declarations** are fully hoisted (both name and body).
2. **`var` variables** are hoisted and initialized to `undefined`.
3. **`let` and `const` variables** are hoisted but **not initialized**. They enter the Temporal Dead Zone (TDZ).

```javascript
console.log(hoistedVar); // undefined (due to hoisting and initialization)
// console.log(hoistedLet); // ReferenceError: Cannot access 'hoistedLet' before initialization

var hoistedVar = "hello";
let hoistedLet = "world";

// Function Hoisting
sayHello(); // "Hello!" (fully hoisted)

function sayHello() {
  console.log("Hello!");
}

// Function Expression Hoisting
// sayHi(); // TypeError: sayHi is not a function (hoisted as undefined)
var sayHi = function() {
  console.log("Hi!");
};
```

---

## 3. What is the Temporary Dead Zone (TDZ)?

The **Temporal Dead Zone (TDZ)** is the period between the entering of a block scope and the actual line where a `let` or `const` variable is declared. If you attempt to access the variable within this zone, JavaScript throws a `ReferenceError`.

```javascript
{ // Scope entered. TDZ for 'value' starts.
  
  // console.log(value); // ReferenceError: Cannot access 'value' before initialization
  
  let unused = 10; // Still inside TDZ for 'value'
  
  let value = 42; // TDZ for 'value' ends here!
  
  console.log(value); // 42 (Safe to access)
}
```

> [!NOTE]
> TDZ is **temporal** (time-based), not spatial (location-based). The code below works because the execution of the call happens *after* the declaration of `myVar`:
> ```javascript
> const func = () => console.log(myVar); // Not executed yet
> let myVar = 3;
> func(); // Safe: outputs 3
> ```

---

## 4. Usage of `var` and Behavior Without Declaration

### Why did we use `var` and why is it deprecated?
Before ES6, `var` was the only way to declare variables. However, its lack of block scoping and implicit global attachment made it prone to bugs (such as variable leakage in loops and variable shadowing issues).

### What happens when you declare a variable without a keyword?
If you assign a value to a variable that has not been declared using `var`, `let`, or `const`, JavaScript (in non-strict mode) will search up the scope chain. If it reaches the global scope and cannot find it, it creates a property on the **global object** (`window` in browsers, `global` in Node.js). This is called **implicit global variable leakage**.

```javascript
function leak() {
  implicitlyGlobal = "I am a global variable now!";
}
leak();
console.log(window.implicitlyGlobal); // "I am a global variable now!"
```
*   **Risks**: Pollutes the global namespace, increases memory usage, and risks naming collisions.
*   **Solution**: Always use `use strict` or modern variable declaration keywords (`let`/`const`).

---

## 5. `"use strict"` and Main Restrictions

Strict mode is a feature introduced in ES5 that opts your code into a restricted, safer variant of JavaScript. It helps catch common coding bloopers and throws exceptions where JavaScript would normally fail silently.

### How to enable it:
Add `"use strict";` at the very top of a script file or a function body. (Note: ES Modules and ES6 Classes are strict by default).

### Main Restrictions under Strict Mode:
1. **Prevents Implicit Globals**: Throws a `ReferenceError` when assigning to an undeclared variable.
   ```javascript
   "use strict";
   // myVar = 10; // ReferenceError: myVar is not defined
   ```
2. **Eliminates `this` coercion to global object**: In normal mode, if a function is called without an owner, `this` defaults to `window`/`global`. In strict mode, `this` is `undefined`.
   ```javascript
   "use strict";
   function checkThis() {
     return this;
   }
   console.log(checkThis()); // undefined
   ```
3. **Disallows Duplicate Parameter Names**:
   ```javascript
   // "use strict";
   // function add(a, a) { ... } // SyntaxError: Duplicate parameter name not allowed in this context
   ```
4. **Prevents deleting variables or functions**:
   ```javascript
   "use strict";
   var x = 5;
   // delete x; // SyntaxError: Delete of an unqualified identifier in strict mode.
   ```
5. **Secures eval and arguments**: Modifying `arguments` doesn't sync with local parameter variables, and `eval` creates variables only inside the eval's private scope.

---

## 6. Loops: `while` and `for`

JavaScript provides several looping mechanisms to iterate over code blocks.

### `while` and `do...while`
*   `while`: Evaluates condition *before* block execution.
*   `do...while`: Guarantees block execution *at least once* before evaluating condition.
```javascript
let count = 5;
while (count > 5) {
  // Never runs
}

do {
  console.log("Runs once");
} while (count > 5);
```

### Modern `for` Loops
1. **Standard `for` loop**: Best for indexing and manual steps.
   ```javascript
   for (let i = 0; i < 5; i++) { ... }
   ```
2. **`for...in` loop**: Iterates over the **enumerable property keys** of an object (including prototype chain).
   ```javascript
   const obj = { a: 1, b: 2 };
   for (const key in obj) {
     if (obj.hasOwnProperty(key)) console.log(key); // "a", "b"
   }
   ```
3. **`for...of` loop**: Iterates over **iterable values** (Arrays, Maps, Sets, Strings). Uses the iterator protocol (`Symbol.iterator`).
   ```javascript
   const arr = [10, 20, 30];
   for (const value of arr) {
     console.log(value); // 10, 20, 30
   }
   ```

---

## 7. `if` / `switch` Constructions

### `if...else`
Executes blocks based on truthy/falsy evaluation.
```javascript
if (condition) {
  // truthy path
} else if (otherCondition) {
  // fallback path
} else {
  // catch-all path
}
```

### `switch`
Executes a branch based on **strict equality (`===`)** evaluation between the switch expression and the `case` values.
```javascript
const value = "2";
switch (value) {
  case 2:
    console.log("Number 2"); // Will not match because "2" === 2 is false
    break;
  case "2":
    console.log("String 2"); // Matches!
    break;
  default:
    console.log("No match");
}
```
*   **Important**: Always add `break` to prevent fall-through unless intentional.

---

## 8. Logical Operators

JavaScript logical operators have unique behaviors because they perform **short-circuit evaluation** and return the evaluated **value** rather than a boolean.

### Operators:
*   **`&&` (AND)**: Evaluates operands from left to right. Returns the **first falsy** value, or the **last value** if all are truthy.
*   **`||` (OR)**: Evaluates operands from left to right. Returns the **first truthy** value, or the **last value** if all are falsy.
*   **`!` (NOT)**: Coerces the value to a boolean and negates it.

### Short-Circuit Examples:
```javascript
const name = "Alice";
const greeting = name && `Hello, ${name}`; // "Hello, Alice" (since name is truthy)

const defaultName = "" || "Guest"; // "Guest" (since "" is falsy)
```

---

## 9. Nullish Coalescing Operator (`??`)

The `??` operator was introduced to solve problems with the logical OR (`||`) operator when dealing with **falsy, but valid, data** (like `0`, `""`, or `false`).

*   **`||`** returns the right-hand operand if the left-hand operand is **any falsy value**.
*   **`??`** returns the right-hand operand **only** if the left-hand operand is `null` or `undefined`.

```javascript
const response = {
  speed: 0,
  title: "",
  isAdmin: false,
  description: null
};

// Logical OR (defaults triggered incorrectly)
console.log(response.speed || 10);      // 10
console.log(response.title || "Default"); // "Default"
console.log(response.isAdmin || true);   // true

// Nullish Coalescing (respects valid falsy values)
console.log(response.speed ?? 10);      // 0
console.log(response.title ?? "Default"); // ""
console.log(response.isAdmin ?? true);   // false
console.log(response.description ?? "N/A");// "N/A"
```

---

## 10. Conditional (Ternary) Operators

The conditional operator is the only JavaScript operator that takes three operands:
`condition ? expressionIfTrue : expressionIfFalse`

It is an **expression** (meaning it evaluates to a value), whereas `if...else` is a **statement**. This allows it to be used in assignments, template literals, and inline UI rendering (such as React's JSX).

```javascript
const age = 20;
const statusMessage = age >= 18 ? "Adult" : "Minor";

// React JSX context:
// <div>{isLoggedIn ? <LogoutButton /> : <LoginButton />}</div>
```
*   **Best Practice**: Avoid nesting ternary operators (e.g., `a ? b : c ? d : e`) as it severely impacts code readability.

---

## 11. Template Strings

Template literals (surrounded by backticks \` \`) allow string interpolation and multi-line strings.

### Key Features
1. **String Interpolation**: Include variables and expressions via `${expression}`.
2. **Multi-line format**: Retains white space and line breaks naturally.
   ```javascript
   const html = `
     <div>
       <h1>Hello</h1>
     </div>
   `;
   ```

### Advanced: Tagged Templates
A tagged template parses template literals with a function. The first argument is an array of string literals, and the remaining arguments correspond to the expression values.
```javascript
function highlight(strings, ...values) {
  return strings.reduce((prev, curr, i) => {
    return `${prev}<span class="highlight">${values[i - 1]}</span>${curr}`;
  });
}

const item = "laptop";
const price = 999;
const sentence = highlight`The ${item} costs $${price}.`;
console.log(sentence); 
// "The <span class="highlight">laptop</span> costs $<span class="highlight">999</span>."
```
*   **Applications**: Styling libraries (like styled-components), SQL query sanitization to prevent injection, and internationalization.

---

## 12. "eval is evil". Why?

The `eval()` function evaluates JavaScript code represented as a string. It is universally considered a bad practice to use it.

```javascript
eval("var temp = 5 + 5;");
console.log(temp); // 10
```

### Three major reasons to avoid `eval()`:
1. **Security Vulnerabilities**:
   If string inputs are user-generated, running them through `eval()` allows execution of arbitrary malicious code (Cross-Site Scripting / XSS).
2. **Performance Degradation**:
   Modern JS engines optimize execution by compiling JavaScript to machine code. They do this by analyzing scope structures at compile-time. Because `eval()` can introduce new variables and change variable scopes at runtime, it forces the engine to disable these optimizations, leading to slow performance.
3. **Scoping Issues and Readability**:
   `eval()` makes code hard to read and debug. In non-strict mode, it can modify local scopes in unexpected ways, making code execution unpredictable.

---

## Practical Checkpoint

Try to run this code snippet in your head (or a console) to verify your understanding of TDZ, scoping, and logical operators:

```javascript
let count = 0;
const increment = () => {
  count += 1;
  return count;
};

// What does this print?
console.log(0 && increment()); 
console.log(count);            
console.log(5 || increment()); 
console.log(count);            
```
