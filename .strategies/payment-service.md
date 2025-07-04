# 💳 Payment Microservice – Implementation Guide

This document outlines the implementation of the `payment-service`, a Spring Boot microservice responsible for simulating payment processing for orders in the shopping cart system.

## 📁 Project Structure

```
payment-service/
├── src/
│   ├── main/
│   │   ├── java/com/example/paymentservice/
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── model/
│   │   │   ├── dto/
│   │   │   ├── repository/
│   │   │   ├── config/
│   │   │   └── PaymentServiceApplication.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── ...
├── pom.xml
└── README.md
```

## 🎯 Responsibilities

- Simulate the payment process for an existing order
- Persist the payment record with a status (e.g., PAID, FAILED)
- Provide endpoint to retrieve payment status by order ID
- Accept simple payloads including order ID, amount, and payment method

## 🌐 Endpoints

| Method | Endpoint            | Description                    |
|--------|---------------------|--------------------------------|
| POST   | `/payments`         | Simulate a payment transaction |
| GET    | `/payments/{orderId}` | Retrieve payment status       |

## ⚙️ Configuration

### `application.yml`

```yaml
server:
  port: 8083

spring:
  datasource:
    url: jdbc:h2:mem:paymentsdb
    driverClassName: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

## 🧾 Models

### `Payment.java`

```java
@Entity
@Table(name = "payments")
@Data
public class Payment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String orderId;
    private BigDecimal amount;
    private String method;

    @Enumerated(EnumType.STRING)
    private PaymentStatus status;

    private LocalDateTime processedAt;
}
```

### `PaymentStatus.java`

```java
public enum PaymentStatus {
    PAID,
    FAILED
}
```

## 📤 DTOs

### `PaymentRequest.java`

```java
@Data
public class PaymentRequest {
    private String orderId;
    private BigDecimal amount;
    private String method; // e.g., CREDIT_CARD, PAYPAL, etc.
}
```

### `PaymentResponse.java`

```java
@Data
public class PaymentResponse {
    private String orderId;
    private BigDecimal amount;
    private String method;
    private String status;
    private LocalDateTime processedAt;
}
```

## 📂 Repository Layer

```java
public interface PaymentRepository extends JpaRepository<Payment, Long> {
    Optional<Payment> findByOrderId(String orderId);
}
```

## 🧠 Service Layer

### `PaymentService.java`

```java
@Service
public class PaymentService {

    private final PaymentRepository repository;

    public PaymentService(PaymentRepository repository) {
        this.repository = repository;
    }

    public PaymentResponse processPayment(PaymentRequest request) {
        Payment payment = new Payment();
        payment.setOrderId(request.getOrderId());
        payment.setAmount(request.getAmount());
        payment.setMethod(request.getMethod());
        payment.setProcessedAt(LocalDateTime.now());

        // Simulate success/failure randomly
        boolean success = ThreadLocalRandom.current().nextBoolean();
        payment.setStatus(success ? PaymentStatus.PAID : PaymentStatus.FAILED);

        Payment saved = repository.save(payment);

        return mapToResponse(saved);
    }

    public PaymentResponse getPaymentByOrderId(String orderId) {
        Payment payment = repository.findByOrderId(orderId)
                .orElseThrow(() -> new RuntimeException("Payment not found"));
        return mapToResponse(payment);
    }

    private PaymentResponse mapToResponse(Payment payment) {
        PaymentResponse res = new PaymentResponse();
        res.setOrderId(payment.getOrderId());
        res.setAmount(payment.getAmount());
        res.setMethod(payment.getMethod());
        res.setStatus(payment.getStatus().name());
        res.setProcessedAt(payment.getProcessedAt());
        return res;
    }
}
```

## 🎛️ Controller

### `PaymentController.java`

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {

    private final PaymentService service;

    public PaymentController(PaymentService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<PaymentResponse> processPayment(@RequestBody @Valid PaymentRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(service.processPayment(request));
    }

    @GetMapping("/{orderId}")
    public ResponseEntity<PaymentResponse> getPayment(@PathVariable String orderId) {
        return ResponseEntity.ok(service.getPaymentByOrderId(orderId));
    }
}
```

## 🧪 Testing

- Unit test `PaymentService` with mocked repository
- Integration test `PaymentController` with `@WebMvcTest`
- Use H2 in-memory DB for tests
- Postman collection for manual API testing

### Postman Collection

A Postman collection is provided in `src/test/resources/Payment_Service.postman_collection.json` for testing the payment-service endpoints. The collection includes:

1. Happy path tests:
   - Process Payment - Tests processing a new payment with valid data
   - Get Payment by Order ID - Tests retrieving an existing payment

2. Error case tests:
   - Process Payment - Missing Order ID - Tests validation for missing order ID
   - Process Payment - Missing Amount - Tests validation for missing amount
   - Process Payment - Missing Method - Tests validation for missing method
   - Get Payment - Not Found - Tests error handling for non-existent payment

## 📦 Packaging

- Packaged as **JAR**
- Spring Boot application with embedded Tomcat
- Runs on port `8083`

## 🧩 Dependencies (Maven)

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

## 🚀 Run Locally

```bash
cd payment-service
./mvnw spring-boot:run
```

## ✅ Example Request

```http
POST /payments
Content-Type: application/json

{
  "orderId": "abc123",
  "amount": 59.99,
  "method": "CREDIT_CARD"
}
```

```http
GET /payments/abc123
```
