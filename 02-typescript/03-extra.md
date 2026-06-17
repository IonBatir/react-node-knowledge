# TypeScript: Advanced Features & Configuration

This file covers TypeScript enums, compilation mechanics, ambient declaration files (`.d.ts`), namespaces, and structural type guidelines.

---

## 1. Enums (Enumerations)

Enums allow you to define a set of named constants. Unlike most TypeScript features, standard enums are **not** just compile-time features; they compile to actual JavaScript objects.

### A. Number Enums
Numeric enums auto-increment their values starting from `0` unless initialized otherwise.
```typescript
enum Role {
  Admin,  // 0
  User,   // 1
  Guest   // 2
}
```
*   **Reverse Mapping**: Numeric enums allow you to map from a value back to its key name at runtime.
    ```typescript
    console.log(Role.Admin);    // 0
    console.log(Role[0]);       // "Admin" (Reverse lookup)
    ```

### B. String Enums
String enums require each member to be initialized with a string literal. They do **not** support reverse mapping.
```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
console.log(Direction.Up); // "UP"
// console.log(Direction["UP"]); // Error: String enums do not support reverse lookup
```

### C. Constant (Static) vs Computed Enums
Enums are divided into static (constant) members and computed members:
*   **Constant members**: Evaluated at compile-time (e.g. literals, binary operators, or reference to other constant enums).
*   **Computed members**: Evaluated at runtime (e.g. calling functions or accessing property lengths).
```typescript
enum FileAccess {
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write, // Constant (resolved at compile time)
  G = "123".length          // Computed (resolved at runtime)
}
```

---

## 2. Enum Compilation Behavior (`const enum` Optimization)

### Standard Enum Transpilation:
Standard enums generate an Immediately Invoked Function Expression (IIFE) in JavaScript, creating an object at runtime.
```javascript
// Output JS for standard 'enum Role { Admin }':
var Role;
(function (Role) {
    Role[Role["Admin"] = 0] = "Admin";
})(Role || (Role = {}));
```

### `const enum` (Zero Runtime Overhead):
If you do not need reverse lookup and want to avoid the code overhead of the IIFE, you can prefix the enum with `const`. The compiler will completely erase the enum definition and inline the literal values directly into your output code.
```typescript
const enum Status {
  Active = 1,
  Inactive = 0
}
const currentStatus = Status.Active;
```
*   **Transpiled Output JS**:
    ```javascript
    const currentStatus = 1; // Inlined directly!
    ```

---

## 3. Why Declaration Files (`.d.ts`) are Needed

TypeScript needs to know the shapes of external code (like Lodash or React) to prevent compilation errors.
*   **Purpose**: A declaration file (`.d.ts`) acts as a "header file" containing **only type information** (no execution code).
*   **Use Cases**:
    1.  Importing plain JavaScript libraries into a TypeScript project.
    2.  Publishing a library written in TypeScript so that consumers using pure JS or TS get autocompletion.
*   **Generation**: Running `tsc --declaration` (or setting `"declaration": true` in `tsconfig.json`) automatically outputs `.d.ts` files alongside compiled `.js` files.

---

## 4. The `declare` Keyword

The `declare` keyword is used to write **ambient declarations**. It tells the compiler: *"This variable/function/class already exists at runtime in the global scope (e.g. from a CDN link or global script), so do not compile it. Just trust that it is there for type checking."*

```typescript
// Tells TS that a global variable 'kakao' exists (e.g. Kakao maps script)
declare const kakao: any;

// Tells TS that a global function exists
declare function loadExternalScript(url: string): Promise<void>;
```
*   **Compile behavior**: `declare` statements are stripped out completely and produce **zero** JavaScript code.

---

## 5. Namespaces

Namespaces (historically called "Internal Modules") are a TypeScript-specific way to organize code and prevent global namespace pollution.

```typescript
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
  
  export const numberRegexp = /^[0-9]+$/;
  
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string): boolean {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}

// Usage
const validator = new Validation.ZipCodeValidator();
```

> [!WARNING]
> **Modern Best Practice: Modules over Namespaces**
> Namespaces are largely obsolete in modern TypeScript. You should use **ES Modules** (`import`/`export`) to structure your code. ES modules are native to JavaScript, handle dependencies cleaner, and work naturally with bundlers (Webpack, Vite, Rollup) to enable tree-shaking.

---

## 6. Best Practices for Typings

1.  **Let TypeScript Infer Types**:
    Do not over-annotate. If the compiler can automatically infer the type, let it do so.
    ```typescript
    // Bad (redundant annotation)
    const count: number = 5; 
    
    // Good (inferred as number)
    const count = 5; 
    ```
2.  **Avoid `any`**:
    Using `any` turns off type checking. Use `unknown` if you don't know the incoming type, and narrow it later.
3.  **Strict Mode**:
    Always enable `"strict": true` in your `tsconfig.json`. This enables rules like `strictNullChecks` and `noImplicitAny`, catching potential runtime crashes.
4.  **Interface vs Type Alias Best Practices**:
    *   Use **Interfaces** for public API definitions, object models, and classes that implement them (they support declaration merging, allowing extensions).
    *   Use **Types** for union types, intersection types, tuples, and mapped types.
5.  **Use Readonly Generics for Arrays**:
    When passing arrays to functions that shouldn't mutate them, type them as `ReadonlyArray<T>` or `readonly T[]`.

---

## Practical Checkpoint

Predict what happens when you transpile the following code containing a standard enum and a `const enum` accessing them dynamic-style:

```typescript
const enum Config {
  Max = 100,
  Min = 10
}

// Can we access:
console.log(Config["Max"]);
// What happens if we try:
// const key = "Max";
// console.log(Config[key]);
```
