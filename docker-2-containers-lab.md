# Docker Container Lab Exercises

__Objective__: Learn how to work with Docker containers through progressive hands-on exercises.

## Exercise 1: Your First Interactive Container
Learn how containers work interactively.

1. Run an Ubuntu container without any options
2. Notice how it exits immediately (try `docker ps` to verify)
3. What happens if you provide a command, such as `date`, `whoami` or `bash` as the last parameter?
4. Now run another 'bash' shell in an Ubuntu container with the interactive (`-it`) flags
5. Again run `docker ps` (in another shell or command window). What name did it give to your container?
6. Try some Linux commands in your interactive container (e.g., `ls`, `pwd`, `whoami`)
7. Exit the container (type `exit`)
8. Run `docker ps`. Is it listed?
9. Run `docker ps -a` to see all containers, including stopped ones

<details>
<summary>Reveal Solution</summary>

```bash
# First attempt (will exit immediately)
docker run ubuntu

# Verify no running containers
docker ps

# Provide a command
docker run ubuntu date

# Run interactively
docker run -it ubuntu

# Show all containers including stopped ones
docker ps

# Inside container, try commands like:
ls
pwd
whoami
exit

# Show all containers
docker ps

# Show all containers including stopped ones
docker ps -a
```

</details>

## Exercise 2: Container Cleanup
Learn about container persistence and cleanup.

1. Run an Ubuntu bash shell container with `-it` flags
2. Exit the container
3. Run `docker ps -a` to see the stopped container
4. Remove the stopped container using `docker rm`
5. Verify it's gone with `docker ps -a`
6. Try running a container with `--rm` flag. What does this do?

<details>
<summary>Reveal Solution</summary>

```bash
# Run container
docker run -it ubuntu
exit

# List all containers
docker ps -a

# Remove container (use container ID or name from docker ps -a)
docker rm <container_id>

# Verify removal
docker ps -a

# Run with automatic removal
docker run --rm -it ubuntu
exit

# Verify it's gone
docker ps -a
```
</details>

## Exercise 3: Named Containers
Learn to work with named containers.

1. Run an Ubuntu container with `-it` flags and name it "my-ubuntu"
2. Exit the container
3. List all containers and find your named container
4. Start the same container again using `docker start -i my-ubuntu`. How does `docker start` differ from `docker run`?
5. Exit and remove the container

<details>
<summary>Reveal Solution</summary>

```bash
# Run named container
docker run -it --name my-ubuntu ubuntu

# After exit, list all containers
docker ps -a

# Start existing container interactively
docker start -i my-ubuntu

# After exit, remove container
docker rm my-ubuntu
```
</details>

## Exercise 4: Background Containers
Learn to run containers in detached mode.

1. Run an nginx container in detached mode using `-d`
2. List running containers
3. Stop the container
4. Remove the container
5. Run another nginx container with both `-d` and `--name` flags
6. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run detached container
docker run -d nginx

# List running containers
docker ps

# Stop container (use container ID from docker ps)
docker stop <container_id>

# Remove container
docker rm <container_id>

# Run named detached container
docker run -d --name my-nginx nginx

# Clean up
docker stop my-nginx
docker rm my-nginx
```
</details>

## Exercise 5: Port Mapping
Learn to map container ports to host ports.

1. Run an nginx container in detached mode
2. Try to access it at http://localhost:80 (it won't work)
3. Remove that container
4. Run a new nginx container with port 80 mapped to port 8080
5. Access it at http://localhost:8080
6. List running containers and observe the port mapping
7. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run without port mapping
docker run -d --name nginx1 nginx

# Remove container
docker stop nginx1
docker rm nginx1

# Run with port mapping
docker run -d --name nginx2 -p 8080:80 nginx

# List containers to see port mapping
docker ps

# Clean up
docker stop nginx2
docker rm nginx2
```
</details>

## Exercise 6: Container Logs
Learn to view container logs.

1. Run a container that pings google.com continuously
   (Hint: Use `docker run alpine ping google.com`)
2. List running containers to get its ID or name
3. View the container logs
4. Try following the logs with `-f` flag
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run ping container
docker run -d --name pinger alpine ping google.com

# View logs
docker logs pinger

# Follow logs (use Ctrl+C to exit)
docker logs -f pinger

# Clean up
docker stop pinger
docker rm pinger
```
</details>

## Exercise 7: Basic Container Inspection
Learn to inspect container details.

1. Run an nginx container in detached mode with a name
2. Use `docker inspect` to view its full configuration
3. Look through the output to find interesting information like:
   - The container's IP address
   - The container's state
   - The container's configuration
4. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container
docker run -d --name inspect-basic nginx

# Inspect container
docker inspect inspect-basic

# Clean up
docker stop inspect-basic
docker rm inspect-basic
```
</details>

## Exercise 8: Formatted Inspection
Learn to extract specific information from container inspection.

1. Run an nginx container in detached mode with a name
2. Use `docker inspect` with format option to extract the container's IP address
   - Use this command: `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_name>`
3. Try getting the container's state:
   - Use this command: `docker inspect -f '{{.State.Status}}' <container_name>`
4. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container
docker run -d --name inspect-format nginx

# Get IP address
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' inspect-format

# Get container state
docker inspect -f '{{.State.Status}}' inspect-format

# Clean up
docker stop inspect-format
docker rm inspect-format
```
</details>

## Exercise 9: Resource Limits
Learn to set and verify container resource limits.

1. Run an nginx container with these resource limits:
   - Maximum memory: 512MB
   - CPU limit: 0.5 (50% of one CPU)
2. Inspect the container and look for these limits under "HostConfig"
   (Hint: Use `docker inspect <container_name> | grep -A 10 "HostConfig"`)
3. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with resource limits
docker run -d --name limited-nginx --memory=512m --cpus=0.5 nginx

# Check resource limits
docker inspect limited-nginx | grep -A 10 "HostConfig"

# Clean up
docker stop limited-nginx
docker rm limited-nginx
```
</details>

## Exercise 10: Environment Variables
Learn to work with container environment variables.

1. Run an Alpine container with these environment variables:
   - NAME=student
   - COURSE=docker
2. Make it run the `env` command to list its environment
3. Try running another container with different variables
4. Use `docker inspect` to view the environment variables
   (Hint: Look in the "Config" section)

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with environment variables
docker run --rm -e NAME=student -e COURSE=docker alpine env

# Run another with different variables
docker run --rm -e STUDENT_ID=12345 -e LEVEL=beginner alpine env

# Run container for inspection
docker run -d --name env-test -e NAME=student -e COURSE=docker alpine sleep 1d

# Inspect environment variables
docker inspect env-test

# Clean up
docker stop env-test
docker rm env-test
```
</details>

## Next Steps
After completing these exercises, you should be comfortable with:
- Running containers interactively and in the background
- Container lifecycle management
- Basic container operations
- Port mapping
- Logging
- Container inspection
- Resource management
- Environment variables

Try combining these concepts to create more complex container deployments!
