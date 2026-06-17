# TypeScript: Class Structures & OOP Integration

This file covers TypeScript's object-oriented programming (OOP) features, access modifiers, parameter properties, abstract classes, and private constructors.

---

## 1. Access Modifiers

TypeScript extends ES6 classes with three access modifiers: `public`, `protected`, and `private`. These control the visibility of class members (properties and methods).

| Modifier | Access inside same class | Access inside subclass | Access outside the class |
| :--- | :---: | :---: | :---: |
| **`public`** (default) | ✅ Yes | ✅ Yes | ✅ Yes |
| **`protected`** | ✅ Yes | ✅ Yes | ❌ No |
| **`private`** | ✅ Yes | ❌ No | ❌ No |

### Code Demonstration:
```typescript
class Person {
  public name: string;
  protected age: number;
  private ssn: string;

  constructor(name: string, age: number, ssn: string) {
    this.name = name;
    this.age = age;
    this.ssn = ssn;
  }
}

class Employee extends Person {
  constructor(name: string, age: number, ssn: string) {
    super(name, age, ssn);
  }

  showInfo() {
    console.log(this.name); // Ok: public
    console.log(this.age);  // Ok: protected (accessible in subclasses)
    // console.log(this.ssn);  // Error: Property 'ssn' is private and only accessible within class 'Person'.
  }
}

const bob = new Person("Bob", 30, "123-45-678");
console.log(bob.name); // Ok: public
// console.log(bob.age);  // Error: Property 'age' is protected and only accessible within class 'Person' and its subclasses.
```

> [!IMPORTANT]
> **Compile-Time vs. Runtime Privacy**
> TypeScript's `private` and `protected` modifiers are **only checked during compilation**. Once transpiled to JavaScript, they disappear, meaning they are standard, public properties at runtime. 
> To enforce true runtime privacy, use modern JavaScript's native private fields (`#` syntax):
> ```typescript
> class Secure {
>   #secret = "12345"; // Immutable and private at runtime
> }
> ```

---

## 2. Parameter Properties

Parameter properties are a shorthand syntax to declare and initialize a class property directly inside the constructor signature, avoiding boilerplate code.

*   By prefixing a constructor parameter with an access modifier (`public`, `protected`, `private`) or `readonly`, TypeScript automatically:
    1.  Declares a property with that name.
    2.  Assigns the argument value to `this.name` when the class is instantiated.

### Boilerplate ES6 Class:
```typescript
class UserES5 {
  name: string;
  constructor(name: string) {
    this.name = name; // manual declaration and assignment
  }
}
```

### TypeScript Parameter Property Shorthand:
```typescript
class UserTS {
  // Automatically declares and assigns this.name, this.age, and this.role
  constructor(
    public name: string,
    private age: number,
    protected readonly role: string
  ) {}
}

const user = new UserTS("Alice", 25, "admin");
console.log(user.name); // "Alice"
```

---

## 3. Abstract Classes

An **abstract class** is a base class that cannot be instantiated directly using the `new` operator. It acts as a contract and template for subclasses.

*   **Abstract methods**: Methods declared inside an abstract class that do not have an implementation. Subclasses **must** implement these methods.
*   **Concrete methods**: Methods inside an abstract class that contain standard implementation code. Subclasses inherit these automatically.

```typescript
abstract class Department {
  constructor(public name: string) {}

  printName(): void {
    console.log(`Department: ${this.name}`); // Concrete method
  }

  abstract getBudget(): number; // Abstract method (no body)
}

class Accounting extends Department {
  constructor() {
    super("Accounting");
  }

  // Must implement the abstract method
  getBudget(): number {
    return 50000;
  }
}

// const dept = new Department("General"); // Error: Cannot create an instance of an abstract class.
const accounting = new Accounting();
accounting.printName(); // "Department: Accounting"
console.log(accounting.getBudget()); // 50000
```

---

## 4. Private Constructors

Declaring a constructor as `private` prevents the class from being instantiated directly from outside its class definition.

### Primary Use Case 1: The Singleton Pattern
Ensures that a class has only one single instance globally, providing a global access point to it.

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection | null = null;

  // Private constructor prevents direct 'new DatabaseConnection()'
  private constructor() {
    console.log("Database connection established.");
  }

  // Static method acts as a control factory
  public static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }

  public query(sql: string) {
    return `Running: ${sql}`;
  }
}

// const db = new DatabaseConnection(); // Error: Constructor of class 'DatabaseConnection' is private and only accessible within the class declaration.

const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true (same instance shared)
```

### Primary Use Case 2: Static Utility Classes
If a class only contains static helper methods, making the constructor private prevents developer mistakes like instantiating it.

---

## Practical Checkpoint

Predict what happens if you inherit from a class that has a private constructor. Can you do it? Why or why not?

```typescript
class Base {
  private constructor() {}
}

class Derived extends Base {
  // What happens here?
}
```
