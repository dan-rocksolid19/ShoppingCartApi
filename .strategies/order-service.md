# 🛒 Order Microservice – Implementation Guide

This document outlines the implementation of the `order-service`, a Spring Boot microservice responsible for handling order creation and retrieval in the shopping cart system.

## 📁 Project Structure

```
order-service/
├── src/
│   ├── main/
│   │   ├── java/com/org/orderservice/
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── model/
│   │   │   ├── dto/
│   │   │   ├── exception/
│   │   │   └── OrderServiceApplication.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── ...
│   └── test/
│       ├── java/
│       └── resources/
│           └── Order_Service.postman_collection.json
├── build.gradle
└── README.md
```

## 🎯 Responsibilities

- Create new orders with multiple items
- Retrieve order information by ID
- Validate order requests
- Handle error cases appropriately

## 🌐 Endpoints

| Method | Endpoint       | Description                   |
|--------|----------------|-------------------------------|
| POST   | `/orders`      | Create a new order            |
| GET    | `/orders/{id}` | Retrieve an order by its ID   |

## ⚙️ Configuration

### `application.yml`

```yaml
server:
  port: 8085

spring:
  datasource:
    url: jdbc:h2:mem:orderdb
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

### `Order.java`

```java
@Entity
@Table(name = "orders")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String customerId;
    private LocalDateTime createdAt;
    private BigDecimal totalAmount;
    
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "order")
    private List<OrderItem> items;
}
```

### `OrderItem.java`

```java
@Entity
@Table(name = "order_items")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private Long productId;
    private Integer quantity;
    private BigDecimal unitPrice;
    
    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
}
```

## 📤 DTOs

### Request DTOs

```java
@Data
public class CreateOrderRequest {
    @NotBlank(message = "Customer ID is required")
    private String customerId;

    @NotEmpty(message = "Order must contain at least one item")
    private List<OrderItemRequest> items;
}

@Data
public class OrderItemRequest {
    @NotNull(message = "Product ID is required")
    private Long productId;
    
    @NotNull(message = "Quantity is required")
    private Integer quantity;
}
```

### Response DTOs

```java
@Data
public class OrderResponse {
    private Long id;
    private String customerId;
    private LocalDateTime createdAt;
    private BigDecimal totalAmount;
    private List<OrderItemResponse> items;
}

@Data
public class OrderItemResponse {
    private Long productId;
    private Integer quantity;
    private BigDecimal unitPrice;
}
```

## 🎛️ Controller

### `OrderController.java`

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody @Valid CreateOrderRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(orderService.createOrder(request));
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.getOrderById(id));
    }
}
```

## ❌ Exception Handling

### `GlobalExceptionHandler.java`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleOrderNotFoundException(OrderNotFoundException ex) {
        ErrorResponse errorResponse = new ErrorResponse();
        errorResponse.setStatus(HttpStatus.NOT_FOUND.value());
        errorResponse.setError(HttpStatus.NOT_FOUND.getReasonPhrase());
        errorResponse.setMessage(ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        ErrorResponse errorResponse = new ErrorResponse();
        errorResponse.setStatus(HttpStatus.BAD_REQUEST.value());
        errorResponse.setError(HttpStatus.BAD_REQUEST.getReasonPhrase());
        errorResponse.setMessage("Validation failed");
        errorResponse.setDetails(errors);

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errorResponse);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneralExceptions(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse();
        errorResponse.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        errorResponse.setError(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
        errorResponse.setMessage("An unexpected error occurred");

        Map<String, String> details = new HashMap<>();
        details.put("exception", ex.getMessage());
        errorResponse.setDetails(details);

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(errorResponse);
    }
}
```

## 🧪 Testing

- Unit tests for `OrderService`
- Integration tests for `/orders` and `/orders/{id}`
- Use in-memory H2 database for order persistence
- Postman collection for manual API testing

### Postman Collection

A Postman collection is provided in `src/test/resources/Order_Service.postman_collection.json` for testing the order-service endpoints. The collection includes:

1. Happy path tests:
   - Create Order - Tests creating a new order with valid data
   - Get Order by ID - Tests retrieving an existing order

2. Error case tests:
   - Create Order - Missing Customer ID - Tests validation for missing customer ID
   - Create Order - Empty Items - Tests validation for empty items list
   - Create Order - Missing Product ID - Tests validation for missing product ID
   - Create Order - Missing Quantity - Tests validation for missing quantity
   - Get Order - Not Found - Tests error handling for non-existent order

## 📦 Packaging

- Packaged as **JAR**
- Runs on port `8085`

## 🚀 Run Locally

```bash
cd order-service
./gradlew bootRun
```

## ✅ Example Request

### Create Order

```bash
curl -X POST http://localhost:8085/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "customer123",
    "items": [
      {
        "productId": 1,
        "quantity": 2
      },
      {
        "productId": 2,
        "quantity": 1
      }
    ]
  }'
```

### Get Order

```bash
curl -X GET http://localhost:8085/orders/1
```