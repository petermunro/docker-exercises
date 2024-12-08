# Docker "Hello World" Lab Exercises

__Objective__: Get comfortable with basic Docker commands through simple hands-on exercises.

## Exercise 1: Hello Docker
Learn the very basics of running a container.

1. Run the "hello-world" container
2. Read the output message carefully
3. List all running containers with `docker ps`. Does anything appear?
4. List all containers (both running and stopped ones) with `docker ps -a`.
    - What name was given to your container?
    - What command was run on the container?
    - What was the name of the image used?
5. Run the "hello-world" container again. Once again, list all containers. What differs from the first time?
6. Remove all stopped containers
    - You can remove them one by one with `docker rm <containerId>`.
    - Note that you don't need to type the entire `containerId` - just enough characters to make it unambiguous.

<details>
<summary>Reveal Solution</summary>

```bash
# Run hello-world container
docker run hello-world

# List running containers
docker ps

# List all containers including stopped ones
docker ps -a

# Run it again
docker run hello-world

# List all containers again
docker ps -a

# Remove stopped containers
docker rm <myContainerId>
```
</details>

## Exercise 2: Getting Help

1. Ask docker for help using `docker help`.
2. Ask docker about the `container` command using `docker help container`.
3. What kinds of things can you do with a container?
    - Notice that commands can be of two kinds, for example:
        - `docker container run`
        - `docker run`
    - The second form is really just a shortcut for the first.
    - Only the common sub-commands have a shortcut.
4. Try running a container with both forms of the command.
5. What options does the `run` command take?


<details>
<summary>Reveal Solution</summary>

```bash
# Ask for help
docker help

# Ask for help about the container command
docker help container

# Run a container with both forms of the command
docker container run hello-world
docker run hello-world

# What options does the run command take?
docker help container run
# or:
docker help run
```
</details>

## Exercise 3: Simple Container Commands
Learn to run basic commands in containers.

1. Run "echo Hello from Alpine" in an Alpine container
2. Run "date" in an Ubuntu container
3. Try running both commands again with the `--rm` flag
4. List all containers - what's different when using `--rm`?

<details>
<summary>Reveal Solution</summary>

```bash
# Run echo in Alpine
docker run alpine echo "Hello from Alpine"

# Run date in Ubuntu
docker run ubuntu date

# Run with auto-removal
docker run --rm alpine echo "Hello from Alpine"
docker run --rm ubuntu date

# List all containers
docker ps -a
```
</details>

## Exercise 4: Basic Docker Information
Learn to get information about your Docker installation.

1. Check your Docker version with `docker --version`
2. What does `docker version` do (as a command, not an option)?
3. Display system-wide information with `docker info`
    - How many containers are running, paused and stopped?
4. Display Docker disk usage using `docker system df`

<details>
<summary>Reveal Solution</summary>

```bash
# Check version of the Docker client only
docker --version

# Display version information of the Docker client and daemon (server)
docker version

# System info
docker info

# Check disk usage
docker system df
```
</details>

## Exercise 5: Interactive Ubuntu Container
Learn how to run and interact with a container in interactive mode.

1. Using the Ubuntu image, start a container in interactive mode
2. Name your container "my-ubuntu"
3. When the container starts, you should get a shell prompt

<details>
<summary>Reveal Solution</summary>

```bash
# Start an interactive Ubuntu container
docker run -it --name my-ubuntu ubuntu bash
```

Note:
- The `-it` flags give us an interactive terminal
- `--name my-ubuntu` assigns a custom name to our container
- `ubuntu` is the image we're using
- `bash` is the command we want to run inside the container
</details>

## Exercise 6: Exploring Your Container
While inside your Ubuntu container from Exercise 5, explore the environment.

1. Check which directory you're in
2. List the contents of the root directory
3. Check which version of Ubuntu you're running

<details>
<summary>Reveal Solution</summary>

```bash
# Inside the container:
pwd
ls /
cat /etc/os-release
```

Note:
- You're in a completely isolated environment
- Only basic commands are available
- This is a minimal Ubuntu installation
</details>

## Exercise 7: Exit and Container State
Learn what happens when you exit an interactive container.

1. Exit your Ubuntu container by typing `exit` or pressing Ctrl+D
2. List all containers (including stopped ones) to see what happened to your container

<details>
<summary>Reveal Solution</summary>

```bash
# Exit the container first
exit

# List all containers
docker ps -a
```

Note:
- The container stops when you exit the shell
- It still exists but is in a "stopped" state
- You can see it with `docker ps -a` but not with just `docker ps`
</details>

