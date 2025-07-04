# 🧾 Product Microservice – Implementation Guide

This document describes the implementation of the `product-service`, a Spring Boot microservice responsible for acting as a proxy to the external Fake Store API (`https://fakestoreapi.com`), providing product data to other services in the shopping cart system.

## 📦 Project Structure

```
product-service/
├── src/
│   ├── main/
│   │   ├── java/com/example/productservice/
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── client/
│   │   │   ├── dto/
│   │   │   ├── config/
│   │   │   └── ProductServiceApplication.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── ...
├── pom.xml
└── README.md
```

## 🎯 Purpose

- Proxy requests to `https://fakestoreapi.com`
- Provide simplified and controlled access to product data
- Optionally, add caching or rate-limiting

## 🌐 Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products` | Get all products |
| GET | `/products/{id}` | Get product by ID |
| GET | `/products/categories` | Get list of product categories |
| GET | `/products/category/{name}` | Get products by category |

## ⚙️ Configuration

### `application.yml`

```yaml
server:
  port: 8081

external:
  fakestore:
    base-url: https://fakestoreapi.com
```

## 🔗 External API Client

### `FakeStoreClient.java`

```java
@FeignClient(name = "fakeStoreClient", url = "${external.fakestore.base-url}")
public interface FakeStoreClient {
    @GetMapping("/products")
    List<ProductDTO> getAllProducts();

    @GetMapping("/products/{id}")
    ProductDTO getProductById(@PathVariable("id") Long id);

    @GetMapping("/products/categories")
    List<String> getCategories();

    @GetMapping("/products/category/{category}")
    List<ProductDTO> getProductsByCategory(@PathVariable("category") String category);
}
```

> Requires `spring-cloud-starter-openfeign`

## 🧠 Service Layer

### `ProductService.java`

```java
@Service
public class ProductService {
    private final FakeStoreClient client;

    public ProductService(FakeStoreClient client) {
        this.client = client;
    }

    public List<ProductDTO> getAllProducts() {
        return client.getAllProducts();
    }

    public ProductDTO getProductById(Long id) {
        return client.getProductById(id);
    }

    public List<String> getCategories() {
        return client.getCategories();
    }

    public List<ProductDTO> getProductsByCategory(String category) {
        return client.getProductsByCategory(category);
    }
}
```

## 🎛️ Controller

### `ProductController.java`

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService service;

    public ProductController(ProductService service) {
        this.service = service;
    }

    @GetMapping
    public ResponseEntity<List<ProductDTO>> getAllProducts() {
        return ResponseEntity.ok(service.getAllProducts());
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> getProductById(@PathVariable Long id) {
        return ResponseEntity.ok(service.getProductById(id));
    }

    @GetMapping("/categories")
    public ResponseEntity<List<String>> getCategories() {
        return ResponseEntity.ok(service.getCategories());
    }

    @GetMapping("/category/{category}")
    public ResponseEntity<List<ProductDTO>> getByCategory(@PathVariable String category) {
        return ResponseEntity.ok(service.getProductsByCategory(category));
    }
}
```

## 🧾 DTO Example

### `ProductDTO.java`

```java
@Data
public class ProductDTO {
    private Long id;
    private String title;
    private Double price;
    private String description;
    private String category;
    private String image;
}
```

## 🧪 Testing

- Unit tests for `ProductService` using `@MockBean FakeStoreClient`
- Integration tests for `ProductController` with `@WebMvcTest`
- Test stubs or mock the `fakestoreapi.com` responses using `MockWebServer` (optional)
- Postman collection for manual API testing

### Postman Collection

A Postman collection is provided in `src/test/resources/Product_Service.postman_collection.json` for testing the product-service endpoints. The collection includes:

1. Happy path tests:
   - Get All Products - Tests retrieving all products
   - Get Product by ID - Tests retrieving a product by ID
   - Get All Categories - Tests retrieving all categories
   - Get Products by Category - Tests retrieving products by category

2. Error case tests:
   - Get Product - Not Found - Tests error handling for non-existent product
   - Get Products by Category - Not Found - Tests error handling for non-existent category

## 🐳 Optional Enhancements

- Add caching with `@Cacheable`
- Retry logic using `@Retryable`
- Circuit Breaker with Resilience4j
- Error handling with `@ControllerAdvice`

## 📦 Packaging

- Packaged as a **JAR**
- Built with Maven or Gradle
- Runs on port `8081` by default

## 🧩 Dependencies (Maven)

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
</dependencies>
```

## 🚀 Run Locally

```bash
cd product-service
./mvnw spring-boot:run
```

## ✅ Example Request

```http
GET http://localhost:8081/products/1
