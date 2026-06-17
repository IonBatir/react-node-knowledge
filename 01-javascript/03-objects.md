# JavaScript Internals: Objects & Memory Management

This file provides a comprehensive guide to JavaScript objects, including manipulation, structure, advanced descriptors, deep copying, collections (`Map`/`Set`/`WeakMap`/`WeakSet`), metaprogramming (`Proxy`/`Reflect`), and iteration.

---

## 1. Object as a Reference Type

In JavaScript, objects (including arrays and functions) are **reference types**.
*   **Memory Allocation**: The variable itself is stored on the **stack** and holds a pointer (memory address). The actual object data is stored in the **heap**.
*   **Comparison**: Two objects are only equal if they refer to the exact same location in memory.

```javascript
const objA = { value: 10 };
const objB = { value: 10 };
const objC = objA; // Copies the reference address

console.log(objA === objB); // false (different addresses in the heap)
console.log(objA === objC); // true (same reference address)

objC.value = 20;
console.log(objA.value); // 20 (mutating objC affects objA because they point to the same object)
```

---

## 2. CRUD on Object Properties & Methods

You can read, write, update, and delete properties using **dot notation** or **bracket notation**.

*   **Dot notation**: `obj.key` (requires the key to be a valid identifier).
*   **Bracket notation**: `obj["key"]` (allows dynamic evaluation, symbols, spaces, or special characters).
*   **Deleting**: The `delete` operator removes a property from an object. It returns `true` if it successfully deletes it or if the property doesn't exist (unless the property is non-configurable).

```javascript
const user = { name: "Alice" };

// Set/Update
user.age = 25;                  // Dot notation
user["favorite-food"] = "Sushi"; // Bracket notation (special character)

// Read/Get
console.log(user.name);               // "Alice"
console.log(user["favorite-food"]);   // "Sushi"

// Methods (functions inside objects)
user.greet = function() {
  return `Hi, I'm ${this.name}`;
};

// Delete
delete user.age;
console.log(user.age); // undefined
```

---

## 3. Getting Object Keys & Values

JavaScript offers several methods to inspect keys, values, and property configurations:

1.  **`Object.keys(obj)`**: Returns an array of an object's own **enumerable string-keyed** property names.
2.  **`Object.values(obj)`**: Returns an array of own enumerable values.
3.  **`Object.entries(obj)`**: Returns an array of own enumerable `[key, value]` pairs.
4.  **`Object.getOwnPropertyNames(obj)`**: Returns an array of **all** own string properties (enumerable and non-enumerable).
5.  **`Object.getOwnPropertySymbols(obj)`**: Returns an array of all own Symbol properties.
6.  **`Reflect.ownKeys(obj)`**: Returns all own keys (string and symbol, enumerable and non-enumerable).

```javascript
const idSym = Symbol("id");
const data = {
  [idSym]: 123,
  name: "Laptop"
};
Object.defineProperty(data, "hidden", { value: "secret", enumerable: false });

console.log(Object.keys(data));             // ["name"] (ignores symbols & non-enumerables)
console.log(Object.getOwnPropertyNames(data)); // ["name", "hidden"]
console.log(Reflect.ownKeys(data));         // ["name", "hidden", Symbol(id)]
```

---

## 4. `for...in` Loops and Prototype Chain

The `for...in` statement iterates over **all enumerable properties** of an object, **including inherited enumerable properties** along the prototype chain.

```javascript
const parent = { parentProp: "I am inherited" };
const child = Object.create(parent); // inherits parent
child.childProp = "I am own";

for (const key in child) {
  console.log(key); // Prints BOTH "childProp" and "parentProp"
}

// To filter out inherited properties:
for (const key in child) {
  if (child.hasOwnProperty(key)) {
    console.log(key); // Prints only "childProp"
  }
}
```

---

## 5. Ways to Create Objects

1.  **Object Literal**: The most common way.
    ```javascript
    const obj = { x: 1 };
    ```
2.  **Constructor Function**: Historically used to mimic classes.
    ```javascript
    function User(name) {
      this.name = name;
    }
    const user = new User("Alice");
    ```
3.  **`Object.create(proto)`**: Creates a new object with the specified prototype object.
    ```javascript
    const protoObj = { greet() { return "Hi"; } };
    const instance = Object.create(protoObj);
    ```
4.  **ES6 Classes**: Modern syntactic sugar over constructor functions.
    ```javascript
    class Person {
      constructor(name) { this.name = name; }
    }
    const p = new Person("Bob");
    ```

---

## 6. Destructuring & the Spread Operator

### Destructuring
Extracts data from arrays or objects into distinct variables.
```javascript
const config = { width: 100, height: 200, title: "App" };

