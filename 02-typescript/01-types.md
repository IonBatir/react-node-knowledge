# TypeScript: Type System & Typing Strategies

This file provides a comprehensive guide to TypeScript's type system, basic types, assertions, interface designs, generic structures, function typing, and advanced hybrid layouts.

---

## 1. Basic Types: `void`, `never`, `unknown`, and Tuples

TypeScript introduces specific types to handle edge cases in JavaScript's dynamic runtime.

### A. `void`
Represents the absence of any value. It is typically used as the return type of functions that do not return a value.
```typescript
function logMessage(msg: string): void {
  console.log(msg); // Returns undefined implicitly, but typed as void
}
```

### B. `never`
Represents the type of values that **never occur**.
*   **Use Case 1: Infinite Loops or Unconditional Errors**:
    ```typescript
    function throwError(msg: string): never {
      throw new Error(msg);
    }
    ```
*   **Use Case 2: Exhaustive Checks in Switch Blocks**:
    ```typescript
    type Shape = "circle" | "square";
    function getArea(shape: Shape) {
      switch (shape) {
        case "circle": return 1;
        case "square": return 2;
        default:
          const _exhaustiveCheck: never = shape; // Compile-time check
          return _exhaustiveCheck;
      }
    }
    ```

### C. `unknown`
The type-safe counterpart of `any`. You can assign any value to `unknown`, but you **cannot perform operations** on it, call methods on it, or assign it to other typed variables without first narrowing the type (type checking).
```typescript
let value: unknown = "Hello";
// value.toUpperCase(); // Error: Object is of type 'unknown'

if (typeof value === "string") {
  console.log(value.toUpperCase()); // Safe! Type narrowed to string.
}
```

### D. Tuples
A tuple is an array with a **fixed number of elements** and **specific types at designated indexes**.
```typescript
let user: [number, string] = [1, "Alice"];
// user = ["Alice", 1]; // Error: Type string is not assignable to type number
```

---

## 2. Union & Literal Types

### Union Types (`|`)
Allow a variable to hold values of multiple distinct types.
```typescript
let id: string | number;
id = "123"; // Ok
id = 123;   // Ok
```

### Literal Types
Restrict a variable to hold only a specific, exact value (string, number, or boolean). Often combined with unions to model state enums:
```typescript
type Direction = "North" | "East" | "South" | "West";
let move: Direction = "North";
// move = "Up"; // Error: Type '"Up"' is not assignable to type 'Direction'
```

---

## 3. Type Casting (Type Assertions)

Type assertions tell the TypeScript compiler: *"Trust me, I know what the type of this value is better than you do."* It performs no runtime checks or casting; it is purely a compile-time tool.

### Syntax Options:
1.  **`as` Syntax** (Recommended, especially for TSX):
    ```typescript
    const rawData: unknown = "hello";
    const strLen = (rawData as string).length;
    ```
2.  **Angle-bracket Syntax**:
    ```typescript
    const strLen = (<string>rawData).length;
    ```

### Double Assertion (Escape Hatch):
If you need to cast a type to an unrelated type, you must cast it to `unknown` or `any` first:
```typescript
const value = "100";
// const num = value as number; // Error: String and Number do not overlap
const num = value as unknown as number; // Success
```

---

## 4. `void` vs `undefined | null`

While `void` and `undefined` seem similar, they are not aliases for each other:
*   A variable of type `void` can only be assigned `undefined` (or `null` if `strictNullChecks` is disabled).
*   **Callback Return Typing**: If a callback is typed to return `void`, TypeScript **allows** it to return any value. However, the calling code will treat the return value as unusable (`void`). This allows array methods like `forEach` (which expects a void callback) to accept callbacks that return values (like `push`).
```typescript
type VoidCallback = () => void;
const runCallback: VoidCallback = () => {
  return 42; // Allowed!
};
const res = runCallback(); // res is typed as void, you cannot use it
```

---

## 5. Interface vs Class

A fundamental distinction in TypeScript is what exists at runtime:

| Feature | Interface | Class |
| :--- | :--- | :--- |
| **Runtime Presence** | **None** (Fully compiled away/erased) | **Yes** (Generates a standard JS prototype constructor) |
| **Purpose** | Pure type checking & structural contracts | Object blueprints, instantiation, logic encapsulation |
| **Usage** | Declaring object shapes & function contracts | Instantiating objects using `new`, inheritances |
| **Merging** | Supports declaration merging | Does not support declaration merging |

---

## 6. Optional & Readonly Properties

TypeScript allows fine-grained control over property access:

