## Introduction to Dockerfiles

In this lab, you'll learn the basics of creating a Dockerfile and building a Docker image. We'll start with a simple example that introduces core Dockerfile concepts and the image building process.

### Prerequisites
- Docker installed and running on your machine
- Basic familiarity with command line operations

### Learning Objectives
- Create your first Dockerfile
- Understand basic Dockerfile instructions
- Build a Docker image
- Tag Docker images

### Estimated Time
20 minutes

### The Lab

#### Step 1: Create a Working Directory
1. Create a new directory called `first-dockerfile`:
   ```bash
   mkdir first-dockerfile
   cd first-dockerfile
   ```

#### Step 2: Create Your First Dockerfile
1. Using your favorite text editor, create a file named `Dockerfile` (no extension) and add the following content:
   ```dockerfile
   FROM ubuntu:22.04
   
   # Update package list and install nginx
   RUN apt-get update && \
       apt-get install -y nginx
   
   # Create a simple index.html
   COPY index.html /var/www/html/

   # Expose port 80
   EXPOSE 80
   ```

2. Create an `index.html` file in the same directory:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My First Docker Image</title>
   </head>
   <body>
       <h1>Hello from Docker!</h1>
       <p>If you can see this, your Dockerfile worked!</p>
   </body>
   </html>
   ```

❓ **Question**: Looking at the Dockerfile above, what do you think each instruction does? Try to explain each one before moving on.

#### Step 3: Understanding Dockerfile Instructions
Let's break down each instruction:

- `FROM`: Specifies the base image. Every Dockerfile must start with a FROM instruction.
- `RUN`: Executes commands in a new layer on top of the current image.
- `COPY`: Copies files from your local filesystem into the image.
- `EXPOSE`: Informs Docker that the container will listen on specified network ports at runtime.

❓ **Question**: Why do you think we combined the `apt-get update` and `apt-get install` commands into a single RUN instruction?

#### Step 4: Building Your Image
1. Build your Docker image:
   ```bash
   docker build -t my-nginx .
   ```

2. Watch the build process and observe each step. Notice how Docker executes each instruction in your Dockerfile.

❓ **Question**: What do you think the `-t` flag does in the build command?

#### Step 5: Tagging Your Image
1. Add a version tag to your image:
   ```bash
   docker tag my-nginx my-nginx:v1.0
   ```

2. List your images to see both tags:
   ```bash
   docker images | grep my-nginx
   ```

#### Step 6: Run Your Container
1. Start a container from your image:
   ```bash
   docker run -d -p 8080:80 my-nginx:v1.0
   ```

2. Visit http://localhost:8080 in your browser to see your webpage.

### Challenge
Try these modifications to your Dockerfile:
1. Add a maintainer label using the LABEL instruction
2. Create a new tag with v1.1 and include your name in it
3. Build the image with multiple tags in a single command (hint: use multiple -t flags)

### Key Takeaways
- A Dockerfile is a text file containing instructions to build a Docker image
- Each instruction creates a new layer in the image
- Images can have multiple tags
- The build context (the `.` in the build command) matters

### Clean Up
```bash
docker stop $(docker ps -q --filter ancestor=my-nginx:v1.0)
docker rmi my-nginx:v1.0 my-nginx
``` 