// Destructuring with defaults and renaming
const { width: w, height: h, mode = "dark" } = config;
console.log(w, h, mode); // 100, 200, "dark"
```

### Spread Operator (`...`)
Shallow-copies enumerable properties from one object into another.
```javascript
const base = { isDark: true, color: "blue" };
const theme = { ...base, color: "red" }; // overrides color
console.log(theme); // { isDark: true, color: "red" }
```

---

## 7. Optional Chaining (`?.`)

The optional chaining operator `?.` allows you to read properties deep within an object chain without manually verifying that each reference in the chain is valid. If a reference is `null` or `undefined`, the evaluation short-circuits and returns `undefined`.

```javascript
const user = {
  profile: {
    // getAge() is omitted
  }
};

// Without optional chaining:
// const age = user.profile.getAge(); // TypeError: user.profile.getAge is not a function

// With optional chaining:
const age = user.profile?.getAge?.(); 
console.log(age); // undefined (safe!)

const street = user.address?.street;
console.log(street); // undefined (safe!)
```

---

## 8. Object to Primitive Conversion

When an object is used in operations expecting a primitive (e.g., alert, subtraction, addition), JavaScript converts it using three algorithm hints: `"string"`, `"number"`, or `"default"`.

The engine searches for methods in this order:
1.  **`obj[Symbol.toPrimitive](hint)`** (if it exists).
2.  Otherwise, if hint is `"string"`: tries `toString()` first, then `valueOf()`.
3.  Otherwise (hint is `"number"` or `"default"`): tries `valueOf()` first, then `toString()`.

```javascript
const item = {
  name: "Book",
  price: 50,
  
  [Symbol.toPrimitive](hint) {
    console.log(`Hint: ${hint}`);
    return hint === "string" ? this.name : this.price;
  }
};

console.log(String(item)); // "Book" (Hint: string)
console.log(item + 10);    // 60     (Hint: default)
console.log(+item);        // 50     (Hint: number)
```

---

## 9. Property Descriptors

Every property in a JavaScript object is configured via attributes called **property descriptors**. There are two types: **Data descriptors** and **Accessor descriptors**.

To view descriptors: `Object.getOwnPropertyDescriptor(obj, prop)`
To set descriptors: `Object.defineProperty(obj, prop, descriptor)`

### Data Descriptors:
*   `value`: The property value.
*   `writable`: If `true`, the value can be changed.
*   `enumerable`: If `true`, the property shows up in loops and `Object.keys()`.
*   `configurable`: If `true`, the property descriptor can be modified, and the property can be deleted from the object.

```javascript
const user = {};
Object.defineProperty(user, "readOnlyName", {
  value: "System",
  writable: false,
  enumerable: true,
  configurable: false
});

// user.readOnlyName = "Hack"; // Fails silently (throws TypeError in strict mode)
// delete user.readOnlyName;   // Fails silently (throws TypeError in strict mode)
```

---

## 10. Getters and Setters (Accessor Descriptors)

Accessor descriptors do not contain a `value` or `writable` attribute. Instead, they define dynamic functions for property access.
*   `get`: A function called when the property is read.
*   `set`: A function called when the property is written to.

```javascript
const wallet = {
  dollars: 100,
  
  get euros() {
    return this.dollars * 0.92;
  },
  
  set euros(value) {
    this.dollars = value / 0.92;
  }
};

