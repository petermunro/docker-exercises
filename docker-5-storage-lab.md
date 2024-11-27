# Docker Storage Lab Exercises

__Objective__: Learn how to work with Docker volumes, bind mounts, and manage persistent data through hands-on exercises.

## Exercise 1: Basic Volume Operations
Learn how to create and manage Docker volumes.

1. Create a new volume named "my-data"
2. List all volumes on your system
3. Inspect the volume you just created
4. Create another volume with a different name
5. Try to remove the first volume
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

## Exercise 2: Using Named Volumes
Learn how to use volumes with containers.

1. Create a new volume named "nginx-data"
2. Run an nginx container that uses this volume to store its logs
   - Mount the volume to `/var/log/nginx` in the container
3. Inspect the container to verify the volume mount
4. Stop and remove the container
5. Create a new container using the same volume
6. Verify that the data persists

<details>
<summary>Reveal Solution</summary>

```bash
# Create volume
docker volume create nginx-data

# Run container with volume
docker run -d --name nginx1 -v nginx-data:/var/log/nginx nginx

# Inspect container
docker inspect nginx1

# Stop and remove container
docker stop nginx1
docker rm nginx1

# Run new container with same volume
docker run -d --name nginx2 -v nginx-data:/var/log/nginx nginx

# Verify data persistence
docker exec nginx2 ls -l /var/log/nginx
```
</details>

## Exercise 3: Bind Mounts
Learn how to use bind mounts to share files between host and container.

1. Create a new directory on your host system named "web-content"
2. Create an index.html file in this directory with some basic HTML
3. Run an nginx container that uses a bind mount to serve this content
4. Access the website through your browser
5. Modify the index.html file on your host
6. Refresh your browser to see the changes

<details>
<summary>Reveal Solution</summary>

```bash
# Create directory and file
mkdir web-content
echo "<h1>Hello from bind mount!</h1>" > web-content/index.html

# Run container with bind mount
docker run -d --name nginx-web -p 8080:80 -v $(pwd)/web-content:/usr/share/nginx/html nginx

# Modify the file
echo "<h1>Updated content!</h1>" > web-content/index.html

# Access at http://localhost:8080

# Clean up
docker stop nginx-web
docker rm nginx-web
```
</details>

## Exercise 4: Volume Sharing Between Containers
Learn how multiple containers can share the same volume.

1. Create a volume named "shared-data"
2. Run a container that writes to this volume
3. Run another container that reads from the same volume
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

## Exercise 5: Volume Backup and Restore
Learn how to backup and restore volume data.

1. Create a volume and add some data to it
2. Create a backup of the volume data
3. Remove the original volume
4. Create a new volume and restore the data
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

## Exercise 6: tmpfs Mounts
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

## Exercise 7: Volume Drivers
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

## Exercise 8: Volume Labels
Learn how to organize volumes using labels.

1. Create volumes with different labels
2. List volumes with a specific label
3. Filter volumes based on labels
4. Remove volumes with specific labels

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

## Exercise 9: Volume Cleanup
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

## Exercise 10: Advanced Volume Scenarios
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

## Next Steps
After completing these exercises, you should be comfortable with:
- Creating and managing Docker volumes
- Using bind mounts
- Working with tmpfs mounts
- Sharing data between containers
- Backing up and restoring volume data
- Volume organization and cleanup
- Complex storage scenarios

Try creating your own development environments using these storage concepts! 