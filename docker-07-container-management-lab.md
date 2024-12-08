# Advanced Docker Container Lab Exercises

__Objective__: Learn advanced Docker container management techniques including resource limits, monitoring, and container behavior control.

## Exercise 1: Basic Resource Limits
Learn to set and monitor memory limits.

1. Run an nginx container with a 100MB memory limit
2. Use `docker stats` to monitor its memory usage
3. Check the memory limit in the container's details:
   - Use `docker inspect` and look in the "HostConfig" section
   - Or use: `docker inspect -f '{{.HostConfig.Memory}}' <container_name>`
4. Try to run a container with an invalid memory limit (e.g., 1KB) and observe the error

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with memory limit
docker run -d --name mem-test --memory=100m nginx

# Monitor stats
docker stats mem-test

# Check memory limit
docker inspect -f '{{.HostConfig.Memory}}' mem-test

# Try invalid limit (will fail)
docker run -d --memory=1k nginx

# Clean up
docker stop mem-test
docker rm mem-test
```
</details>

## Exercise 2: CPU Resource Management
Learn different ways to limit CPU usage.

1. Run a container with:
   - 0.5 CPUs (50% of one CPU)
   - A maximum of 2 CPU cores
2. Start a CPU-intensive process (use stress-ng image)
3. Monitor CPU usage with `docker stats`
4. Compare this with a container without CPU limits
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with CPU limits
docker run -d --name cpu-limited \
  --cpus=0.5 \
  --cpuset-cpus=0-1 \
  nginx

# Run CPU-intensive container
docker run -d --name stress-test \
  progrium/stress --cpu 4

# Monitor both containers
docker stats cpu-limited stress-test

# Clean up
docker stop cpu-limited stress-test
docker rm cpu-limited stress-test
```
</details>

## Exercise 3: Simple Health Checks
Learn to implement basic container health monitoring.

1. Run an nginx container with a basic health check that:
   - Uses curl to check localhost
   - Checks every 30 seconds
2. Monitor the health status:
   - Use `docker ps` to see the health status column
   - Use `docker inspect` to see detailed health status
3. Try to access the container's HTTP port
4. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with health check
docker run -d --name health-basic \
  --health-cmd='curl -f http://localhost/ || exit 1' \
  --health-interval=30s \
  nginx

# Check health status
docker ps

# View detailed health info
docker inspect -f '{{.State.Health.Status}}' health-basic

# Clean up
docker stop health-basic
docker rm health-basic
```
</details>

## Exercise 4: Custom Health Checks
Learn to create more sophisticated health checks.

1. Create a container with a custom health check script that:
   - Checks multiple conditions (e.g., process running AND port accessible)
   - Has a 10-second timeout
   - Starts checking after 5 seconds
2. Monitor the health check logs
3. Intentionally break one condition and observe the health status change
4. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with custom health check
docker run -d --name health-custom \
  --health-cmd='pgrep nginx && curl -f http://localhost/ || exit 1' \
  --health-interval=30s \
  --health-timeout=10s \
  --health-start-period=5s \
  --health-retries=3 \
  nginx

# Monitor health status
docker inspect -f '{{json .State.Health}}' health-custom | jq

# View health check logs
docker inspect -f '{{range .State.Health.Log}}{{.Output}}{{end}}' health-custom

# Clean up
docker stop health-custom
docker rm health-custom
```
</details>

## Exercise 5: Basic Restart Policies
Learn about container restart behavior.

1. Run a container that exits immediately:
   - Use `alpine` image with `sleep 5` command
   - Add `--restart=always` policy
2. Watch it restart repeatedly
3. Try different restart policies:
   - no
   - always
   - on-failure
   - unless-stopped
4. Check the restart count using `docker inspect`

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with restart policy
docker run -d --name restart-test --restart=always alpine sleep 5

# Watch container restart
docker ps -a

# Check restart count
docker inspect -f '{{.RestartCount}}' restart-test

# Try different policies
docker run -d --name test-no --restart=no alpine sleep 5
docker run -d --name test-onfail --restart=on-failure alpine sleep 5
docker run -d --name test-unless --restart=unless-stopped alpine sleep 5