console.log(wallet.euros); // 92
wallet.euros = 184;
console.log(wallet.dollars); // 200
```

---

## 11. Mutable vs Immutable Objects

By default, objects are mutable. JavaScript provides three built-in methods to restrict object modifications:

| Method | Can Add Properties? | Can Delete Properties? | Can Modify Property Values? |
| :--- | :--- | :--- | :--- |
| **`Object.preventExtensions(obj)`** | ❌ No | Yes | Yes |
| **`Object.seal(obj)`** | ❌ No | ❌ No | Yes |
| **`Object.freeze(obj)`** | ❌ No | ❌ No | ❌ No |

> [!WARNING]
> All these operations are **shallow**. Nested objects inside a frozen object can still be mutated unless you recursively freeze them.

```javascript
const deepObj = { nested: { count: 0 } };
Object.freeze(deepObj);
deepObj.nested.count = 5; // Success! The nested object is NOT frozen.
```

---

## 12. Map vs Set

### Map
A collection of keyed data items, similar to an Object, but with crucial differences:
*   **Keys**: Any value (including functions, objects, or primitives) can be a key.
*   **Ordering**: Elements are iterated in insertion order.
*   **Performance**: Optimized for frequent additions and removals.
```javascript
const map = new Map();
const keyObj = {};
map.set(keyObj, "metadata");
console.log(map.get(keyObj)); // "metadata"
```

### Set
A collection of **unique values** (no duplicates allowed).
```javascript
const uniqueSet = new Set([1, 1, 2, 3]);
console.log(uniqueSet.size); // 3 (duplicate 1 is removed)
```

---

## 13. Deep Copying & `structuredClone`

Because objects are copied by reference, we often need to clone them.

*   **Shallow Copy**: Copies only the top-level references. Nested objects remain shared.
    *   Methods: `{ ...obj }`, `Object.assign({}, obj)`.
*   **Deep Copy**: Copies every nesting layer, ensuring complete structural isolation.
    *   *Approach 1: `JSON.parse(JSON.stringify(obj))`*
        *   **Cons**: Loses functions, undefined values, Symbols, Dates (converted to string), RegExps, Map, and Set.
    *   *Approach 2: Recursive Deep Clone*
        *   **Cons**: Complex to write correctly to handle circular references.
    *   *Approach 3 (Modern Standard): `structuredClone(obj)`*
        *   Introduced globally in all modern browsers and Node 17+. It clones complex data types (Date, RegExp, Map, Set, ArrayBuffer) and handles circular references automatically.
        *   **Cons**: Still cannot clone functions or DOM elements (throws an error).

```javascript
const original = {
  date: new Date(),
  set: new Set([1, 2]),
  self: null
};
original.self = original; // Circular reference!

const clone = structuredClone(original);
console.log(clone.date instanceof Date); // true
console.log(clone.self === clone);      // true (retained circular reference map)
```

---

## 14. WeakMap & WeakSet

`WeakMap` and `WeakSet` are special variations of `Map` and `Set` that do not prevent the garbage collector from reclaiming keys.

### Differences from standard Map/Set:
1.  **Keys must be objects** (or in recent engines, registered symbols). Primitives are forbidden.
2.  **Weak references**: Keys are held weakly. If there are no other references to a key object, the garbage collector will reclaim it, along with its associated entry in the WeakMap.
3.  **Non-iterable**: You cannot run `keys()`, `values()`, `entries()`, or `.size` because the garbage collector operates asynchronously, making the size of the collection unpredictable.

### Primary Use Case: Metadata storage / Caching
If you want to associate metadata with a DOM element or a user session without causing memory leaks when that element/session is deleted:

```javascript
let user = { name: "John" };
const sessionData = new WeakMap();
sessionData.set(user, { loginTime: Date.now() });

user = null; // The object is garbage collected. sessionData automatically cleans up the entry.
```

---

## 15. Proxy & Reflect

### Proxy
The `Proxy` object wraps a target object, allowing you to intercept and customize core operations (like getting/setting properties, calling constructors, etc.). This pattern is widely used in reactive frameworks (like Vue 3).

### Reflect
`Reflect` is a built-in object that provides static methods for interceptable JavaScript operations, matching the exact trap signatures of `Proxy` handlers. It returns standard boolean success flags instead of throwing errors.

```javascript
const target = { count: 0 };
const proxy = new Proxy(target, {
  get(obj, prop) {
    console.log(`Reading property: ${prop}`);
    return Reflect.get(obj, prop); // delegates action safely
  },
  
  set(obj, prop, value) {
    if (prop === "count" && value < 0) {
      throw new Error("Count cannot be negative");
    }
    console.log(`Writing property: ${prop} = ${value}`);
    return Reflect.set(obj, prop, value);
  }
});

proxy.count = 10; // Writing property: count = 10
console.log(proxy.count); // Reading property: count \n 10
// proxy.count = -1; // Throws: Count cannot be negative
```

---

## 16. Iterable Objects

An object is considered **iterable** if it defines a method at the `Symbol.iterator` key. This method must return an **iterator** object which implements the iterator protocol (returning `{ value, done }` via a `.next()` method).

```javascript
const countdown = {
  from: 3,
  
  [Symbol.iterator]() {
    let current = this.from;
    return {
      next() {
        if (current >= 0) {
          return { value: current--, done: false };
        }
        return { done: true };
      }
    };
  }
};

