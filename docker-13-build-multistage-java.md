## Multi-Stage Builds with Spring Boot

In this lab, you'll create a Spring Boot application and optimize its Docker image using multi-stage builds. You'll see how to dramatically reduce image size while maintaining all functionality.

### Prerequisites
- Docker installed and running on your machine
- Basic familiarity with Spring Boot
- Git installed (to clone the starter project)

### Learning Objectives
- Create an optimized Docker image for a Spring Boot application
- Understand Maven dependency caching in Docker
- Implement multi-stage builds for Java applications
- Configure Spring Boot for containerization
- Optimize for production deployment

### Estimated Time
45 minutes

### The Lab

#### Step 1: Get the Spring Boot Application
1. Clone the starter project:
   ```bash
   git clone https://github.com/your-org/spring-docker-demo
   cd spring-docker-demo
   ```

   Or create a new Spring Boot project with:
   - Spring Web
   - Spring Actuator
   - Spring Data JPA
   - H2 Database
   - Lombok

2. The project should have this basic structure:
   ```
   src/
   ├── main/
   │   ├── java/
   │   │   └── com/example/demo/
   │   │       ├── DemoApplication.java
   │   │       ├── controller/
   │   │       │   └── ProductController.java
   │   │       └── model/
   │   │           └── Product.java
   │   └── resources/
   │       └── application.properties
   pom.xml
   ```

#### Step 2: Create Initial Dockerfile
1. Create a basic Dockerfile:
   ```dockerfile
   FROM openjdk:17-jdk
   WORKDIR /app
   COPY target/*.jar app.jar
   EXPOSE 8080
   CMD ["java", "-jar", "app.jar"]
   ```

2. Build and test:
   ```bash
   ./mvnw package
   docker build -t myapp:1.0 .
   docker images | grep myapp
   ```

❓ **Question**: What's the size of this image? Why is it so large?

#### Step 3: Implement Multi-Stage Build
1. Create an optimized Dockerfile:
   ```dockerfile
   # Build stage
   FROM maven:3.8.4-openjdk-17 AS builder
   WORKDIR /app
   COPY pom.xml .
   RUN mvn dependency:go-offline
   
   COPY src ./src
   RUN mvn package -DskipTests
   
   # Runtime stage
   FROM openjdk:17-jdk-slim
   WORKDIR /app
   COPY --from=builder /app/target/*.jar app.jar
   
   EXPOSE 8080
   CMD ["java", "-jar", "app.jar"]
   ```

2. Build and compare:
   ```bash
   docker build -t myapp:2.0 .
   docker images | grep myapp
   ```

❓ **Question**: What makes this build more efficient? Notice the dependency caching strategy.

#### Step 4: Further Optimization
1. Create a production-optimized Dockerfile:
   ```dockerfile
   # Build stage
   FROM maven:3.8.4-openjdk-17 AS builder
   WORKDIR /app
   COPY pom.xml .
   RUN mvn dependency:go-offline
   
   COPY src ./src
   RUN mvn package -DskipTests
   
   # Runtime stage
   FROM eclipse-temurin:17-jre-alpine
   WORKDIR /app
   
   # Create non-root user
   RUN addgroup -S spring && adduser -S spring -G spring
   USER spring:spring
   
   COPY --from=builder /app/target/*.jar app.jar
   
   EXPOSE 8080
   ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
   ```

2. Build and test:
   ```bash
   docker build -t myapp:3.0 .
   docker images | grep myapp
   ```

❓ **Question**: What security and performance improvements were added in this version?

#### Step 5: Development Configuration
1. Create a development-focused Dockerfile:
   ```dockerfile
   # Dev stage
   FROM maven:3.8.4-openjdk-17
   WORKDIR /app
   
   # Keep maven dependencies cached in layer
   COPY pom.xml .
   RUN mvn dependency:go-offline
   
   # For dev, we'll mount the src directory
   CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=dev"]
   ```

2. Create a docker-compose.yml for development:
   ```yaml
   version: '3.8'
   services:
     app:
       build:
         context: .
         dockerfile: Dockerfile.dev
       ports:
         - "8080:8080"
       volumes:
         - ./src:/app/src
       environment:
         - SPRING_PROFILES_ACTIVE=dev
   ```

### Challenge
1. Add Spring Boot DevTools for development
2. Implement health checks
3. Add multi-stage build arguments:
   - Java version
   - Maven version
   - Spring profile
4. Optimize JVM settings for containers
5. Implement layered JAR approach

### Key Takeaways
- Maven dependency caching improves build time
- Multi-stage builds reduce final image size
- Different base images for different needs
- Security considerations in production images
- Development vs production configurations

### Clean Up
```bash
docker rmi myapp:1.0 myapp:2.0 myapp:3.0
```

### Extra Notes
- Consider using Spring Boot's layered JAR feature
- Spring profiles help with environment-specific config
- JVM container awareness is crucial
- Security scanning is important for production images
- Development workflow needs fast feedback loop 