# 📘 Spring Boot Pagination Guide

Pagination is a technique of dividing large datasets into smaller, manageable chunks (or pages), making it easier to retrieve and display data in web applications. This is especially useful when dealing with thousands of records.

---

## 🚀 Key Components in Spring Boot

### 1. `Pageable` 🧭
Represents a request for a **page of results**. It includes:
- **Page Number** (`page`) – Starts from `0` (first page)
- **Page Size** (`size`) – Number of records per page
- **Sorting** (optional) – Sorts records in ascending/descending order

#### ✅ Example:
```java
Pageable pageable = PageRequest.of(0, 10, Sort.by("column_name"));
```

---

### 2. `Page<T>` 📦
The result of a paginated query. It includes:
- `content` – The actual list of records
- `metadata` – Details like total pages, total elements, current page, etc.

#### ✅ Example:
```java
Page<User> usersPage = userRepository.findAll(pageable);
```

---

## 📂 Repository Layer
Use Spring Data JPA methods like this:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByCountry(String country, Pageable pageable);
}
```

---

## 🌐 Controller Layer
Let Spring handle pagination automatically with `Pageable`:

```java
@GetMapping("/users")
public Page<User> getUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

No need for `@RequestParam` or `@RequestBody`. Spring parses the query params automatically.

---

## 🧾 Pagination Metadata Returned

When returning a `Page<T>`, the JSON includes:
- `content`: list of users
- `totalElements`: total number of matching records
- `totalPages`: how many pages exist
- `number`: current page number
- `size`: number of records per page
- `hasNext()`, `hasPrevious()`

---

## 💻 Frontend Example (React)

```javascript
const getUsers = async (page, size, sortField = "name", sortOrder = "asc") => {
  const response = await fetch(
    `http://localhost:8080/users?page=${page}&size=${size}&sort=${sortField},${sortOrder}`
  );

  const data = await response.json();
  console.log(data);
};
```

➡️ This URL is automatically parsed by Spring Boot:
```
GET /users?page=1&size=10&sort=name,asc
```
➡️ Translates to:
```java
Pageable pageable = PageRequest.of(1, 10, Sort.by("name").ascending());
```

---

## 🎯 Why Use Pagination?

### ✅ Benefits:
- **Performance**: Loads only the current page, not the entire dataset
- **Scalability**: Handles large datasets more effectively
- **Better UI**: Enables smooth page-wise navigation, infinite scroll, etc.

---

## ⚙️ Default Behavior
If `page` or `size` is **not** provided in the URL:
- Default `page = 0`
- Default `size = 20`

You can customize these via `application.properties`:
```properties
spring.data.web.pageable.default-page-size=10
spring.data.web.pageable.max-page-size=50
```

---

## 📌 Summary
- Use `Pageable` as a method parameter
- Use `Page<T>` as return type for paginated + metadata-rich responses
- Spring automatically maps URL params like `?page=0&size=10&sort=name,asc`
- Ideal for any frontend that uses tables, search, or pagination controls

---

## 📚 Example: Complete Flow

### 🗂️ Repository
```java
Page<User> findByCountry(String country, Pageable pageable);
```

### 🌐 Controller
```java
@GetMapping("/users")
public Page<User> getUsers(@RequestParam String country, Pageable pageable) {
    return userRepository.findByCountry(country, pageable);
}
```

### 🌍 Frontend Request (React or Postman):
```
GET /users?country=India&page=0&size=5&sort=name,asc
```

---
