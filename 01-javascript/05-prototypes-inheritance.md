# JavaScript OOP: Prototypes, Inheritance & Classes

This file covers the JavaScript prototype system, inheritance mechanisms, ES6 classes, private/static features, mixins, decorators, and the transition between class-based and prototype-based programming.

---

## 1. `__proto__` vs `prototype`

Understanding prototypes requires distinguishing between the **internal prototype link** of an instance and the **prototype property** of a constructor function.

*   **`[[Prototype]]`**: The internal link that every object has pointing to its fallback prototype.
*   **`__proto__`**: A legacy getter/setter accessor property on `Object.prototype` that exposes the internal `[[Prototype]]`. (Deprecated in favor of modern standard methods).
*   **`prototype`**: A property belonging **only to functions** (specifically constructor functions and ES6 classes, excluding arrow functions). It is the object that will be assigned as the `[[Prototype]]` to any new instances created when the function is invoked with the `new` operator.

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function() { return `Hi ${this.name}`; };

const alice = new Person("Alice");

// alice's internal prototype points to Person's prototype property
console.log(Object.getPrototypeOf(alice) === Person.prototype); // true
console.log(alice.__proto__ === Person.prototype);              // true (legacy syntax)
```

---

## 2. The Prototype Chain

When you access a property on an object, the JavaScript engine performs a search:
1.  It checks if the property exists directly on the object (own properties).
2.  If not found, it follows the internal `[[Prototype]]` link to the parent prototype.
3.  This lookup continues up the chain until the property is found, or it reaches `Object.prototype` (whose prototype is `null`). If still not found, it returns `undefined`.

```
[ alice ]  ───►  [ Person.prototype ]  ───►  [ Object.prototype ]  ───►  null
name: "Alice"    sayHi: function()           toString: function()
```

---

## 3. Creating Objects with Prototypes

Apart from constructors and classes, you can create objects linked to a specific prototype using `Object.create(proto, [propertiesObject])`.

```javascript
const animal = {
  eats: true,
  walk() { return "Animal walks"; }
};

// Create rabbit with animal as its prototype
const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats); // true (inherited)
console.log(rabbit.walk()); // "Animal walks" (inherited method)
```

---

## 4. Get and Set Object Prototypes

Instead of using the deprecated `__proto__` property, modern JavaScript provides static helper methods on the `Object` constructor:

*   **Get prototype**: `Object.getPrototypeOf(obj)`
*   **Set prototype**: `Object.setPrototypeOf(obj, newProto)`

```javascript
const user = { role: "guest" };
const admin = { permissions: ["read", "write"] };

Object.setPrototypeOf(user, admin); // user now inherits from admin
console.log(user.permissions); // ["read", "write"]
```

> [!CAUTION]
> Changing an object's prototype with `Object.setPrototypeOf` is a slow operation in all JavaScript engines. It affects engine optimizations (like Inline Caches) for code that accesses properties on the modified objects. Avoid changing prototypes on the fly; create objects with the correct prototype using `Object.create()` instead.

---

## 5. Prototypal vs Classical (OOP) Inheritance

| Feature | Prototypal Inheritance (JS) | Classical Inheritance (Java / C++) |
| :--- | :--- | :--- |
| **Concept** | Objects link directly to other objects | Classes act as blueprints; objects are instances |
| **Mechanism** | Dynamic delegation via prototype references | Static structure copied from class definition |
| **Memory** | Instances share methods from a single prototype | Methods are typically duplicated or resolved via VMTs |
| **Flexibility**| High: prototypes can be modified at runtime | Low: class hierarchies are fixed at compile time |

---

## 6. Functional Prototyping (Constructor Functions)

Before classes, inheritance was achieved by linking constructor functions together.

```javascript
// Parent Constructor
function Vehicle(wheels) {
  this.wheels = wheels;
}
Vehicle.prototype.drive = function() {
  return `Driving on ${this.wheels} wheels`;
};

// Child Constructor
function Car(make) {
  Vehicle.call(this, 4); // 1. Super call (bind parent properties to child instance)
  this.make = make;
}

// 2. Link child prototype to parent prototype
Car.prototype = Object.create(Vehicle.prototype);

// 3. Fix the constructor reference (otherwise Car.prototype.constructor resolves to Vehicle)
Car.prototype.constructor = Car;

Car.prototype.honk = function() {
  return "Beep!";
};

const myCar = new Car("Toyota");
console.log(myCar.drive()); // "Driving on 4 wheels" (inherited)
console.log(myCar.honk());  // "Beep!" (own prototype method)
```

---

## 7. Retrieving Object Constructor via Prototype

Every prototype object has a `.constructor` property pointing back to the constructor function that created it.

```javascript
console.log(myCar.constructor); // [Function: Car]

// If a constructor is lost during prototype reassignments (like Car.prototype = Object.create(Vehicle.prototype)),
// you must manually re-establish it:
Car.prototype.constructor = Car;
```

---

## 8. ES6 Class Basic Syntax

ES6 classes are syntactic sugar over the prototype mechanism, offering a cleaner syntax.

```javascript
class User {
  // Runs automatically upon 'new User()'
  constructor(name) {
    this.name = name;
  }
  
