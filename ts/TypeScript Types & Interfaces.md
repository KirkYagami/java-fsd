# TypeScript Types & Interfaces - Complete  Notes

## Table of Contents
1. [Introduction to Types](#introduction-to-types)
2. [Basic Types](#basic-types)
3. [Type Annotations](#type-annotations)
4. [Type Inference](#type-inference)
5. [Union Types](#union-types)
6. [Type Aliases](#type-aliases)
7. [Interfaces](#interfaces)
8. [Type vs Interface](#type-vs-interface)
9. [Advanced Type Concepts](#advanced-type-concepts)
10. [Practice Exercises](#practice-exercises)

---

## Introduction to Types

TypeScript's type system adds static typing to JavaScript, helping catch errors during development rather than at runtime.

```typescript
// JavaScript (no type safety)
let value = "hello";
value = 42; // No error in JS

// TypeScript (with type safety)
let value: string = "hello";
value = 42; // ❌ Error: Type 'number' is not assignable to type 'string'
```

---

## Basic Types

### Primitive Types

```typescript
// String
let username: string = "Alice";
let message: string = `Hello, ${username}!`;

// Number
let age: number = 25;
let price: number = 19.99;
let hex: number = 0xf00d;
let binary: number = 0b1010;

// Boolean
let isActive: boolean = true;
let isComplete: boolean = false;

// Null and Undefined
let nullValue: null = null;
let undefinedValue: undefined = undefined;

// Symbol (ES6+)
let uniqueKey: symbol = Symbol("key");
```

### Special Types

```typescript
// Any - escape hatch from type checking
let flexible: any = 42;
flexible = "now it's a string";
flexible = true;

// Unknown - type-safe alternative to any
let userInput: unknown = "something";
// userInput.toUpperCase(); // ❌ Error: Object is of type 'unknown'

// Need to check type first
if (typeof userInput === "string") {
    console.log(userInput.toUpperCase()); // ✅ OK
}

// Void - absence of return value
function logMessage(message: string): void {
    console.log(message);
    // No return statement
}

// Never - function never returns
function throwError(message: string): never {
    throw new Error(message);
}

function infiniteLoop(): never {
    while (true) {}
}
```

---

## Type Annotations

Explicitly telling TypeScript what type a variable should be:

```typescript
// Variable annotations
let productName: string = "Laptop";
let quantity: number = 5;
let inStock: boolean = true;

// Function parameter annotations
function greetUser(name: string, age: number): string {
    return `Hello ${name}, you are ${age} years old`;
}

// Function return type annotation
function add(x: number, y: number): number {
    return x + y;
}

// Array annotations
let fruits: string[] = ["apple", "banana", "orange"];
let numbers: Array<number> = [1, 2, 3, 4, 5];
let mixed: (string | number)[] = ["hello", 42, "world"];

// Object annotations
let person: { name: string; age: number } = {
    name: "Bob",
    age: 30
};
```

---

## Type Inference

TypeScript can automatically infer types:

```typescript
// TypeScript infers this is string
let city = "New York"; // city: string

// TypeScript infers this is number
let count = 100; // count: number

// TypeScript infers this is boolean
let isDone = false; // isDone: boolean

// TypeScript infers this is string[]
let items = ["apple", "banana", "orange"]; // items: string[]

// TypeScript infers return type as number
function multiply(a: number, b: number) {
    return a * b;
}

// Best common type algorithm
let mixedArray = [1, "hello", true]; // (string | number | boolean)[]
```

---

## Union Types

Allow a value to be one of several types:

```typescript
// Basic union
let id: string | number;
id = "abc123"; // ✅ OK
id = 123456;   // ✅ OK
// id = true;  // ❌ Error

// Union with function parameters
function printId(id: string | number) {
    // Type narrowing needed
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    } else {
        console.log(id.toFixed(2));
    }
}

// Union with literal types
let direction: "north" | "south" | "east" | "west";
direction = "north"; // ✅ OK
direction = "northeast"; // ❌ Error

let httpStatus: 200 | 404 | 500;
httpStatus = 404; // ✅ OK
httpStatus = 403; // ❌ Error

// Union with arrays
let result: string | number[];
result = "success";
result = [1, 2, 3, 4];
```

---

## Type Aliases

Create custom names for types:

```typescript
// Simple type alias
type UserId = string | number;
let userId: UserId = "user123";

// Object type alias
type Person = {
    name: string;
    age: number;
    email?: string; // optional
};

let user: Person = {
    name: "Alice",
    age: 28
};

// Function type alias
type GreetingFunction = (name: string) => string;

const greet: GreetingFunction = (name) => {
    return `Hello, ${name}!`;
};

// Union type alias
type Status = "pending" | "processing" | "completed" | "failed";

let orderStatus: Status = "pending";
orderStatus = "completed"; // ✅ OK
// orderStatus = "shipped"; // ❌ Error

// Complex type alias
type Point = {
    x: number;
    y: number;
};

type Shape = 
    | { kind: "circle"; radius: number }
    | { kind: "rectangle"; width: number; height: number }
    | { kind: "triangle"; base: number; height: number };

// Using the complex type
function area(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "rectangle":
            return shape.width * shape.height;
        case "triangle":
            return (shape.base * shape.height) / 2;
    }
}
```

---

## Interfaces

### Basic Interface

```typescript
interface User {
    name: string;
    age: number;
    email: string;
}

function registerUser(user: User): void {
    console.log(`Registering ${user.name}...`);
}

let newUser: User = {
    name: "Charlie",
    age: 25,
    email: "charlie@example.com"
};

registerUser(newUser);
```

### Optional and Readonly Properties

```typescript
interface Configuration {
    readonly id: number;      // Cannot be changed after creation
    name: string;
    version?: string;         // Optional
    debugMode?: boolean;      // Optional
}

let config: Configuration = {
    id: 1,
    name: "MyApp"
};

config.id = 2; // ❌ Error: Cannot assign to readonly property
config.version = "1.0.0"; // ✅ OK (adding optional property)
```

### Methods in Interfaces

```typescript
interface Calculator {
    add(x: number, y: number): number;
    subtract(x: number, y: number): number;
    multiply: (x: number, y: number) => number; // Alternative syntax
    readonly pi: number;
}

const simpleCalc: Calculator = {
    pi: 3.14,
    add(x, y) {
        return x + y;
    },
    subtract(x, y) {
        return x - y;
    },
    multiply(x, y) {
        return x * y;
    }
};
```

### Extending Interfaces

```typescript
interface Animal {
    name: string;
    age: number;
    eat(): void;
}

interface Mammal extends Animal {
    hasFur: boolean;
    giveBirth(): void;
}

interface Dog extends Mammal {
    breed: string;
    bark(): void;
}

// Must implement all properties from all extended interfaces
let myDog: Dog = {
    name: "Rex",
    age: 3,
    hasFur: true,
    breed: "German Shepherd",
    eat() {
        console.log("Eating...");
    },
    giveBirth() {
        console.log("Giving birth...");
    },
    bark() {
        console.log("Woof!");
    }
};
```

### Multiple Interface Extension

```typescript
interface Flyable {
    fly(): void;
    maxAltitude: number;
}

interface Swimmable {
    swim(): void;
    maxDepth: number;
}

// Interface can extend multiple interfaces
interface Duck extends Flyable, Swimmable {
    name: string;
    quack(): void;
}

let donald: Duck = {
    name: "Donald",
    maxAltitude: 1000,
    maxDepth: 50,
    fly() {
        console.log("Flying...");
    },
    swim() {
        console.log("Swimming...");
    },
    quack() {
        console.log("Quack!");
    }
};
```

### Interface Declaration Merging

```typescript
// First declaration
interface User {
    name: string;
    email: string;
}

// Second declaration (same name) - MERGES with first
interface User {
    age: number;
    isActive?: boolean;
}

// Result: User has name, email, age, and optional isActive
let user: User = {
    name: "Alice",
    email: "alice@example.com",
    age: 28,
    isActive: true
};
```

### Index Signatures

```typescript
interface Dictionary {
    [key: string]: string;  // Any number of properties with string values
}

let colors: Dictionary = {
    red: "#ff0000",
    green: "#00ff00",
    blue: "#0000ff"
};

interface NumberDictionary {
    [key: string]: number;
    length: number;    // OK
    // name: string;   // ❌ Error: Property 'name' must be of type 'number'
}

interface UserMap {
    [id: number]: User;  // Index with numbers
    getById(id: number): User;
}
```

### Interface for Function Types

```typescript
interface StringProcessor {
    (input: string, times: number): string;
}

const repeat: StringProcessor = (text, count) => {
    return text.repeat(count);
};

console.log(repeat("Hello", 3)); // "HelloHelloHello"

interface MathOperation {
    (x: number, y: number): number;
    description: string;  // Can also have properties!
}

const add: MathOperation = (a, b) => a + b;
add.description = "Adds two numbers";
```

---

## Type vs Interface

### When to Use Interface

```typescript
// 1. When defining object shapes (especially for classes)
interface Vehicle {
    brand: string;
    start(): void;
}

class Car implements Vehicle {
    constructor(public brand: string) {}
    
    start() {
        console.log(`${this.brand} car started`);
    }
}

// 2. When you need declaration merging
interface Request {
    body: any;
}
interface Request {
    headers: Record<string, string>;
}
// Result: Request has both body AND headers

// 3. When extending from multiple sources
interface A { a: string; }
interface B { b: number; }
interface C extends A, B { c: boolean; }
```

### When to Use Type Alias

```typescript
// 1. For union types
type ID = string | number;
type Status = "pending" | "completed" | "failed";

// 2. For primitive types
type UserName = string;

// 3. For tuple types
type Point = [number, number];
let coordinates: Point = [10, 20];

// 4. For complex function types
type Callback = (error: Error | null, result?: any) => void;

// 5. When you need mapped types
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

// 6. For utility types
type Nullable<T> = T | null | undefined;
type UserWithoutEmail = Omit<User, "email">;
```

### Comparison Table

| Feature | Interface | Type Alias |
|---------|-----------|------------|
| Object shape | ✅ Yes | ✅ Yes |
| Union types | ❌ No | ✅ Yes |
| Primitive types | ❌ No | ✅ Yes |
| Declaration merging | ✅ Yes | ❌ No |
| Extend multiple | ✅ Yes (extends) | ✅ Yes (&) |
| Mapped types | ❌ No | ✅ Yes |
| Performance | Better for object types | Similar |
| Error messages | More readable | Can be complex |

---

## Advanced Type Concepts

### Intersection Types (&)

```typescript
interface Name {
    name: string;
}

interface Age {
    age: number;
}

interface Contact {
    email: string;
}

// Intersection type - must have ALL properties
type Person = Name & Age & Contact;

let person: Person = {
    name: "Alice",
    age: 30,
    email: "alice@example.com"
};
```

### Type Guards

```typescript
// typeof type guard
function processValue(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    return value.toFixed(2);
}

// instanceof type guard
class Dog {
    bark() { console.log("Woof!"); }
}

class Cat {
    meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
        animal.bark();
    } else {
        animal.meow();
    }
}

// in operator type guard
interface Car {
    drive(): void;
    honk(): void;
}

interface Boat {
    sail(): void;
}

function operate(vehicle: Car | Boat) {
    if ("drive" in vehicle) {
        vehicle.drive();
    } else {
        vehicle.sail();
    }
}

// User-defined type guard
interface Fish {
    swim(): void;
    name: string;
}

interface Bird {
    fly(): void;
    name: string;
}

function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
    if (isFish(pet)) {
        pet.swim();
    } else {
        pet.fly();
    }
}
```

### Generics with Interfaces

```typescript
interface Box<T> {
    value: T;
    isEmpty: boolean;
}

let stringBox: Box<string> = {
    value: "hello",
    isEmpty: false
};

let numberBox: Box<number> = {
    value: 42,
    isEmpty: false
};

interface Repository<T, K> {
    getById(id: K): T | undefined;
    save(entity: T): void;
    delete(id: K): void;
}

class UserRepository implements Repository<User, number> {
    private users: Map<number, User> = new Map();
    
    getById(id: number): User | undefined {
        return this.users.get(id);
    }
    
    save(user: User): void {
        this.users.set(user.id, user);
    }
    
    delete(id: number): void {
        this.users.delete(id);
    }
}
```

---

## Practice Exercises

### Exercise 1: Basic Types
Create variables with appropriate types for a product:
- name (string)
- price (number)
- inStock (boolean)
- tags (array of strings)
- dimensions (object with width, height, depth as numbers)

```typescript
// Your code here
let productName: string = "Laptop";
let productPrice: number = 999.99;
let productInStock: boolean = true;
let productTags: string[] = ["electronics", "computer"];
let productDimensions: { width: number; height: number; depth: number } = {
    width: 35.8,
    height: 2.5,
    depth: 24.9
};
```

### Exercise 2: Union Types
Create a function that accepts either a string or number and returns its length (for string) or its square (for number).

```typescript
// Solution
function processInput(input: string | number): number {
    if (typeof input === "string") {
        return input.length;
    } else {
        return input * input;
    }
}

console.log(processInput("hello")); // 5
console.log(processInput(5));       // 25
```

### Exercise 3: Interface Definition
Define an interface for a `Book` with properties:
- title (string)
- author (string)
- publishedYear (number)
- isbn (optional string)
- getSummary (method returning string)

```typescript
// Solution
interface Book {
    title: string;
    author: string;
    publishedYear: number;
    isbn?: string;
    getSummary(): string;
}

const myBook: Book = {
    title: "The TypeScript Guide",
    author: "John Doe",
    publishedYear: 2023,
    getSummary() {
        return `${this.title} by ${this.author} (${this.publishedYear})`;
    }
};

console.log(myBook.getSummary());
```

### Exercise 4: Extending Interfaces
Create a base interface `Vehicle` and extend it to create `Car` and `Motorcycle` interfaces.

```typescript
// Solution
interface Vehicle {
    brand: string;
    year: number;
    start(): void;
    stop(): void;
}

interface Car extends Vehicle {
    numberOfDoors: number;
    trunkCapacity: number;
}

interface Motorcycle extends Vehicle {
    hasSidecar: boolean;
    engineSize: number;
}

// Implementation
const myCar: Car = {
    brand: "Toyota",
    year: 2022,
    numberOfDoors: 4,
    trunkCapacity: 500,
    start() {
        console.log("Car started");
    },
    stop() {
        console.log("Car stopped");
    }
};
```

### Exercise 5: Type Aliases with Union
Create a type alias `Result` that can be either:
- `{ success: true; data: T }`
- `{ success: false; error: string }`

Then create a function that returns this type.

```typescript
// Solution
type Result<T> = 
    | { success: true; data: T }
    | { success: false; error: string };

function divide(a: number, b: number): Result<number> {
    if (b === 0) {
        return {
            success: false,
            error: "Division by zero"
        };
    }
    
    return {
        success: true,
        data: a / b
    };
}

const result1 = divide(10, 2);
const result2 = divide(10, 0);

if (result1.success) {
    console.log("Result:", result1.data);
} else {
    console.log("Error:", result1.error);
}
```

### Exercise 6: Interface with Methods
Create an interface `Counter` that:
- Has a `count` property (number)
- Has `increment()`, `decrement()`, and `reset()` methods
- Has a `getCount()` method that returns the current count

```typescript
// Solution
interface Counter {
    count: number;
    increment(): void;
    decrement(): void;
    reset(): void;
    getCount(): number;
}

class SimpleCounter implements Counter {
    constructor(public count: number = 0) {}
    
    increment(): void {
        this.count++;
    }
    
    decrement(): void {
        this.count--;
    }
    
    reset(): void {
        this.count = 0;
    }
    
    getCount(): number {
        return this.count;
    }
}

const counter = new SimpleCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
counter.decrement();
console.log(counter.getCount()); // 1
counter.reset();
console.log(counter.getCount()); // 0
```

### Exercise 7: Index Signatures
Create an interface `ShoppingCart` that uses an index signature to store items with their quantities.
Then add methods to add items, remove items, and calculate total.

```typescript
// Solution
interface ShoppingCart {
    [itemName: string]: number;
    addItem(item: string, quantity: number): void;
    removeItem(item: string): void;
    getTotal(): number;
}

class Cart implements ShoppingCart {
    [itemName: string]: any; // Index signature
    
    constructor() {
        return new Proxy(this, {
            get(target, prop) {
                if (prop in target) {
                    return target[prop as keyof Cart];
                }
                return target[prop as string] || 0;
            }
        });
    }
    
    addItem(item: string, quantity: number): void {
        this[item] = (this[item] || 0) + quantity;
    }
    
    removeItem(item: string): void {
        delete this[item];
    }
    
    getTotal(): number {
        let total = 0;
        for (let key in this) {
            if (typeof this[key] === 'number' && !['addItem', 'removeItem', 'getTotal'].includes(key)) {
                total += this[key];
            }
        }
        return total;
    }
}

const cart = new Cart();
cart.addItem("apple", 3);
cart.addItem("banana", 2);
console.log(cart.getTotal()); // 5
console.log(cart.apple); // 3
```

### Exercise 8: Generic Interface
Create a generic interface `Pair<T, U>` that holds two values of possibly different types.
Then implement a function that swaps the values.

```typescript
// Solution
interface Pair<T, U> {
    first: T;
    second: U;
}

function swapPair<T, U>(pair: Pair<T, U>): Pair<U, T> {
    return {
        first: pair.second,
        second: pair.first
    };
}

const numberStringPair: Pair<number, string> = {
    first: 42,
    second: "hello"
};

console.log(numberStringPair); // { first: 42, second: "hello" }

const swapped = swapPair(numberStringPair);
console.log(swapped); // { first: "hello", second: 42 }
```

### Exercise 9: Type vs Interface Decision
Create a scenario where you need both:
- A type alias for union types (e.g., `PaymentMethod`)
- An interface for an object that uses that type

```typescript
// Solution
// Type alias for union (Type is better here)
type PaymentMethod = "credit" | "debit" | "paypal" | "cash";

// Interface for object (Interface is better here)
interface Transaction {
    id: string;
    amount: number;
    method: PaymentMethod;
    date: Date;
    status: "pending" | "completed" | "failed";
    process(): Promise<boolean>;
}

class Payment implements Transaction {
    constructor(
        public id: string,
        public amount: number,
        public method: PaymentMethod,
        public date: Date = new Date(),
        public status: "pending" | "completed" | "failed" = "pending"
    ) {}
    
    async process(): Promise<boolean> {
        // Simulate payment processing
        this.status = "completed";
        return true;
    }
}

const payment = new Payment("txn_123", 99.99, "credit");
console.log(payment);
```

### Exercise 10: Complex Interface Design
Design interfaces for a simple blog system with:
- `Author` (id, name, email, bio)
- `Post` (id, title, content, author, publishedDate, tags)
- `Comment` (id, content, author, postId, createdAt)
- Include methods where appropriate

```typescript
// Solution
interface Author {
    id: string;
    name: string;
    email: string;
    bio?: string;
    getFullBio(): string;
}

interface Post {
    id: string;
    title: string;
    content: string;
    author: Author;
    publishedDate: Date;
    tags: string[];
    comments: Comment[];
    addComment(comment: Comment): void;
    getSummary(): string;
}

interface Comment {
    id: string;
    content: string;
    author: Author;
    postId: string;
    createdAt: Date;
    formattedDate(): string;
}

// Implementation
class BlogAuthor implements Author {
    constructor(
        public id: string,
        public name: string,
        public email: string,
        public bio?: string
    ) {}
    
    getFullBio(): string {
        return `${this.name} - ${this.email}${this.bio ? ': ' + this.bio : ''}`;
    }
}

class BlogPost implements Post {
    public comments: Comment[] = [];
    
    constructor(
        public id: string,
        public title: string,
        public content: string,
        public author: Author,
        public publishedDate: Date = new Date(),
        public tags: string[] = []
    ) {}
    
    addComment(comment: Comment): void {
        this.comments.push(comment);
    }
    
    getSummary(): string {
        return `${this.title} by ${this.author.name} (${this.comments.length} comments)`;
    }
}

// Usage
const author = new BlogAuthor("1", "Alice", "alice@blog.com", "Tech writer");
const post = new BlogPost(
    "post1",
    "TypeScript Tips",
    "Content here...",
    author,
    new Date(),
    ["typescript", "programming"]
);

console.log(post.getSummary());
```

---

## Key Takeaways

1. **Types** are about the shape and structure of data
2. **Interfaces** are best for object definitions and classes
3. **Type Aliases** are more flexible (unions, primitives, tuples)
4. Use **type guards** for narrowing union types
5. **Generics** add flexibility while maintaining type safety
6. **Interfaces support declaration merging**, types don't
7. Choose based on your specific use case and future needs

---

## Additional Resources

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Playground](https://www.typescriptlang.org/play)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)