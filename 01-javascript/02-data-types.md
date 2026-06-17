# JavaScript Basics: Data Types

This file provides a comprehensive guide to JavaScript's data types, type conversions, equality mechanics, and modern primitive features like `BigInt` and `Symbol`.

---

## 1. List of Data Types

JavaScript is a dynamically and weakly typed language. Dynamic means variables don't have types; values do. Weakly typed means it performs implicit type conversions (coercion) when operators expect different types.

JavaScript's types are divided into **Primitives** and **Objects (Reference Types)**.

### A. Primitive Types (Immutable, Stored by Value)
Primitives are stored directly on the stack (or inline in optimized memory). They are copied by value.
1.  **`number`**: Double-precision 64-bit binary format IEEE 754 (includes `Infinity`, `-Infinity`, and `NaN`).
2.  **`string`**: Sequence of UTF-16 code units.
3.  **`boolean`**: `true` or `false`.
4.  **`undefined`**: Automatically assigned to declared variables that haven't been initialized.
5.  **`null`**: Represents the intentional absence of any object value.
6.  **`bigint`**: Arbitrary-precision integers (safely operates past the standard 64-bit integer limit).
7.  **`symbol`**: Unique, immutable identifier.

### B. Objects / Reference Types (Mutable, Stored by Reference)
Objects are stored on the heap. Variables contain a reference pointer pointing to the heap memory.
*   **Plain Objects** (`{}`), **Arrays** (`[]`), **Functions**, **Dates**, **Maps**, **Sets**, etc.

```javascript
// Primitive: value copy
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 (unchanged)

// Object: reference copy
let obj1 = { name: "Alice" };
let obj2 = obj1;
obj2.name = "Bob";
console.log(obj1.name); // "Bob" (mutated via the shared reference)
```

---

## 2. Strict (`===`) vs Non-Strict (`==`) Comparison

JavaScript has three comparison operations:
1.  **Abstract/Loose Equality (`==`)**: Performs type coercion if the types are different, then compares them.
2.  **Strict Equality (`===`)**: Does not perform type coercion. Returns `false` if types differ.
3.  **Same-Value Equality (`Object.is`)**: Similar to `===` but handles edge cases differently (specifically `NaN` and signed zeros `+0`/`-0`).

### Equality Rules for `==`
If types differ, JS coerces values following a complex set of rules:
*   Comparing `string` and `number`: Coerces string to number.
*   Comparing `boolean` to anything: Coerces boolean to number (`true` -> `1`, `false` -> `0`).
*   Comparing `null` and `undefined`: Returns `true` (`null == undefined`). They do not coerce to anything else for `==`.
*   Comparing `object` to primitive: Coerces object to primitive using its `[Symbol.toPrimitive]`, `valueOf()`, or `toString()` methods.

```javascript
console.log(1 == "1");       // true (coercion occurs)
console.log(1 === "1");      // false (different types)

console.log(false == 0);     // true (false coerces to 0)
console.log(null == 0);      // false (null only equals null or undefined in loose comparison)
```

### The `Object.is()` Exception
There are two places where `===` and `Object.is()` differ:

```javascript
// NaN equality
console.log(NaN === NaN);        // false (per IEEE 754 specification)
console.log(Object.is(NaN, NaN)); // true

// Signed Zeros
console.log(+0 === -0);          // true
console.log(Object.is(+0, -0));   // false
```

---

## 3. `null` vs `undefined`

While both represent empty or missing values, they are used differently:

| Feature | `undefined` | `null` |
| :--- | :--- | :--- |
| **Meaning** | Declared but not initialized (missing) | Intentional absence of value (empty) |
| **Type** | `"undefined"` | `"object"` (A historical JavaScript engine quirk) |
| **Arithmetic** | Coerces to `NaN` (`undefined + 1` is `NaN`) | Coerces to `0` (`null + 1` is `1`) |
| **JSON** | Omitted during serialization | Kept as `null` |

```javascript
let testVar;
console.log(testVar); // undefined

let emptyRef = null;
console.log(emptyRef); // null

console.log(typeof undefined); // "undefined"
console.log(typeof null);      // "object"
```

---

## 4. Type Conversion (Coercion)

Coercion can be **explicit** (done manually) or **implicit** (done automatically by the JS engine under certain operators).

### A. To String
*   **Explicit**: `String(value)` or `value.toString()`.
*   **Implicit**: Occurs with the binary `+` operator if one operand is a string.
    ```javascript
    console.log(5 + "5"); // "55" (Number coerced to String)
    ```

### B. To Number
*   **Explicit**: `Number(value)`, `parseInt(value)`, `parseFloat(value)`.
*   **Implicit**: Occurs with mathematical operators `-`, `*`, `/`, `%`, or unary `+`.
    ```javascript
    console.log(+"10");   // 10 (string coerced to number)
    console.log("10" - 2); // 8 (string coerced to number)
    console.log("ten" - 2);// NaN (cannot coerce "ten" to a valid number)
    ```

