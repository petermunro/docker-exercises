# Docker Networking Lab Exercises

__Objective__: Learn how to work with Docker networks through hands-on exercises that demonstrate key networking concepts.

## Exercise 1: Exploring Default Networks
Learn about Docker's default network types.

1. List all networks on your system
2. Inspect the default bridge network
3. Run an nginx container and check which network it's connected to
4. Inspect the container to find its IP address
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# List all networks
docker network ls

# Inspect bridge network
docker network inspect bridge

# Run container
docker run -d --name web1 nginx

# Check container's network
docker inspect web1 | grep -A 10 "Networks"

# Clean up
docker stop web1
docker rm web1
```
</details>

## Exercise 1B: Default Bridge Network Limitations
Learn about the limitations of the default bridge network.

> Using ping
  

1. Run two nginx containers on the default bridge network
2. Try to ping between containers using container names (this will fail)
3. Find the IP address of the second container
4. Ping using IP address (this will work)
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run two containers on default bridge network
docker run -d --name bridge1 nginx
docker run -d --name bridge2 nginx

# Install ping utility in first container
docker exec -it bridge1 apt-get update
docker exec -it bridge1 apt-get install -y iputils-ping

# Try to ping by name (this will fail)
docker exec -it bridge1 ping bridge2

# Get IP address of bridge2
docker inspect bridge2 | grep IPAddress

# Ping using IP address (this will work)
docker exec -it bridge1 ping <IP_ADDRESS_OF_BRIDGE2>

# Clean up
docker stop bridge1 bridge2
docker rm bridge1 bridge2
```
</details>

## Exercise 2: Creating Custom Networks
Learn to create and manage custom bridge networks.

1. Create a custom bridge network called "my-network"
2. List networks to verify creation
3. Inspect the new network
4. Create another network with a custom subnet (172.20.0.0/16)
5. Remove the second network
6. List networks to verify removal

<details>
<summary>Reveal Solution</summary>

```bash
# Create custom network
docker network create my-network

# List networks
docker network ls

# Inspect network
docker network inspect my-network

# Create network with custom subnet
docker network create --subnet=172.20.0.0/16 custom-subnet

# Remove network
docker network rm custom-subnet

# Verify removal
docker network ls
```
</details>

## Exercise 3: Container Communication
Learn how containers communicate on the same network.

1. Create a new bridge network called "app-network"
2. Run two nginx containers on this network
   - Name them "web1" and "web2"
3. Connect to web1 using `docker exec`
4. Try to ping web2 using its container name
5. Inspect the network to see both containers
6. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Create network
docker network create app-network

# Run containers
docker run -d --name web1 --network app-network nginx
docker run -d --name web2 --network app-network nginx

# Connect to web1 and ping web2
docker exec -it web1 apt-get update
docker exec -it web1 apt-get install -y iputils-ping
docker exec -it web1 ping web2

# Inspect network
docker network inspect app-network

# Clean up
docker stop web1 web2
docker rm web1 web2
docker network rm app-network
```
</details>

## Exercise 4: Network Isolation
Learn about network isolation between different networks.

1. Create two networks: "network1" and "network2"
2. Run a container in network1
3. Run another container in network2
4. Try to ping between containers
5. Connect one container to both networks
6. Test connectivity again
7. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Create networks
docker network create network1
docker network create network2

# Run containers
docker run -d --name container1 --network network1 nginx
docker run -d --name container2 --network network2 nginx

# Try to ping (will fail)
docker exec -it container1 apt-get update
docker exec -it container1 apt-get install -y iputils-ping
docker exec -it container1 ping container2

# Connect container1 to network2
docker network connect network2 container1

# Try ping again
docker exec -it container1 ping container2

# Clean up
docker stop container1 container2
docker rm container1 container2
docker network rm network1 network2
```
</details>

## Exercise 5: Port Mapping
Learn about exposing container ports to the host.

