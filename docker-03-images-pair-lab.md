# Docker Image Investigation Exercise

**Format**: Pairs in breakout rooms  
**Prerequisites**: Basic Docker commands (pull, run, ls)

## Learning Objectives

- Understand how Docker images are structured
- Compare different base images and their tradeoffs
- Practice reading Docker image metadata
- Develop critical thinking about image selection

## Exercise Steps

### Part 1: Image Collection
Each pair should run:
```bash
docker pull nginx:latest
docker pull python:3.13
docker pull alpine:latest
```

### Part 2: Size Investigation
Run:
```bash
docker image ls
```

**Discussion Point 1**:
- Compare the sizes you see
- Why do you think there's such a big difference between Python and Alpine?
- What might be included in the larger images?

### Part 3: Layer Investigation
For each image, run:
```bash
docker image history nginx:latest
docker image history python:3.13
docker image history alpine:latest
```

**Discussion Point 2**:
- What patterns do you notice in the layers?
- Which image has the most layers? Why?
- What types of commands do you see in the layer history?

### Part 4: Deep Dive
Run:
```bash
docker inspect alpine:latest
```

**Discussion Point 3**:
- What interesting metadata can you find?
- What environment variables are set?
- What's the default command?
- Compare this with the Python image - what's different?

### Key Discussion Questions for Wrap-up
1. When would you choose each of these base images?
2. What are the tradeoffs between image size and functionality?
3. How might these differences impact your production environment?

## Tips for Instructors
- Encourage pairs to share their screens with each other
- Have them take turns reading out and interpreting the command outputs
- Ask them to make notes of any surprising findings
- If time permits, let pairs share their most interesting discovery with the main group

## Common Findings to Highlight
- Python image size vs included functionality
- Alpine's minimalist approach
- Layer reuse between images
- Default commands and environments
