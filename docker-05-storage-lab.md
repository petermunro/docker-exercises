# Docker Storage Lab Exercises

__Objective__: Learn how to work with Docker volumes, bind mounts, and manage persistent data through hands-on exercises.

## Exercise 1: Basic Volume Operations
Learn how to create and manage Docker volumes.

1. Create a new volume named "my-data"
2. List all volumes on your system
3. Inspect the volume you just created
4. Create another volume with a different name
5. Remove the first volume
6. List volumes again to verify the removal

<details>
<summary>Reveal Solution</summary>

```bash
# Create volume
docker volume create my-data

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-data

# Create another volume
docker volume create my-second-volume

# Remove first volume
docker volume rm my-data

# Verify removal
docker volume ls
```
</details>

## Exercise 2: Media Upload Service
Learn how volumes persist user-generated content by working with a file upload application.

1. Clone the file upload application repository:
   ```bash
   git clone https://github.com/petermunro/file-upload-demo.git
   cd file-upload-demo
   ```

2. Build the Docker image:
   ```bash
   docker build -t file-upload-demo .
   ```

3. Create a named volume for storing uploads:
   ```bash
   docker volume create upload-data
   ```

4. Run the container with the volume mounted:
   ```bash
   docker run -d \
     --name upload-app \
     -p 5000:5000 \
     -v upload-data:/app/uploads \
     file-upload-demo
   ```

5. Test the application:
   - Open http://localhost:5000 in your browser
   - Upload several files through the web interface
   - Verify the files are listed on the page

6. Demonstrate persistence:
   - Stop and remove the container
   - Create a new container using the same app and volume
   - Verify your uploads are still available
   - You can also exec into the container (or start an Ubuntu one) and verify the files are present

<details>
<summary>Reveal Solution</summary>

```bash
# Stop and remove the first container
docker stop upload-app
docker rm upload-app

# Create new container with same volume
docker run -d \
  --name upload-app-2 \
  -p 5000:5000 \
  -v upload-data:/app/uploads \
  file-upload-demo

# Verify files persist by visiting http://localhost:5000

# Optional: Inspect files directly in container
docker exec upload-app-2 ls -l /app/uploads

# Start an Ubuntu container and verify files are present
docker run --volume upload-data:/uploads ubuntu ls -l /uploads

# Clean up
docker stop upload-app-2
docker rm upload-app-2
docker volume rm upload-data
```
</details>



## Exercise 3: Configuration Management with Bind Mounts

Learn how to use bind mounts to manage application configuration files externally.

1. Create a simple Express.js application that reads configuration from a file
2. Use bind mounts to modify the configuration without rebuilding the container
3. Observe real-time configuration updates

### Setup

1. Clone the repository:
```bash
git clone https://github.com/petermunro/docker-config-demo.git
cd docker-config-demo
```

### Exercise Steps

1. Build the image:
```bash
# Build the image
docker build -t config-demo .
```

2. Run with config directory mounted as a bind mount
    - The `config` directory needs to be mapped in to the container's directory at `/app/config`
    - The container listens on port 3000, so you'll need to map that in too

3. Test the initial configuration:
    - Visit http://localhost:3000


4. Modify the configuration without restarting the container
   - On your host, edit the `config/settings.json` file. Try different scenarios:
        - Change the theme
        - Change the welcome message
        - Change the font size

5. Test the updated configuration:
    - Visit http://localhost:3000

6. Clean up:
```bash
docker stop config-app
docker rm config-app
```

<details>
<summary>Reveal Solution</summary>

```bash
docker run -d --name config-app \
    -p 3000:3000 \
    -v $(pwd)/config:/app/config \
    config-demo
```
</details>



## Exercise 4: Volume Sharing Between Containers

Learn how multiple containers can share the same volume.

1. Create a volume named "shared-data"
2. Run a container that writes to this volume
    - Hint: this command will write data to the volume, but of course you will need to run it in a container:
        ```bash
        sh -c 'echo "Hello from container 1" > /data/test.txt'
        ```
3. Run another container that reads from the same volume
    - Hint: the Unix command `cat` will read data from a file

4. Verify that both containers can access the same data

<details>
<summary>Reveal Solution</summary>

```bash
# Create volume
docker volume create shared-data

# Run first container to write data
docker run --rm -v shared-data:/data alpine sh -c 'echo "Hello from container 1" > /data/test.txt'

# Run second container to read data
docker run --rm -v shared-data:/data alpine cat /data/test.txt


# Alternative solution:
# Run both containers simultaneously
docker run -d --name writer -v shared-data:/data alpine tail -f /dev/null
docker run -d --name reader -v shared-data:/data alpine tail -f /dev/null

# Verify data access
docker exec writer sh -c 'echo "New data" > /data/new.txt'
docker exec reader cat /data/new.txt

# Clean up
docker stop writer reader
docker rm writer reader
```
</details>




## Exercise 5: Volume Labels

Learn how to organize volumes using labels.

1. Create volumes with different labels
2. List volumes with a specific label
3. Filter volumes based on labels
4. Remove volumes with specific labels
    - Hint: use bash shell command substitution to pass the label to the `docker volume rm` command
        - Example: `echo Today is $(date)`

<details>
<summary>Reveal Solution</summary>

