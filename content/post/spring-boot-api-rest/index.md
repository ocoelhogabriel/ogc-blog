---
title: "Spring Boot do Zero: Criando sua Primeira API REST"
description: "Um guia prático para criar uma API REST com Spring Boot, validação de dados e tratamento de erros."
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

## O que vamos construir

Neste artigo vamos criar uma API REST completa com **Spring Boot 3**, incluindo:

- Endpoints CRUD para um recurso `Produto`
- Validação de dados com Bean Validation
- Tratamento global de erros com `@ControllerAdvice`
- Paginação de resultados

## Dependências

No `pom.xml`, adicione:

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

## Modelo de Domínio

```java
@Entity
@Table(name = "produtos")
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 3, max = 100)
    private String nome;

    @NotNull
    @Positive(message = "Preço deve ser positivo")
    private BigDecimal preco;

    // getters e setters omitidos
}
```

## Controller REST

```java
@RestController
@RequestMapping("/api/produtos")
@Validated
public class ProdutoController {

    private final ProdutoService service;

    public ProdutoController(ProdutoService service) {
        this.service = service;
    }

    @GetMapping
    public Page<ProdutoDTO> listar(Pageable pageable) {
        return service.listarTodos(pageable);
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProdutoDTO> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ProdutoDTO criar(@RequestBody @Valid ProdutoCriarDTO dto) {
        return service.criar(dto);
    }
}
```

## Tratamento de Erros Global

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> erros = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(err ->
            erros.put(err.getField(), err.getDefaultMessage())
        );
        return erros;
    }

    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Map<String, String> handleNotFound(EntityNotFoundException ex) {
        return Map.of("erro", ex.getMessage());
    }
}
```

## Testando com cURL

```bash
# Criar produto
curl -X POST http://localhost:8080/api/produtos \
  -H "Content-Type: application/json" \
  -d '{"nome": "Notebook", "preco": 4999.99}'

# Listar com paginação
curl "http://localhost:8080/api/produtos?page=0&size=10&sort=nome"
```

## Próximos Passos

- Adicionar autenticação com **Spring Security + JWT**
- Implementar cache com **Spring Cache + Redis**
- Escrever testes de integração com **@SpringBootTest**
