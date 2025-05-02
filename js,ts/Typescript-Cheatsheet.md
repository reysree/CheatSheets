# 📘 TypeScript Cheatsheet (Beginner to Advanced)

---

## 🧱 TypeScript Basics

### 🔧 TypeScript Needs a Compiler
- TypeScript always needs to be compiled to JavaScript using `tsc`. It cannot run natively in the browser or Node.js without being converted to JavaScript first.

### 🔢 Primitive Data Types
- `number` — used for integers, floats, and doubles. Covers all numerical data.
- `string` — used to assign a string or a stream of characters (e.g., names, messages).
- `boolean` — used when a value can be either `true` or `false`. Great for conditions.

---

## 💡 Variable Declarations

### 🔀 `var`, `let`, and `const`
- `var` — Does not have block scope. It’s an older way of declaring variables and can cause bugs due to scope leakage.
- `let` — Has block scope, which means the variable is only accessible within the block it’s declared in.
- `const` — Also block scoped, but the value assigned must be initialized at declaration and cannot be reassigned. Use it for values that shouldn't change.

---

## 🧩 Custom Types

### 🧑‍🎨 Defining Custom Types
- Custom types are defined using the `type` keyword. They are used to describe the shape of an object and can be reused across the codebase.
```ts
type Person = {
  name: string,
  age: number,
  isStudent: boolean
};

let person1: Person = {
  name: "Sreeram",
  age: 10,
  isStudent: false
};
```

### ❓ Optional Properties
- Optional properties are declared using a `?`. This tells TypeScript that this field is not mandatory.
```ts
type Address = {
  street: string;
  city: string;
  country: string;
};

type Person = {
  name: string;
  age: number;
  isStudent: boolean;
  address?: Address;
};
```
- If `address` is not provided when creating a `Person`, it won’t throw an error.
```ts
let person: Person = {
  name: "Sreeram",
  age: 25,
  isStudent: false
};
```
- When accessing optional fields, use the `?.` operator to safely access nested properties:
```ts
function displayPerson(person: Person) {
  console.log(`${person.address?.street}`);
}
```

---

## 📚 Arrays in TypeScript
- Arrays store multiple values of the same type.
```ts
let numbers: number[] = [1, 2, 3, 4];
let names: string[] = ["Sree", "Ram"];
```

- Array of objects:
```ts
let people: Person[] = [person1];
```

---

## 🔀 Union Types
- A union allows a variable to hold more than one type.
```ts
type User = "admin" | "manager" | "master";
let currentUser: User = "admin";
```
- It works like enums in other languages, restricting values to a specific list.

---

## 🔎 Type Narrowing
- Type narrowing means starting with a variable of a broad type (like `string | number`) and using logic to narrow it down to one type.
- Use `typeof`, `in`, or `instanceof` to narrow down the type.
```ts
function getPizzaDetail(identifier: string | number) {
  if (typeof identifier === "string") {
    console.log("Perform task using the string value");
  } else if (typeof identifier === "number") {
    console.log("Perform task using the number value");
  } else {
    console.error("Throw error");
  }
}
```

---

## 🔁 Function Return Types
- TypeScript can infer the return type, but it’s good practice to specify it for clarity and safety.
```ts
type User = {
  id: number;
  name: string;
};

function fetchUserDetails(username: string): User {
  return user;
}
```
- If you try to return something not matching the return type, TypeScript will show an error.

---

## 🛑 `any` Type
- `any` disables type checking. Use it cautiously.
- Best used when migrating from JavaScript to TypeScript.

---

## 🚫 `void` Type
- Used when a function does not return anything.
- Common in logging or functions that only perform actions.
```ts
function logMessage(): void {
  console.log("Hello");
}
```

---

## 🔢 Auto-Incrementing IDs
- Use a global variable and increment it each time a new object is added.
```ts
let pizzaId: number = 1;

type Pizza = {
  id: number;
  name: string;
  price: number;
};

function addPizza(name: string, price: number) {
  pizzas.push({ id: pizzaId++, name, price });
}
```

---

## 🧰 Utility Types
- Utility types simplify and reduce boilerplate when working with complex types.
- Examples:
  - `Partial<T>`: Makes all properties optional.
  - `Required<T>`: Makes all properties required.
  - `Readonly<T>`: Makes all properties immutable.
  - `Omit<T, K>`: Removes one or more properties from a type.
  - `Pick<T, K>`: Picks only specified properties from a type.

### 🔧 `Partial<T>`
- Allows updating only some properties of a type without requiring the full structure.
```ts
type User = {
  id: number;
  address: string;
  isPresent: boolean;
};

type UpdatedUser = Partial<User>;

function updateUser(id: number, updates: UpdatedUser) {
  const foundUser = users.find((user) => user.id === id);
  if (!foundUser) return;
  Object.assign(foundUser, updates);
}
```

### ✂️ `Omit<T, K>`
- Creates a new type excluding one or more fields.
```ts
type ToDo = {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
};

type ToDo1 = Omit<ToDo, "description">;
type ToDo2 = Omit<ToDo, "description" | "completed">;
```

---

## 🧠 Generics
- Generics let you write functions and types that work with any data type.
```ts
function identity<T>(value: T): T {
  return value;
}

identity<string>("Hello");
identity<number>(42);
```
- `<T>` is a placeholder that gets replaced with the actual type when the function is called.

---

## ✅ Concepts to Cover Next
- Mapped Types
- More Utility Types: `Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`, `InstanceType`
- `keyof`, `typeof`, and Indexed Access Types
- Conditional Types
- Discriminated Unions
- `infer` Keyword and Advanced Generics

---

## 📌 Extra Notes

### 📤 `export` / `import`
- Use `export` on a function/type to use it in other files.
```ts
// file1.ts
export function greet() {}

// file2.ts
import { greet } from './file1';
```

### 🧪 `Object.assign()`
- Copies enumerable properties from source objects to a target object.
```ts
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };
const result = Object.assign(target, source); // { a: 1, b: 4, c: 5 }
```

### 🌬️ Spread Operator
- Spread operator (`...`) spreads contents of an array or object.
- Use when you want to:
  - Copy
  - Merge
  - Extend
  - Override
```ts
const base = [1, 2, 3];
const newArray = [...base]; // [1, 2, 3]
```
```ts
const baseProps = { disabled: false, type: "button" };
const newButtonProps = { ...baseProps, className: "primary" };
```