  // Method defined on User.prototype
  greet() {
    return `Hello, my name is ${this.name}`;
  }
}

const user = new User("John");
console.log(typeof User); // "function" (a class is still a function under the hood)
```

---

## 9. Class Inheritance (`extends` & `super`)

To inherit from another class, use `extends`. Within the child constructor, you must call `super()` before accessing `this`.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  makeSound() {
    return `${this.name} makes a noise.`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Invokes the parent constructor. Mandatory!
    this.breed = breed;
  }
  
  // Method overriding
  makeSound() {
    return `${super.makeSound()} Bark!`; // Call parent method via super
  }
}

const rex = new Dog("Rex", "German Shepherd");
console.log(rex.makeSound()); // "Rex makes a noise. Bark!"
```

---

## 10. Private & Protected Properties

JavaScript supports native private fields, while protected properties rely on conventions.

### A. Private Fields (`#`)
Prefixed with a `#`. They are completely inaccessible outside the class block (checked at compile time).
```javascript
class BankAccount {
  #balance = 0; // Private field
  
  constructor(amount) {
    this.#balance = amount;
  }
  
  getBalance() {
    return this.#balance; // Allowed inside the class
  }
}

const account = new BankAccount(100);
// console.log(account.#balance); // SyntaxError: Private field '#balance' must be declared in an enclosing class
console.log(account.getBalance()); // 100
```

### B. Protected Fields (`_` Convention)
Prefixed with a single underscore `_`. This is a convention signifying that the field should only be accessed within the class and its subclasses. It is **not** enforced by the engine.
```javascript
class Engine {
  _temp = 90; // Protected convention
}
```

---

## 11. `instanceof` Operator

Checks if the prototype property of a constructor appears anywhere in the prototype chain of an object.

```javascript
console.log(rex instanceof Dog);    // true (rex inherits from Dog.prototype)
console.log(rex instanceof Animal); // true (rex inherits from Animal.prototype)
console.log(rex instanceof Object); // true
```

### Customizing `instanceof` via `Symbol.hasInstance`:
```javascript
class StrangeArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}
console.log([] instanceof StrangeArray); // true
```

---

## 12. Static Properties & Methods

Declared with the `static` keyword. They belong to the class constructor itself, not to instances.

```javascript
class Config {
  static API_URL = "https://api.com"; // Static property
  
  static getUrl() {
    return this.API_URL; // 'this' inside static methods points to the Class constructor itself
  }
}

console.log(Config.API_URL); // "https://api.com"
// const conf = new Config();
// console.log(conf.API_URL); // undefined (not on the instance)
```

---

## 13. Reverting Classes to Functional Prototypes (and Vice Versa)

To understand class mechanics, it helps to see the equivalence between ES6 class syntax and ES5 functional prototypes.

### ES6 Class Code:
```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hi ${this.name}`;
  }
  static info() {
    return "Human";
  }
}
```

### Equivalent ES5 Prototype Code:
```javascript
function Person(name) {
  // Class constructor checks if it is invoked with new
  if (!(this instanceof Person)) {
    throw new TypeError("Cannot call a class as a function");
  }
  this.name = name;
}

// Instance method
Person.prototype.greet = function() {
  return "Hi " + this.name;
};

// Static method
Person.info = function() {
  return "Human";
};
```

---

## 14. Mixins

A mixin is a design pattern where an object or class inherits properties and methods from multiple sources. It allows combining behaviors without creating complex deep inheritance hierarchies.

```javascript
// Mixin containing helper methods
const flyMixin = {
  fly() {
    return `${this.name} takes off into the sky!`;
  },
  land() {
    return `${this.name} lands safely.`;
  }
};

class Bird {
  constructor(name) {
    this.name = name;
  }
}

// Copy mixin methods to Bird prototype
Object.assign(Bird.prototype, flyMixin);

const eagle = new Bird("Eagle");
console.log(eagle.fly()); // "Eagle takes off into the sky!"
```

---

## 15. Decorators

Decorators are an advanced JavaScript proposal (currently Stage 3) that allows modifying class declarations, methods, accessors, properties, and parameters using a simple annotation syntax (`@decoratorName`).

*   *Note*: To run decorators, you currently need a compiler/transpiler like TypeScript or Babel configured with experimental/modern proposal flags.

### Conceptual Syntax:
```typescript
function readonly(value, { kind, name }) {
  if (kind === "field") {
    return function (initialValue) {
      // Custom modification logic
      return initialValue;
    };
  }
}

class User {
  @readonly
  id = "123-ABC"; // field cannot be modified at runtime
}
```

---

## Practical Checkpoint

What does the following snippet print? Walk through the prototype chain lookups:

```javascript
function Foo() {}
Foo.prototype.x = 10;

const obj = new Foo();
Foo.prototype = { x: 20 };

const obj2 = new Foo();

console.log(obj.x);
console.log(obj2.x);
```
