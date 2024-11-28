## Multi-Stage Builds and Image Optimization

In this lab, you'll learn how to create efficient Docker images using multi-stage builds and various optimization techniques. You'll see how to dramatically reduce image sizes while maintaining functionality.

### Prerequisites
- Completed the previous Dockerfile labs
- Docker installed and running on your machine
- Basic familiarity with a compiled language (Go will be used in examples)

### Learning Objectives
- Understand and implement multi-stage builds
- Reduce Docker image sizes
- Learn best practices for layer optimization
- Create production-ready images

### Estimated Time
45 minutes

### The Lab

#### Step 1: Understanding the Problem
1. Create a new directory and navigate to it:
   ```bash
   mkdir multistage-lab
   cd multistage-lab
   ```

2. Create a simple Go application (`main.go`):
   ```go
   package main
   
   import (
       "fmt"
       "net/http"
   )
   
   func main() {
       http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
           fmt.Fprintf(w, "Hello from our optimized container!")
       })
       
       fmt.Println("Server starting on port 8080...")
       http.ListenAndServe(":8080", nil)
   }
   ```

3. Create a basic Dockerfile (`Dockerfile.basic`):
   ```dockerfile
   FROM golang:1.21
   
   WORKDIR /app
   COPY main.go .
   
   RUN go build -o server .
   
   EXPOSE 8080
   CMD ["./server"]
   ```

4. Build and check the size:
   ```bash
   docker build -t app-basic -f Dockerfile.basic .
   docker images app-basic
   ```

❓ **Question**: Why is this image so large? What components are included that we don't need at runtime?

#### Step 2: Creating a Multi-Stage Build
1. Create an optimized Dockerfile (`Dockerfile.multistage`):
   ```dockerfile
   # Build stage
   FROM golang:1.21 AS builder
   
   WORKDIR /app
   COPY main.go .
   
   RUN CGO_ENABLED=0 GOOS=linux go build -o server .
   
   # Runtime stage
   FROM alpine:3.18
   
   WORKDIR /app
   COPY --from=builder /app/server .
   
   EXPOSE 8080
   CMD ["./server"]
   ```

2. Build and compare sizes:
   ```bash
   docker build -t app-multistage -f Dockerfile.multistage .
   docker images | grep app-
   ```

❓ **Question**: What makes this image significantly smaller? What role does Alpine Linux play?

#### Step 3: Further Optimization with Scratch
1. Create an even more optimized Dockerfile (`Dockerfile.scratch`):
   ```dockerfile
   # Build stage
   FROM golang:1.21 AS builder
   
   WORKDIR /app
   COPY main.go .
   
   RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o server .
   
   # Runtime stage
   FROM scratch
   
   WORKDIR /
   COPY --from=builder /app/server .
   
   EXPOSE 8080
   CMD ["./server"]
   ```

2. Build and compare all versions:
   ```bash
   docker build -t app-scratch -f Dockerfile.scratch .
   docker images | grep app-
   ```

❓ **Question**: What are the trade-offs between using `scratch` vs `alpine` as a base image?

#### Step 4: Layer Optimization
1. Create a Node.js application to demonstrate layer optimization:
   ```bash
   npm init -y
   npm install express
   ```

2. Create `server.js`:
   ```javascript
   const express = require('express');
   const app = express();
   
   app.get('/', (req, res) => {
       res.send('Hello from Node.js!');
   });
   
   app.listen(8080, () => {
       console.log('Server running on port 8080');
   });
   ```

3. Create an unoptimized Dockerfile (`Dockerfile.node.unopt`):
   ```dockerfile
   FROM node:18
   
   WORKDIR /app
   COPY . .
   
   RUN npm install
   
   EXPOSE 8080
   CMD ["node", "server.js"]
   ```

4. Create an optimized version (`Dockerfile.node.opt`):
   ```dockerfile
   FROM node:18-alpine
   
   WORKDIR /app
   
   COPY package*.json ./
   RUN npm install --production
   
   COPY server.js .
   
   EXPOSE 8080
   CMD ["node", "server.js"]
   ```

5. Build and compare both versions:
   ```bash
   docker build -t node-unopt -f Dockerfile.node.unopt .
   docker build -t node-opt -f Dockerfile.node.opt .
   docker images | grep node-
   ```

❓ **Question**: How does the order of COPY instructions affect layer caching and rebuild times?

### Challenge
Create a multi-stage build for a React application:
1. Stage 1: Build the React application
2. Stage 2: Serve it using nginx
3. Optimize for:
   - Minimal image size
   - Efficient layer caching
   - Fast rebuild times
4. Include development and production configurations
5. Bonus: Add build-time arguments for configuration

### Key Takeaways
- Multi-stage builds separate build and runtime environments
- Base image selection significantly impacts final image size
- Layer order affects build cache efficiency
- Different applications may require different optimization strategies

### Clean Up
```bash
docker rmi app-basic app-multistage app-scratch node-unopt node-opt
```

### Extra Notes
- Use `scratch` for compiled languages without runtime dependencies
- Use `alpine` when you need a minimal OS environment
- Consider security implications when choosing base images
- Layer optimization is crucial for CI/CD efficiency
- Always verify application functionality in optimized containers 