# Clean up
docker stop restart-test test-no test-onfail test-unless
docker rm restart-test test-no test-onfail test-unless
```
</details>

## Exercise 6: Restart Policy with Exit Codes
Learn how restart policies interact with exit codes.

1. Create a script that exits with different codes:
   ```bash
   if [ "$RANDOM" -gt 16383 ]; then
     exit 1
   else
     exit 0
   fi
   ```
2. Run a container with this script and `--restart=on-failure`
3. Monitor its behavior with `docker ps -a`
4. Check the last exit code using `docker inspect`
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Create container with random exit code
docker run -d --name exit-test --restart=on-failure \
  --entrypoint sh alpine -c \
  'while true; do if [ "$RANDOM" -gt 16383 ]; then exit 1; else exit 0; fi; done'

# Monitor container status
docker ps -a

# Check last exit code
docker inspect -f '{{.State.ExitCode}}' exit-test

# Clean up
docker stop exit-test
docker rm exit-test
```
</details>

## Exercise 7: Resource Quotas and Limits
Learn to set comprehensive resource limits.

1. Run a container with these limits:
   - Memory: 256MB
   - Memory swap: 512MB
   - CPU: 0.5
   - CPU shares: 512
   - Maximum number of processes: 20
2. Verify all limits using `docker inspect`
3. Try to exceed the process limit
4. Monitor resource usage
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with resource quotas
docker run -d --name quota-test \
  --memory=256m \
  --memory-swap=512m \
  --cpus=0.5 \
  --cpu-shares=512 \
  --pids-limit=20 \
  nginx

# Verify limits
docker inspect quota-test

# Try to exceed process limit
docker exec quota-test sh -c 'for i in $(seq 1 25); do sleep 1000 & done'

# Monitor usage
docker stats quota-test

# Clean up
docker stop quota-test
docker rm quota-test
```
</details>

## Exercise 8: Container Logging
Learn advanced logging options.

1. Run a container with custom logging options:
   - Set max log size to 10MB
   - Keep maximum 3 log files
   - Use json-file driver
2. Generate some logs
3. Check log configuration with `docker inspect`
4. Verify log rotation
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with logging options
docker run -d --name log-test \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  alpine sh -c 'while true; do echo "Log entry at $(date)"; sleep 1; done'

# Check logging configuration
docker inspect -f '{{json .HostConfig.LogConfig}}' log-test | jq

# View logs
docker logs log-test

# Clean up
docker stop log-test
docker rm log-test
```
</details>

## Exercise 9: Update Container Resources
Learn to update container resources on the fly.

1. Start a container with basic resource limits
2. Use `docker update` to change:
   - Memory limit
   - CPU limit
   - Restart policy
3. Verify changes with `docker inspect`
4. Monitor the effects
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Start container
docker run -d --name update-test \
  --memory=256m \
  --cpus=0.5 \
  nginx

# Update resources
docker update --memory=512m --cpus=1 update-test

# Verify changes
docker inspect -f '{{.HostConfig.Memory}}' update-test
docker inspect -f '{{.HostConfig.NanoCpus}}' update-test

# Clean up
docker stop update-test
docker rm update-test
```
</details>

## Exercise 10: Container Events
Learn to monitor container events.

1. Open a new terminal and start watching Docker events
2. In another terminal, perform these actions:
   - Start a container
   - Stop it
   - Remove it
   - Create another with different parameters
3. Observe the events generated
4. Filter events by container name
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Terminal 1: Watch events
docker events

# Terminal 2: Perform actions
docker run -d --name event-test nginx
docker stop event-test
docker rm event-test
docker run --name event-test2 nginx

# Watch filtered events
docker events --filter container=event-test2

# Clean up
docker stop event-test2
docker rm event-test2
```
</details>

## Exercise 11: Process Inspection
Learn to inspect processes running inside containers.

1. Run a container that spawns multiple processes:
   - Use nginx as the base image, as it will spawn a few processes
   - You can also add a few additional long-running processes of your own using `docker exec`
2. Use `docker top` to:
   - View all processes
   - Use different format options
4. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with multiple processes
docker run -d --name process-test nginx

# Spawn some additional processes
docker exec process-test sh -c 'sleep 1000 &'
docker exec process-test sh -c 'tail -f /dev/null &'

# View processes with docker top
docker top process-test
docker top process-test aux

# Clean up
docker stop process-test
docker rm process-test
```
</details>

## Next Steps
After completing these exercises, you should be comfortable with:
- Resource management and limits
- Health checks and monitoring
- Restart policies
- Logging configuration
- Container updates
- Event monitoring

Try combining these concepts to create robust container deployments.
