# ⚡ Circuit Breaker Design Pattern

The **Circuit Breaker** design pattern is a powerful tool used in distributed systems to **prevent cascading failures** and maintain system stability.

---

## 🧠 What Is the Circuit Breaker Pattern?

The Circuit Breaker pattern is a **resiliency design pattern** that helps systems handle failures gracefully, especially in distributed architectures where one service might depend on others. It acts like an electrical circuit breaker:

- By **default**, the circuit is in a **closed state**, allowing requests to flow normally.
- When a service becomes unreliable—due to timeouts, exceptions, or high latency—and the number of failures exceeds a configured **threshold**, the circuit trips to the **open state**.
- In the **open state**, requests are immediately failed or redirected to a **fallback mechanism**, preventing the system from wasting resources on likely-failed operations.
- After a specified **cooldown period**, the circuit transitions to a **half-open state**, where it allows a limited number of test requests to check if the dependent service has recovered.
- If the test calls are successful, the circuit **closes** again and resumes normal operation. If not, it reverts to the open state.

This pattern helps maintain the overall **responsiveness and stability** of an application, especially under failure conditions.

---

## 🚦 Circuit Breaker States

1. **🔴 Open State (Tripped):**   When the failure rate crosses a defined threshold, all future calls are short-circuited.

2. **🟡 Half-Open State (Test Period):**   After a cooldown, a few trial calls are made. If they succeed, the circuit is restored.

3. **🟢 Closed State (Recovered):**   Normal operations resume once the trial calls are successful.

---

## 💪 Where Do We Use It?

- In **microservice architectures** where services depend on each other.
- For **third-party API calls**, especially when they become unresponsive.

---

## 💡 Why Use This Pattern?

- Limits resource usage (threads, connections) for failing services.
- Prevents full system crash due to a single service's failure.
- Enables fallback (e.g., cached data or default response) when the service is down.

---

## 🔁 Example Scenario

Imagine a microservices flow like:`user_service` → `catalog_service` → `discount_service`

If we define a failure threshold of 50% for `catalog_service`, and over 50% of the calls fail, the circuit will **open**. Further calls are blocked, and the app may use cached or default responses to keep running.

---

## 🚀 Circuit Breaker Implementation in Spring Boot

We can implement this pattern using the **Resilience4j** library.

### 🧰 Required Dependencies

- `resilience4j-spring-boot3`
- `spring-boot-starter-actuator` (for health checks)

### 🛁 Health Monitoring Setup

In `application.yml`:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  health:
    circuitbreakers:
      enabled: true
```

Now access health info at:📅 `http://localhost:8080/actuator/health/myCircuitBreaker`

### ⚙️ Sample Configuration in `application.yml`

```yaml
resilience4j:
  circuitbreaker:
    instances:
      myCircuitBreaker:
        registerHealthIndicator: true
        slidingWindowSize: 5
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 2
```

#### 🔍 Explanation:

- `instances`: Define different circuit breakers per service.
- `registerHealthIndicator`: Enables monitoring of this instance.
- `slidingWindowSize`: Number of recent calls tracked to calculate failure rate.
- `minimumNumberOfCalls`: Circuit breaker won't evaluate until this number of calls has occurred.
- `failureRateThreshold`: If failed calls exceed this %, the circuit opens.
- `waitDurationInOpenState`: Time to wait before moving to Half-Open.
- `permittedNumberOfCallsInHalfOpenState`: Number of trial calls to decide if service has recovered.

---

## ✍️ Example Usage in Code

Consider `user_service` → `catalog_service`. In the REST call from `user_service`, we use the annotation:

```java
@CircuitBreaker(name = "myCircuitBreaker", fallbackMethod = "defaultResponse")
public String getCatalogData() {
    // REST call logic here
}
```

Where:

- `name` refers to the circuit breaker instance from `application.yml`
- `fallbackMethod` is called when the circuit is **OPEN**

---

Enjoy building resilient microservices! 🔧🛡️