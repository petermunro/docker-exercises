## Container Entry Points and Commands

In this lab, you'll learn about the `CMD` and `ENTRYPOINT` instructions in Dockerfiles. These instructions are crucial for controlling how containers start and behave when they run.

### Prerequisites
- Completed the "Getting Started with Dockerfiles" lab
- Docker installed and running on your machine

### Learning Objectives
- Understand the difference between CMD and ENTRYPOINT
- Learn how to use CMD and ENTRYPOINT together
- Explore how to override default commands
- Create containers with configurable behavior

### Estimated Time
30 minutes

### The Lab

#### Step 1: Understanding CMD
1. Create a new directory and navigate to it:
   ```bash
   mkdir cmd-entrypoint-lab
   cd cmd-entrypoint-lab
   ```

2. Create a Dockerfile with a simple CMD:
   ```dockerfile
   FROM ubuntu:22.04
   CMD ["echo", "Hello, Docker!"]
   ```

3. Build and run this image:
   ```bash
   docker build -t cmd-demo .
   docker run cmd-demo
   ```

4. Now try overriding the CMD:
   ```bash
   docker run cmd-demo echo "Different message"
   ```

❓ **Question**: What happened when you ran the container with and without the override? Why?

#### Step 2: Working with ENTRYPOINT
1. Create a new Dockerfile named `Dockerfile.entrypoint`:
   ```dockerfile
   FROM ubuntu:22.04
   ENTRYPOINT ["echo", "Hello"]
   CMD ["World"]
   ```

2. Build and run this image:
   ```bash
   docker build -t entrypoint-demo -f Dockerfile.entrypoint .
   docker run entrypoint-demo
   docker run entrypoint-demo Docker
   ```

❓ **Question**: How does the behavior differ from the previous example? What happens to the CMD parameter?

#### Step 3: Creating a Practical Example
1. Create a script named `greet.sh`:
   ```bash
   #!/bin/bash
   if [ $# -eq 0 ]; then
       echo "Hello, stranger!"
   else
       echo "Hello, $@!"
   fi
   ```

2. Create a new Dockerfile:
   ```dockerfile
   FROM ubuntu:22.04
   COPY greet.sh /usr/local/bin/
   RUN chmod +x /usr/local/bin/greet.sh
   ENTRYPOINT ["/usr/local/bin/greet.sh"]
   CMD ["stranger"]
   ```

3. Build and test the image:
   ```bash
   docker build -t greeter .
   docker run greeter
   docker run greeter Alice
   docker run greeter Alice and Bob
   ```

❓ **Question**: What's the benefit of using ENTRYPOINT with CMD in this example?

#### Step 4: Shell vs Exec Form
1. Create two more Dockerfiles to understand the different forms:

   `Dockerfile.shell`:
   ```dockerfile
   FROM ubuntu:22.04
   CMD echo "Running in shell form"
   ```

   `Dockerfile.exec`:
   ```dockerfile
   FROM ubuntu:22.04
   CMD ["echo", "Running in exec form"]
   ```

2. Build and run both:
   ```bash
   docker build -t shell-form -f Dockerfile.shell .
   docker build -t exec-form -f Dockerfile.exec .
   docker run shell-form
   docker run exec-form
   ```

❓ **Question**: Try running 'ps' inside both containers. What differences do you notice in the process list?

### Challenge
Create a configurable weather report container:
1. Create a script that accepts a city name and outputs "Weather report for [city]: Sunny"
2. Use ENTRYPOINT to run the script
3. Use CMD to provide a default city
4. Make it possible to override the city at runtime
5. Bonus: Add a `-u` flag that makes the city uppercase

### Key Takeaways
- CMD provides default arguments that can be overridden
- ENTRYPOINT configures the container to run as an executable
- ENTRYPOINT and CMD can be used together for flexible container behavior
- The exec form `["command", "param1"]` is preferred over shell form `command param1`

### Clean Up
```bash
docker rmi cmd-demo entrypoint-demo greeter shell-form exec-form
```

### Extra Notes
- ENTRYPOINT can be overridden using the `--entrypoint` flag
- Shell form runs your command inside a shell, which has implications for signal handling
- Exec form doesn't invoke a command shell, making it more suitable for production
- When both ENTRYPOINT and CMD are used, CMD provides default arguments to ENTRYPOINT 