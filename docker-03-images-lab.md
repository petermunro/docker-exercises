# Docker Images Lab Exercises

__Objective__: Learn to work with Docker images through hands-on exercises.

## Exercise 1: Exploring Local Images
Learn to list and inspect local Docker images.

1. List all Docker images on your system
2. Filter the list to show only Ubuntu images
3. List images and format the output to show only repository names and tags
4. Count how many images you have locally

<details>
<summary>Reveal Solution</summary>

```bash
# List all images
docker images

# Filter for Ubuntu images
docker images ubuntu

# Format output
docker images --format "{{.Repository}}:{{.Tag}}"

# Count images
docker images | wc -l
```
</details>

## Exercise 2: Pulling Images
Learn to download images from Docker Hub.

1. Pull the latest mysql image
2. Pull a specific version of Ubuntu (20.04)
3. Try pulling both images again and observe the output
4. List your images to verify they're present

<details>
<summary>Reveal Solution</summary>

```bash
# Pull mysql latest
docker pull mysql:latest

# Pull specific Ubuntu version
docker pull ubuntu:20.04

# Pull again (should show "Image is up to date")
docker pull mysql:latest
docker pull ubuntu:20.04

# List images
docker images
```
</details>

## Exercise 3: Image History
Learn to inspect image layers.

1. Show the history of the nginx image
2. Show the history of the Ubuntu image
3. Compare their number of layers
4. Find the total size of each image

<details>
<summary>Reveal Solution</summary>

```bash
# Show nginx history
docker history nginx

# Show Ubuntu history
docker history ubuntu

# Count layers (in a bash shell)
docker history nginx | wc -l
docker history ubuntu | wc -l

# Show image sizes
docker images nginx
docker images ubuntu
```
</details>

## Exercise 4: Image Cleanup
Learn to manage local images.

1. Remove the mysql image
2. Try to remove the Ubuntu image while a container is using it
3. Remove all unused images
4. List images to verify the cleanup

<details>
<summary>Reveal Solution</summary>

```bash
# Remove mysql
docker rmi mysql

# Try removing Ubuntu (will fail if container exists)
docker rmi ubuntu

# Remove unused images
docker image prune

# List remaining images
docker images
```
</details>

## Exercise 5: Working with Tags
Learn to manage image tags.

1. Pull the nginx:latest image
2. Create a new tag for it called "my-nginx"
3. List images to see both tags
4. Remove the "my-nginx" tag
5. Verify the original image remains

<details>
<summary>Reveal Solution</summary>

```bash
# Pull nginx
docker pull nginx:latest

# Create new tag
docker tag nginx:latest my-nginx:latest

# List images
docker images

# Remove tag
docker rmi my-nginx:latest

# Verify original remains
docker images nginx
```
</details>

## Exercise 6: Image Inspection
Learn to get detailed image information.

1. Inspect the nginx image
2. Use format option to show:
   - Exposed ports
   - Environment variables
   - Command
3. Compare this information with the Ubuntu image

<details>
<summary>Reveal Solution</summary>

```bash
# Full inspection
docker inspect nginx

# Show specific details
docker inspect -f '{{.Config.ExposedPorts}}' nginx
docker inspect -f '{{.Config.Env}}' nginx
docker inspect -f '{{.Config.Cmd}}' nginx

# Compare with Ubuntu
docker inspect ubuntu
```
</details>

## Exercise 7: Image Stats
Learn to monitor image disk usage.

1. Show disk space used by Docker images
2. List images sorted by size
3. Check how much space would be freed by pruning
4. Show detailed system information

<details>
<summary>Reveal Solution</summary>

```bash
# Show disk usage
docker system df

# Sort images by size
docker images --format "{{.Size}}\t{{.Repository}}:{{.Tag}}" | sort -h

# Check potential space savings
docker system df -v

# Show system info
docker system info
```
</details>

## Next Steps
After completing these exercises, you should be comfortable with:
- Listing and filtering images
- Pulling images from Docker Hub
- Managing image tags
- Understanding image layers
- Inspecting image details

Try combining these concepts by working with different images and exploring their characteristics!
