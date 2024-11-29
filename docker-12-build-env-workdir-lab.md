## Working with Environment Variables and Directories

In this lab, you'll learn about Dockerfile instructions that help you set up the working environment inside your containers. You'll explore `WORKDIR`, `ENV`, `ARG`, and the differences between `COPY` and `ADD`.

### Prerequisites
- Completed the previous Dockerfile labs
- Docker installed and running on your machine

### Learning Objectives
- Set up and manage working directories in containers
- Work with environment variables
- Use build arguments for flexible image building
- Understand when to use COPY vs ADD

### The Lab

#### Step 1: Working with WORKDIR
1. Create a new directory and navigate to it:
   ```bash
   mkdir docker-env-lab
   cd docker-env-lab
   ```

2. Create a Dockerfile that demonstrates WORKDIR:
   ```dockerfile
   FROM ubuntu:22.04
   
   WORKDIR /app
   RUN pwd > location.txt
   
   WORKDIR /app/subdirectory
   RUN pwd >> ../location.txt
   
   WORKDIR /another/path
   RUN pwd >> /app/location.txt
   ```

3. Build and check the results:
   ```bash
   docker build -t workdir-demo .
   docker run workdir-demo cat /app/location.txt
   ```

❓ **Question**: What do you think would happen if we used relative paths in the RUN commands instead of absolute paths? Why is WORKDIR important?

#### Step 2: Environment Variables with ENV
1. Create a new Dockerfile named `Dockerfile.env`:
   ```dockerfile
   FROM ubuntu:22.04
   
   ENV APP_HOME=/application \
       APP_ENV=production \
       DEBUG_MODE=false
   
   WORKDIR ${APP_HOME}
   
   RUN echo "Environment: $APP_ENV" > config.txt && \
       echo "Debug Mode: $DEBUG_MODE" >> config.txt
   
   CMD ["cat", "config.txt"]
   ```

2. Build and run with different environment settings:
   ```bash
   docker build -t env-demo -f Dockerfile.env .
   docker run env-demo
   docker run -e DEBUG_MODE=true env-demo
   ```

   ❓ **Question**: What's the difference between setting environment variables in the Dockerfile and passing them at runtime?

#### Step 3: Build Arguments with ARG
1. Create a new Dockerfile named `Dockerfile.arg`:
   ```dockerfile
   FROM ubuntu:22.04
   
   ARG VERSION=latest
   ARG BUILD_DATE
   
   LABEL version=${VERSION}
   LABEL build-date=${BUILD_DATE}
   
   ENV APP_VERSION=${VERSION}
   
   RUN echo "Building version: $VERSION" > build-info.txt && \
       echo "Build date: $BUILD_DATE" >> build-info.txt
   
   CMD ["cat", "build-info.txt"]
   ```

2. Build with different arguments:
   ```bash
   docker build -t arg-demo -f Dockerfile.arg .
   docker build -t arg-demo:1.0 -f Dockerfile.arg --build-arg VERSION=1.0 --build-arg BUILD_DATE=$(date +%F) .
   ```

❓ **Question**: Why might you use ARG instead of ENV? What's the key difference between them?

#### Step 4: COPY vs ADD
1. Create some test files:
   ```bash
   echo "Regular file" > regular.txt

   mkdir sample-content
   echo "sample file 1" > sample-content/file1
   echo "sample file 2" > sample-content/file2
   echo "sample file 3" > sample-content/file3
   tar cvf test.tar.gz sample-content
   ```

2. Create a Dockerfile named `Dockerfile.copy`:
   ```dockerfile
   FROM ubuntu:22.04
   
   WORKDIR /app
   
   # Using COPY
   COPY regular.txt ./
   COPY test.tar.gz ./copy-example/
   
   # Using ADD
   ADD test.tar.gz ./add-example/
   ADD https://xanadu.ie/robots.txt ./add-example/
   
   CMD ["ls", "-R", "/app"]
   ```

3. Build and examine the differences:
   ```bash
   docker build -t copy-demo -f Dockerfile.copy .
   docker run copy-demo
   ```

❓ **Question**: Looking at the output, what are the key differences between how COPY and ADD handled the files?

### Challenge
Create a configurable application builder:
1. Create a Dockerfile that builds a simple application
2. Use ARG to configure:
   - Base image version
   - Application version
   - Build environment (dev/prod)
3. Use ENV to set runtime configurations
4. Use WORKDIR to organize your application
5. Use both COPY and ADD appropriately
6. Make it possible to override configurations at build and run time

### Key Takeaways
- WORKDIR helps organize your container filesystem
- ENV variables persist in the running container
- ARG is only available during build time
- COPY is preferred for simple file copying
- ADD has special features for URLs and archives

### Clean Up
```bash
docker rmi workdir-demo env-demo arg-demo arg-demo:1.0 copy-demo
```

### Extra Notes
- WORKDIR is preferred over using RUN cd
- ENV values can be overridden at runtime
- ARG values must be provided at build time
- COPY is more explicit and preferred for most cases
- ADD should be used sparingly and only when its special features are needed 