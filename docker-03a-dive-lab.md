---
## Lab Exercise: Analyzing Docker Images with Dive

### Objective
In this lab, you'll learn how to:
- Analyze the layers of a Docker image
- Identify inefficiencies in Docker images
- Compare different versions of the same image
- Understand the impact of Dockerfile commands on image size

### Prerequisites
- Docker installed and running
- Dive installed (see installation section above)

### Exercise 1: Analyzing the Official Nginx Image

1. Pull the latest Nginx image:
   ```bash
   docker pull nginx:latest
   ```

2. Set up the dive command alias:

   ```bash
   alias dive="docker run -ti --rm  -v /var/run/docker.sock:/var/run/docker.sock wagoodman/dive"
   ```

3. Analyze the image with Dive:
   ```bash
   dive nginx:latest
   ```

4. Tasks:
   - Count how many layers the image has
   - Find the largest layer and note what changes it introduces
   - Look for any duplicate files across layers
   - Calculate the total wasted space

### Exercise 2: Comparing Image Variants

1. Pull different variants of the Nginx image:
   ```bash
   docker pull nginx:alpine
   docker pull nginx:mainline
   ```

2. Analyze each variant:
   ```bash
   dive nginx:alpine
   dive nginx:mainline
   ```

3. Tasks:
   - Compare the total size of each image
   - Note the differences in base layers
   - Identify which variant would be best for production use
   - Document the trade-offs between the variants

### Exercise 3: Analyzing a Python Application

1. Create a simple Python application Dockerfile:
   ```dockerfile
   FROM python:3.9
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY . .
   CMD ["python", "app.py"]
   ```

2. Build and analyze the image:
   ```bash
   docker build -t myapp:v1 .
   dive myapp:v1
   ```

3. Tasks:
   - Identify which layer contains the pip packages
   - Look for any cached pip files that could be cleaned up
   - Note any files that appear in multiple layers
   - Suggest improvements to reduce the image size

### Exercise 4: Layer Investigation Challenge

1. Analyze the Node.js official image:
   ```bash
   docker pull node:latest
   dive node:latest
   ```

2. Investigation Tasks:
   - Use the file tree to locate where npm packages are installed
   - Find any development dependencies that might not be needed
   - Identify the layer that adds the node binary
   - Calculate the percentage of wasted space

### Tips for the Lab
- Use Tab to switch between the layer view and file tree
- Press Ctrl+Space to expand/collapse all directories
- Use Ctrl+A to see aggregated changes across layers
- Remember to look at the efficiency score in the bottom right

### Discussion Questions
1. What patterns did you notice in how official images are structured?
2. How do base images affect the overall efficiency of the final image?
3. What are some common sources of wasted space in Docker images?
4. How might you modify your Dockerfile practices based on what you learned?

### Bonus Challenge
Take any existing Dockerfile from your projects and analyze it with Dive. Create a list of potential optimizations based on your findings. Try to achieve at least a 90% efficiency score.