```bash
# Create volumes with labels
docker volume create --label env=prod --label app=web prod-web-data
docker volume create --label env=dev --label app=web dev-web-data
docker volume create --label env=prod --label app=db prod-db-data

# List volumes with label
docker volume ls --filter label=env=prod

# Remove volumes with label
docker volume rm $(docker volume ls --filter label=env=dev -q)

# List remaining volumes
docker volume ls
```
</details>


## Exercise 6: Volume Cleanup
Learn how to manage volume cleanup and avoid storage leaks.

1. Create several volumes
2. Create and remove containers that use these volumes
3. Identify unused volumes
4. Clean up unused volumes
5. Verify cleanup

<details>
<summary>Reveal Solution</summary>

```bash
# Create volumes
docker volume create vol1
docker volume create vol2
docker volume create vol3

# Use volumes in containers
docker run --rm -v vol1:/data1 -v vol2:/data2 alpine echo "test"
docker run --rm -v vol3:/data3 alpine echo "test"

# List unused volumes
docker volume ls

# Remove unused volumes
docker volume prune -f

# Verify cleanup
docker volume ls
```
</details>



## Next Steps
After completing these exercises, you should be comfortable with:
- Creating and managing Docker volumes
- Using bind mounts
- Working with tmpfs mounts
- Sharing data between containers
- Backing up and restoring volume data
- Volume organization and cleanup
- Complex storage scenarios

---

## Appendix: Further Exercises

## Exercise A1: Volume Drivers

Learn about volume drivers and their capabilities.

1. List available volume drivers
2. Create a volume with the local driver explicitly
3. Inspect the volume to see its driver
4. Try to create a volume with different options

<details>
<summary>Reveal Solution</summary>

```bash
# List volume drivers
docker info | grep "Volume"

# Create volume with explicit driver
docker volume create --driver local my-local-vol

# Inspect volume
docker volume inspect my-local-vol

# Create volume with options
docker volume create --driver local \
  --opt type=none \
  --opt o=bind \
  --opt device=/home/user/data \
  my-custom-vol

# Clean up
docker volume rm my-local-vol my-custom-vol
```
</details>



## Exercise A2: Volume Backup and Restore

Learn how to backup and restore volume data.

1. Create a volume and add some data to it
2. Create a backup of the volume data
    - Hint: the Unix command `tar` can be used to create a backup
3. Remove the original volume
4. Create a new volume, and then restore the data
    - Hint: you can again use the Unix command `tar` to restore the data
5. Verify the restored data

<details>
<summary>Reveal Solution</summary>

```bash
# Create volume and add data
docker volume create data-vol
docker run --rm -v data-vol:/data alpine sh -c 'echo "Important data" > /data/file.txt'

# Backup volume
docker run --rm -v data-vol:/source -v $(pwd):/backup alpine tar czf /backup/backup.tar /source

# Remove original volume
docker volume rm data-vol

# Create new volume and restore
docker volume create data-vol-restored
docker run --rm -v data-vol-restored:/source -v $(pwd):/backup alpine sh -c 'cd /source && tar xzf /backup/backup.tar --strip 1'

# Verify restoration
docker run --rm -v data-vol-restored:/data alpine cat /data/file.txt
```
</details>



## Exercise A3: tmpfs Mounts

Learn how to use temporary file system mounts (Linux only).

1. Run a container with a tmpfs mount
2. Write some data to the tmpfs mount
3. Verify the data exists
4. Restart the container
5. Verify the data is gone

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with tmpfs
docker run -d --name tmpfs-test --tmpfs /app:rw,noexec,nosuid,size=100M alpine tail -f /dev/null

# Write data
docker exec tmpfs-test sh -c 'echo "Temporary data" > /app/temp.txt'

# Verify data
docker exec tmpfs-test cat /app/temp.txt

# Restart container
docker restart tmpfs-test

# Verify data is gone
docker exec tmpfs-test ls /app

# Clean up
docker stop tmpfs-test
docker rm tmpfs-test
```
</details>


## Exercise A4: Advanced Volume Scenarios

Combine multiple concepts in a more complex scenario.

1. Create a development environment with:
   - A volume for database data
   - A bind mount for source code
   - A tmpfs mount for temporary files
2. Verify all mounts are working
3. Test data persistence
4. Perform a backup
5. Clean up everything

<details>
<summary>Reveal Solution</summary>

```bash
# Setup environment
mkdir src
echo "console.log('Hello');" > src/app.js

# Create volume for DB
docker volume create db-data

# Run container with all mount types
docker run -d --name dev-env \
  -v db-data:/var/lib/mysql \
  -v $(pwd)/src:/app \
  --tmpfs /tmp:rw,noexec,nosuid \
  alpine tail -f /dev/null

# Verify mounts
docker inspect dev-env

# Test persistence
docker exec dev-env sh -c 'echo "DB data" > /var/lib/mysql/test.txt'
docker exec dev-env sh -c 'echo "Temp data" > /tmp/temp.txt'

# Backup volume
docker run --rm -v db-data:/source -v $(pwd):/backup \
  alpine tar czf /backup/db-backup.tar /source

# Clean up
docker stop dev-env
docker rm dev-env
docker volume rm db-data
rm -rf src db-backup.tar
```
</details>

