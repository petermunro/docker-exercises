# Docker Compose Lab Exercises

__Objective__: Learn how to use Docker Compose through progressive hands-on exercises.

## Exercise 1: Your First Docker Compose File
Learn to create and use a basic Docker Compose file.

1. Create a directory called `compose-test` and navigate into it
2. Create a `docker-compose.yml` file with a single nginx service
3. Run the compose stack
4. List the running containers
5. Stop and remove the stack

<details>
<summary>Reveal Solution</summary>

```bash
# Create directory and navigate into it
mkdir compose-test
cd compose-test

# Create docker-compose.yml with content:
```

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

```bash
# Run the stack
docker compose up -d

# List running containers
docker compose ps

# Stop and remove the stack
docker compose down
```
</details>

## Exercise 2: Multiple Services
Learn to define and manage multiple services.

1. Modify your `docker-compose.yml` to add a Redis service alongside nginx
2. Make sure nginx is exposed on port 8080
3. Start the stack and verify both services are running
4. View the logs of both services
5. Clean up when done

<details>
<summary>Reveal Solution</summary>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
  redis:
    image: redis:alpine
```

```bash
# Start the stack
docker compose up -d

# View running services
docker compose ps

# View logs
docker compose logs

# Follow logs of a specific service
docker compose logs -f web

# Clean up
docker compose down
```
</details>

## Exercise 3: Environment Variables
Learn to work with environment variables in Docker Compose.

1. Create a `.env` file with the following content:
   ```
   NGINX_PORT=8080
   POSTGRES_PASSWORD=mysecretpassword
   ```
2. Create a Docker Compose file using these variables for a Postgres database and nginx
3. Start the stack and verify the variables are being used
4. Use `docker compose config` to see the resolved configuration

<details>
<summary>Reveal Solution</summary>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "${NGINX_PORT}:80"
  db:
    image: postgres:alpine
    environment:
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
```

```bash
# View resolved configuration
docker compose config

# Start the stack
docker compose up -d

# Verify services
docker compose ps

# Clean up
docker compose down
```
</details>

## Exercise 4: Building from Dockerfile
Learn to build and run services from Dockerfiles.

1. Create a simple Python Flask application:
   - Create `app.py` with a "Hello World" endpoint
   - Create `requirements.txt` with Flask as a dependency
   - Create a `Dockerfile` to build it
2. Create a Docker Compose file that builds and runs this application
3. Build and start the stack
4. Verify you can access the application
5. Make a change to the application and rebuild

<details>
<summary>Reveal Solution</summary>

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Flask!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```
# requirements.txt
flask
```

```dockerfile
# Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "5000:5000"
```

```bash
# Build and start
docker compose up --build -d

# Verify it's running
curl http://localhost:5000

# Clean up
docker compose down
```
</details>

## Exercise 5: Volumes and Persistence
Learn to work with volumes in Docker Compose.

1. Create a Docker Compose file with a PostgreSQL service using a named volume
2. Add a service for pgAdmin (database management tool)
3. Start the stack and verify data persistence:
   - Create a database using pgAdmin
   - Stop and remove containers
   - Start again and verify the database still exists
4. Clean up, including volumes

<details>
<summary>Reveal Solution</summary>

```yaml
services:
  db:
    image: postgres:alpine
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: user@domain.com
      PGADMIN_DEFAULT_PASSWORD: example
    ports:
      - "8080:80"

volumes:
  postgres_data:
```

```bash
# Start services
docker compose up -d

# Verify services
docker compose ps

# Stop services but keep volumes
docker compose down

# Remove everything including volumes
docker compose down -v
```
</details>

## Exercise 6: Networks and Service Discovery
Learn about Docker Compose networking.

1. Create a compose file with three services:
   - A Flask API service
   - A Redis cache
   - An nginx reverse proxy
2. Configure nginx to proxy requests to the Flask API
3. Make the Flask API use Redis for caching
4. Verify services can communicate using their service names
5. Test the complete setup

<details>
<summary>Reveal Solution</summary>

```python
# app.py
from flask import Flask
import redis
import json

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = cache.incr('hits')
    return f'Hello! I have been seen {count} times.\n'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream flask_app {
        server api:5000;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://flask_app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

```yaml
# docker-compose.yml
services:
  api:
    build: .
    depends_on:
      - redis

  redis:
    image: redis:alpine

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "8080:80"
    depends_on:
      - api
```

```bash
# Start the stack
docker compose up -d

# Test the endpoint
curl http://localhost:8080

# View logs
docker compose logs

# Clean up
docker compose down
```
</details>

## Exercise 7: Health Checks
Learn to implement and use health checks.

1. Add health checks to your services:
   - For nginx: test the HTTP endpoint
   - For Redis: use the Redis CLI ping
   - For your application: add a /health endpoint
2. Use `depends_on` with condition: service_healthy
3. Monitor the health status of your services

<details>
<summary>Reveal Solution</summary>

```yaml
services:
  api:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    depends_on:
      redis:
        condition: service_healthy

  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 1s
      timeout: 3s
      retries: 30

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3
    depends_on:
      api:
        condition: service_healthy
```

```bash
# Start services
docker compose up -d

# Check service health
docker compose ps

# View health check logs
docker compose events

# Clean up
docker compose down
```
</details>

## Exercise 8: Development vs Production
Learn to manage different environments with Docker Compose.

1. Create three compose files:
   - `docker-compose.yml` (base configuration)
   - `docker-compose.override.yml` (development overrides)
   - `docker-compose.prod.yml` (production overrides)
2. Add development-specific settings (volumes, debug mode)
3. Add production-specific settings (replicas, resource limits)
4. Test running in both environments

<details>
<summary>Reveal Solution</summary>

```yaml
# docker-compose.yml (base)
services:
  api:
    build: .
    image: myapp:latest

  redis:
    image: redis:alpine
```

```yaml
# docker-compose.override.yml (dev)
services:
  api:
    volumes:
      - .:/app
    environment:
      - DEBUG=1
    ports:
      - "5000:5000"
```

```yaml
# docker-compose.prod.yml
services:
  api:
    environment:
      - DEBUG=0
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

```bash
# Run in development (default)
docker compose up -d

# Run in production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Clean up
docker compose down
```
</details>

## Next Steps
After completing these exercises, you should be comfortable with:
- Creating and managing Docker Compose files
- Working with multiple services
- Using environment variables
- Building services from Dockerfiles
- Managing volumes and persistence
- Understanding Docker Compose networking
- Implementing health checks
- Managing different environments

Try creating a more complex application stack using these concepts! 