1. Run an nginx container without port mapping
2. Try to access it from your browser (it won't work)
3. Run another nginx container with port 80 mapped to 8080
4. Access it through your browser at localhost:8080
5. Run a third container mapping multiple ports
6. List containers to see port mappings
7. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container without port mapping
docker run -d --name web1 nginx

# Run container with port mapping
docker run -d --name web2 -p 8080:80 nginx

# Run container with multiple port mappings
docker run -d --name web3 -p 8081:80 -p 8443:443 nginx

# List containers
docker ps

# Clean up
docker stop web1 web2 web3
docker rm web1 web2 web3
```
</details>

## Exercise 6: Host Network Mode
Learn about host network mode.

1. Run an nginx container using host network mode
2. Inspect the container's network settings
3. Compare the network settings with a container on bridge network
4. Try to access nginx through localhost
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Run container with host network
docker run -d --name host-nginx --network host nginx

# Inspect container
docker inspect host-nginx

# Run and inspect bridge container
docker run -d --name bridge-nginx nginx
docker inspect bridge-nginx

# Clean up
docker stop host-nginx bridge-nginx
docker rm host-nginx bridge-nginx
```
</details>

## Exercise 7: DNS Resolution
Learn about Docker's built-in DNS.

1. Create a custom network
2. Run a container named "db" running MongoDB
3. Run another container named "web" running nginx
4. Connect to the web container
5. Try to ping "db" using the container name
6. Inspect DNS settings inside the container
7. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Create network
docker network create app-net

# Run containers
docker run -d --name db --network app-net mongo
docker run -d --name web --network app-net nginx

# Test DNS resolution
docker exec -it web apt-get update
docker exec -it web apt-get install -y iputils-ping
docker exec -it web ping db

# Check DNS settings
docker exec -it web cat /etc/resolv.conf

# Clean up
docker stop db web
docker rm db web
docker network rm app-net
```
</details>

## Exercise 8: Network Troubleshooting
Learn basic network troubleshooting techniques.

1. Create a network and run two containers
2. Use various commands to investigate the network:
   - Inspect network
   - Check container IP addresses
   - View network interfaces inside container
   - Test connectivity between containers
3. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Create network and containers
docker network create trouble-net
docker run -d --name cont1 --network trouble-net nginx
docker run -d --name cont2 --network trouble-net nginx

# Inspect network
docker network inspect trouble-net

# Check container IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cont1

# View network interfaces
docker exec -it cont1 apt-get update
docker exec -it cont1 apt-get install -y iproute2
docker exec -it cont1 ip addr

# Test connectivity
docker exec -it cont1 apt-get install -y iputils-ping
docker exec -it cont1 ping cont2

# Clean up
docker stop cont1 cont2
docker rm cont1 cont2
docker network rm trouble-net
```
</details>

## Exercise 9: Internal Networks
Learn about internal networks without external access.

1. Create an internal network
2. Run containers on the internal network
    - Use the alpine image, as it has `ping` installed
    - Run them as daemon processes, using the command `sleep 600` to sleep for 10 minutes
3. Try to access the internet from the containers
4. Verify containers can communicate with each other
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Create internal network
docker network create --internal internal-net

# Run containers
docker run -d --rm --name internal1 --network internal-net alpine sleep 600
docker run -d --rm --name internal2 --network internal-net alpine sleep 600

# Try internet access from within containers (should fail)
docker exec -it internal1 ping google.com

# Test internal communication
docker exec -it internal1 ping internal2

# Clean up
docker stop internal1 internal2
docker rm internal1 internal2
docker network rm internal-net
```
</details>

## Exercise 10: Multi-Network Setup
Create a multi-tier application network setup.

1. Create two networks: "frontend-net" and "backend-net"
2. Run a web container connected to frontend-net
3. Run a database container (use the "mongo" image)connected to backend-net
4. Connect the web container to backend-net
5. Verify connectivity between all containers
6. Clean up when done

<details>
<summary>Reveal Solution</summary>

```bash
# Create networks
docker network create frontend-net
docker network create backend-net

# Run containers
docker run -d --name webapp --network frontend-net nginx
docker run -d --name database --network backend-net mongo

# Connect webapp to backend
docker network connect backend-net webapp

# Test connectivity
docker exec -it webapp apt-get update
docker exec -it webapp apt-get install -y iputils-ping
docker exec -it webapp ping database

# Clean up
docker stop webapp database
docker rm webapp database
docker network rm frontend-net backend-net
```
</details>

## Next Steps
After completing these exercises, you should be comfortable with:
- Creating and managing Docker networks
- Container network communication
- Network isolation and security
- Port mapping and exposure
- DNS resolution and service discovery
- Network troubleshooting
- Multi-tier network architecture

Try creating more complex network setups or combining these concepts with other Docker features! 