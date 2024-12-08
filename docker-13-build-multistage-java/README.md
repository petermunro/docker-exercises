# Spring Boot Docker Demo

This is a demo Spring Boot application used to demonstrate Docker multi-stage builds and optimization techniques.

## Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/example/demo/
│   │       ├── DemoApplication.java
│   │       ├── controller/
│   │       │   └── ProductController.java
│   │       ├── model/
│   │       │   └── Product.java
│   │       └── repository/
│   │           └── ProductRepository.java
│   └── resources/
│       ├── application.properties
│       └── data.sql
```

## Features

- REST API for product management
- H2 in-memory database
- Spring Boot Actuator for monitoring
- JPA for data persistence
- Lombok for reducing boilerplate code

## API Endpoints

- GET /api/products - List all products
- GET /api/products/{id} - Get a specific product
- POST /api/products - Create a new product
- DELETE /api/products/{id} - Delete a product

## Health and Monitoring

- /actuator/health - System health information
- /actuator/info - Application information
- /actuator/metrics - Application metrics
- /h2-console - Database console (dev only)

## Building and Running

### Maven

```bash
./mvnw clean package
java -jar target/docker-demo-1.0.0.jar
```

### Docker

Basic build:
```bash
docker build -t myapp:1.0 .
docker run -p 8080:8080 myapp:1.0
```

Multi-stage build:
```bash
docker build -f Dockerfile.multistage -t myapp:2.0 .
docker run -p 8080:8080 myapp:2.0
```

Development mode:
```bash
docker-compose up
```

## Testing the Application

```bash
# List all products
curl http://localhost:8080/api/products

# Get a specific product
curl http://localhost:8080/api/products/1

# Create a new product
curl -X POST http://localhost:8080/api/products \
     -H "Content-Type: application/json" \
     -d '{"name":"Tablet","description":"New tablet","price":299.99}'

# Delete a product
curl -X DELETE http://localhost:8080/api/products/1
``` 