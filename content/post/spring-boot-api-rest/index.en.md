---
title: "Spring Boot from Scratch: Creating Your First REST API"
description: "A practical guide to creating a REST API with Spring Boot, data validation, and error handling."
date: 2026-02-19
slug: spring-boot-api-rest
categories:
  - Spring Framework
tags:
  - java
  - spring-boot
  - api-rest
  - tutorial
image: cover.jpg
draft: false
---

## What we will build

In this article, we will create a complete REST API with **Spring Boot 3**, including:

- CRUD endpoints for a `Product` resource
- Data validation with Bean Validation
- Global error handling with `@ControllerAdvice`
- Result pagination

## Dependencies

In `pom.xml`, add:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

## Domain Model

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Name is mandatory")
    @Size(min = 3, max = 100)
    private String name;

    @NotNull
    @Positive(message = "Price must be positive")
    private BigDecimal price;

    // getters and setters omitted
}
```

## REST Controller

```java
@RestController
@RequestMapping("/api/products")
@Validated
public class ProductController {

    private final ProductService service;

    public ProductController(ProductService service) {
        this.service = service;
    }

    @GetMapping
    public Page<ProductDTO> list(Pageable pageable) {
        return service.findAll(pageable);
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> find(@PathVariable Long id) {
        return ResponseEntity.ok(service.findById(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ProductDTO create(@RequestBody @Valid ProductCreateDTO dto) {
        return service.create(dto);
    }
}
```

## Global Error Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(err ->
            errors.put(err.getField(), err.getDefaultMessage())
        );
        return errors;
    }

    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Map<String, String> handleNotFound(EntityNotFoundException ex) {
        return Map.of("error", ex.getMessage());
    }
}
```

## Testing with cURL

```bash
# Create product
curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Notebook", "price": 4999.99}'

# List with pagination
curl "http://localhost:8080/api/products?page=0&size=10&sort=name"
```

## Next Steps

- Add authentication with **Spring Security + JWT**
- Implement cache with **Spring Cache + Redis**
- Write integration tests with **@SpringBootTest**
