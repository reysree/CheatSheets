
# 📘 Functional Interfaces and Lambda Expressions in Java

## 🔹 What is a Functional Interface?

A **Functional Interface** is an interface that contains **only one abstract method**. These interfaces can be optionally annotated with `@FunctionalInterface`. 

Functional interfaces allow us to write **lambda expressions** — a concise way to implement the single method defined by the interface.

### ✅ Examples of Functional Interfaces in Java:

- `Runnable`
- `Comparable`
- `Comparator`

### 📦 Functional Interfaces from `java.util.function` package:

- `Function<T, R>`
- `BiFunction<T, U, R>`
- `Supplier<T>`
- `Consumer<T>`
- `Predicate<T>`
- `BiPredicate<T, U>`
- `UnaryOperator<T>`
- `BinaryOperator<T>`

---

## 🔄 Functional vs Normal Interfaces

| Feature | Functional Interface | Normal Interface |
|--------|----------------------|------------------|
| Abstract Methods | Only **one** | One or **more** |
| Static/Default Methods | ✅ Allowed | ✅ Allowed |
| Lambda Support | ✅ Yes | ❌ No (not directly) |

---

## 🔧 `Function<T, R>` Interface

A functional interface that takes one argument and returns a result. Input and output types can be different.

```java
@FunctionalInterface
interface Function<T, R> {
    R apply(T t);
}
```

**Usage:**
```java
Function<String, Integer> funLen = t -> t.length();
System.out.println(funLen.apply("Hello")); // Outputs: 5
```

---

## ✅ `Predicate<T>` Interface

Used for evaluating a condition — returns `true` or `false` based on input.

```java
@FunctionalInterface
interface Predicate<T> {
    boolean test(T t);
}
```

**Usage:**
```java
Predicate<Integer> pred = n -> n % 2 == 0;
System.out.println(pred.test(2)); // Outputs: true
```

---

## 📤 `Consumer<T>` Interface

Represents an operation that takes a single input and returns no result. Often used for printing or modifying state.

```java
@FunctionalInterface
interface Consumer<T> {
    void accept(T t);
}
```

**Usage:**
```java
Consumer<String> c = s -> System.out.println("My Name is " + s);
c.accept("Sreeram"); // Outputs: My Name is Sreeram
```

---

## 📥 `Supplier<T>` Interface

Supplies a result without taking any input.

```java
@FunctionalInterface
interface Supplier<T> {
    T get();
}
```

**Usage:**
```java
Supplier<ArrayList<Integer>> s = () -> new ArrayList<>(Arrays.asList(1, 2, 3));
ArrayList<Integer> alist = s.get();
System.out.println(alist); // Outputs: [1, 2, 3]
```

---

Happy Coding! 🚀