for (const num of countdown) {
  console.log(num); // 3, 2, 1, 0
}
```

---

## 17. Garbage Collection & Memory Management Internals

### What is the Garbage Collector (GC)? Why do we need it?
JavaScript manages memory automatically through an execution runtime feature called the **Garbage Collector (GC)**.
*   **Purpose**: The GC monitors memory allocation, identifies when blocks of allocated memory are no longer needed by the application, and releases them back to the operating system.
*   **Why we need it**: In low-level languages like C/C++, developers must manually manage memory using functions like `malloc()` and `free()`. Forgetting to free memory leads to **memory leaks**, while freeing memory prematurely causes **dangling pointers** and security vulnerabilities. JavaScript abstracts this away to prevent these errors, ensuring memory safety.

---

### Object Pointers Reachability & Graph Roots
The JS engine determines whether memory is "needed" based on the concept of **reachability**. A value is reachable if it is accessible or usable in some way from a set of baseline references called **roots**.

#### 1. Garbage Collection Roots:
*   Local variables and parameters in the current execution stack.
*   Variables in the scope chain (closures).
*   Global variables (e.g., properties on the `window` object in browsers or the `global` object in Node.js).

#### 2. The Reference Graph:
The GC starts from these roots, looks at all pointers contained inside them, and recursively traverses the reference chain to build a graph of reachable objects. Any object not connected to the root graph is marked for garbage collection.

#### How to mark a pointer for garbage collection:
To make an object eligible for garbage collection, we must ensure it is no longer reachable from any root. This can be done by:
*   Allowing the variable holding the pointer to **go out of scope** (e.g., when a function execution terminates, all local variables are popped off the stack).
*   Explicitly **severing the reference** by reassigning the variable to `null` or `undefined` (e.g., `myObject = null;`).

---

### The "Mark and Sweep" Algorithm
Modern JS engines use the **Mark and Sweep** algorithm instead of the older "Reference Counting" algorithm (which failed on circular reference loops).

The algorithm runs in two primary phases:
1.  **Mark Phase**: The GC starts from the roots and traverses the reference graph. Every object visited is "marked" (usually by setting a specific bit flag in its memory header).
2.  **Sweep Phase**: The GC scans the heap memory sequentially. Any object that is *not* marked is reclaimed (swept) and its memory is added back to the allocation lists. The flags of marked objects are then cleared for the next cycle.

```
[ Roots (Global / Stack) ]
       │          │
       ▼          ▼
   [ Object A ] [ Object B ]
       │
       ▼
   [ Object C ]            [ Object D ] (No reference path from Roots)
                                           ↳ Reclaimed by Sweep phase!
```

---

### Advanced GC Optimizations (V8 Engine)
Running a full heap search is computationally expensive and causes "Stop-The-World" pauses where code execution freezes. To minimize this, engines like V8 use advanced techniques:

1.  **Generational Collection**:
    Objects are split into two categories based on their lifespan:
    *   **Young Generation (Scavenge)**: Stored in a small, fast memory space (1-8 MB). Most objects die young. The Scavenger algorithm frequently clears this space by copying surviving objects to a second buffer and reclaiming the first. Objects that survive multiple rounds are promoted to the Old Generation.
    *   **Old Generation (Major GC)**: Contains long-lived objects. Checked less frequently because it is much larger.
2.  **Incremental Marking**:
    Instead of pausing execution to mark the entire heap at once, the GC splits the marking phase into multiple small steps. It interleaves marking actions with regular code execution. To track changes made by the application between GC steps, V8 uses a "Write Barrier" mechanism.
3.  **Idle-Time Collection**:
    The GC coordinates with the event loop to perform garbage collection tasks when the CPU is idle (e.g., during frame drops or while waiting for user input), ensuring maximum rendering performance (60fps/120fps).

---

## Practical Checkpoint

Predict the behavior of the following snippet:

```javascript
let element = { id: "main-btn" };
const registry = new Map();
const weakRegistry = new WeakMap();

registry.set(element, "active");
weakRegistry.set(element, "active");

// We lose the reference variable
element = null; 

// Will the object { id: "main-btn" } be garbage collected? Why or why not?
```