*   **Optional (`?`)**: Properties that do not need to be present on the object.
*   **Readonly (`readonly`)**: Properties that can only be written to when the object is initialized.
```typescript
interface Car {
  readonly brand: string;
  model: string;
  color?: string; // Optional
}

const myCar: Car = { brand: "Tesla", model: "Model 3" };
// myCar.brand = "Ford"; // Error: Cannot assign to 'brand' because it is a read-only property
```

---

## 7. Advanced Interfaces

### A. Function Types in Interfaces
You can define an interface that represents a function structure.
```typescript
interface MathOperation {
  (x: number, y: number): number;
}
const add: MathOperation = (a, b) => a + b;
```

### B. Indexable Types (Index Signatures)
Used to describe objects where you don't know the property names in advance but know the type of keys and values.
```typescript
interface StringDictionary {
  [key: string]: string; // Key index signature
}
const dict: StringDictionary = { hello: "world", foo: "bar" };
```

### C. Construct Signature
Interfaces can describe objects that can be instantiated with `new`. This is useful for describing class constructors passed as arguments.
```typescript
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
  tick(): void;
}

function createClock(ctor: ClockConstructor, h: number, m: number): ClockInterface {
  return new ctor(h, m);
}
```

### D. Hybrid Interfaces
An object that acts as both a function and an object with additional properties (common in legacy libraries like jQuery).
```typescript
interface Counter {
  (start: number): string; // Function signature
  interval: number;        // Property
  reset(): void;           // Method
}

function getCounter(): Counter {
  const c = function (start: number) { return `Started at ${start}`; } as Counter;
  c.interval = 123;
  c.reset = () => { c.interval = 0; };
  return c;
}
```

---

## 8. Classes as Interfaces

In TypeScript, classes create two things: a **value** (the constructor function) and a **type** (the instance shape). This means you can use a class as an interface.
```typescript
class Point {
  x = 0;
  y = 0;
}

// Implement Point's shape without inheriting its constructor logic
class NamedPoint implements Point {
  x = 10;
  y = 20;
  name = "Center";
}
```

---

## 9. Typing Functions

Functions can be annotated with parameter types and return types.

### A. Optional Parameters (`?`) and Rest Parameters (`...`)
*   Optional parameters must follow mandatory parameters.
*   Rest parameters are typed as arrays.
```typescript
function greet(name: string, greeting?: string, ...aliases: string[]): string {
  return `${greeting ?? "Hello"} ${name}! Also known as ${aliases.join(", ")}`;
}
```

### B. Typing `this`
You can declare a dummy parameter named `this` at the very beginning of the parameter list to type the execution context of the function:
```typescript
interface Button {
  id: string;
}
function handleClick(this: Button, event: Event) {
  console.log(`Clicked button: ${this.id}`);
}
```

### C. Function Overloads
Provides multiple signatures for a single function implementation, helping the compiler match different call patterns.
```typescript
// Overload Signatures
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

// Implementation Signature (invisible to callers)
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}

makeDate(12345678); // Ok
makeDate(5, 5, 2026); // Ok
// makeDate(5, 5); // Error: No overload expects 2 arguments.
```

---

## 10. Generics (`<T>`)

Generics allow you to write reusable code that can work with a variety of types while maintaining compile-time type safety.

### A. Simple Generic Functions
```typescript
function identity<T>(arg: T): T {
  return arg;
}
const output = identity<string>("hello"); // T is explicitly 'string'
const implicit = identity(42);            // T is inferred as 'number'
```

### B. Generic Classes
```typescript
class Box<T> {
  contents: T;
  constructor(value: T) {
    this.contents = value;
  }
}
const stringBox = new Box("Secrets"); // Box<string>
```

### C. Generic Constraints (`extends`)
Restrict the types that a generic parameter can accept.
```typescript
interface Lengthwise {
  length: number;
}

// T must be an object that has a .length property
function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

loggingIdentity("hello"); // Ok (string has .length)
// loggingIdentity(3);    // Error: number does not have .length
```

### D. Using Class Types in Generics (Constructors)
To refer to class constructors in generics, use a construct signature matching the class constructor:
```typescript
function createInstance<T>(ctor: new () => T): T {
  return new ctor();
}

class Animal { name = "Generic Animal"; }
const myAnimal = createInstance(Animal); // Instantiates Animal safely
```

---

## Practical Checkpoint

Verify your understanding of generics and constraints. Will the following code compile? Why or why not?

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

const x = { a: 1, b: 2, c: 3 };

getProperty(x, "a");
getProperty(x, "m");
```