### C. To Boolean
*   **Explicit**: `Boolean(value)` or double negation `!!value`.
*   **Truthy/Falsy Rules**: All values are **truthy** except the following **falsy** values:
    *   `false`
    *   `0`, `-0`, `0n` (BigInt zero)
    *   `""` (empty string)
    *   `null`
    *   `undefined`
    *   `NaN`

### D. Classic Quirks & Puzzles
```javascript
console.log([] + []); // "" (both arrays coerce to strings: "" + "")
console.log([] + {}); // "[object Object]" (array -> "", object -> "[object Object]")
console.log({} + []); // 0 or "[object Object]" depending on runtime context/parentheses
console.log([] == ![]); // true 
// Steps:
// 1. ![] becomes false (since [] is truthy)
// 2. [] == false
// 3. Coerce false to 0: [] == 0
// 4. Coerce [] to primitive: "" == 0
// 5. Coerce "" to number: 0 == 0 -> true!
```

---

## 5. `"typeof"` Usage

The `typeof` operator returns a string indicating the type of the unevaluated operand.

### Common Results Table
| Operand | `typeof` Return | Notes |
| :--- | :--- | :--- |
| `42` / `NaN` | `"number"` | Even though `NaN` means "Not a Number", its type is `"number"`. |
| `"hello"` | `"string"` | |
| `true` | `"boolean"` | |
| `undefined` | `"undefined"` | |
| `null` | `"object"` | A historical bug in Javascript's initial release. |
| `10n` | `"bigint"` | |
| `Symbol("id")` | `"symbol"` | |
| `function() {}`| `"function"` | Special object return type. |
| `[]` / `{}` / `new Set()` | `"object"` | Arrays, Maps, Sets are all structurally `"object"`. |

### How to detect arrays safely:
Since `typeof []` returns `"object"`, we must use helper methods:
```javascript
const arr = [1, 2, 3];
console.log(Array.isArray(arr)); // true
console.log(arr instanceof Array); // true
```

---

## 6. BigInt

Introduced in ES2020, `BigInt` allows you to store and operate on integers beyond the safe integer limit for standard Numbers (`Number.MAX_SAFE_INTEGER` = $2^{53} - 1 = 9007199254740991$).

### Creating a BigInt:
Append `n` to the end of an integer literal, or call the `BigInt()` function.
```javascript
const maxSafe = 9007199254740991n; 
const bigNum = BigInt("900719925474099123456789");
```

### Key Restrictions:
1.  **No Mixing**: You cannot mix `BigInt` with normal `Number` objects in operations (to avoid losing precision implicitly). You must explicitly convert them.
    ```javascript
    // const badMath = 5n + 10; // TypeError: Cannot mix BigInt and other types
    const goodMath = 5n + BigInt(10); // 15n
    ```
2.  **No decimals**: Division rounds towards zero.
    ```javascript
    console.log(5n / 2n); // 2n (decimal portion is truncated)
    ```

---

## 7. Symbol Type

A `Symbol` is an immutable primitive value that is guaranteed to be unique.

### Core Syntax
```javascript
const sym1 = Symbol("description");
const sym2 = Symbol("description");

console.log(sym1 === sym2); // false (each Symbol call creates a unique value)
```

### Key Use Cases
1.  **Hidden/Collision-free Property Keys**: Adding properties to objects without risking collision with existing keys or library updates.
    ```javascript
    const USER_ID = Symbol("userId");
    const user = {
      name: "John",
      [USER_ID]: "12345" // unique key
    };
    console.log(user[USER_ID]); // "12345"
    ```
2.  **Exclusion from loops**: Symbol-keyed properties are ignored by `for...in` loops and methods like `Object.keys()`.
    ```javascript
    console.log(Object.keys(user)); // ["name"] (Symbol key is omitted)
    console.log(Object.getOwnPropertySymbols(user)); // [Symbol(userId)] (accessible explicitly)
    ```

### Global Symbol Registry
If you want to share a Symbol across files or code contexts, you can use `Symbol.for(key)` which retrieves or creates a symbol from a global registry.
```javascript
const symA = Symbol.for("app.config");
const symB = Symbol.for("app.config");
console.log(symA === symB); // true
```

### Well-Known Symbols (Metaprogramming)
JavaScript has built-in symbols that customize default language behaviors:
*   `Symbol.iterator`: Defines how an object is iterated by `for...of`.
*   `Symbol.toPrimitive`: Defines how an object is converted to a primitive value.
*   `Symbol.toStringTag`: Defines the custom string description of an object when `toString()` is called.

---

## Practical Checkpoint

What does the following print? Think through the coercion steps:

```javascript
console.log(typeof (typeof null));
console.log([] == 0);
console.log(Number.MIN_VALUE > 0);
console.log("5" + 2 - 1);
```
