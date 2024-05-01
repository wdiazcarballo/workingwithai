Thanks. Please generate the upgraded related files as the solution. Also provide step by step method how to use them.

Here are the upgraded related files and a step-by-step method to use them:

### Upgraded Files:

1. **Dockerfile** (Using multi-stage builds and adding health checks):

```Dockerfile
# Build stage
FROM node:14-alpine as builder
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:14-alpine
WORKDIR /usr/src/app
COPY --from=builder /usr/src/app/dist ./dist
COPY --from=builder /usr/src/app/node_modules ./node_modules
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
  CMD node dist/healthcheck.js
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

2. **docker-compose.yml** (Setting resource limits and adding a monitoring service):

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_URL=mongodb://mongo:27017/CRMdb
      - JWT_SECRET=your_jwt_secret
    depends_on:
      - mongo
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

  mongo:
    image: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  monitoring:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

volumes:
  mongo-data:
```

3. **prometheus.yml** (Configuration file for Prometheus):

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-app'
    static_configs:
      - targets: ['app:3000']
```

### Step-by-Step Method:

1. **Build and Run the Application:**
   - Run `docker-compose up --build` to build and start the application and the MongoDB database.
   - The application will be available at `http://localhost:3000`.

2. **Health Check:**
   - The health check is configured to check the `/health` endpoint every 30 seconds.
   - You can monitor the health status of the application using Docker commands or by checking the logs.

3. **Resource Limits:**
   - The `docker-compose.yml` file sets a CPU limit of 0.5 and a memory limit of 256 MB for the application container.
   - You can adjust these limits based on your requirements.

4. **Monitoring with Prometheus:**
   - Prometheus is configured to scrape metrics from the application at `http://localhost:3000`.
   - Access the Prometheus UI at `http://localhost:9090` to query and visualize metrics.

5. **Logging:**
   - For enhanced logging, you can integrate a logging driver or a logging service like ELK Stack.
   - Configure the logging driver in the `docker-compose.yml` file under the `services > app > logging` section.

By following these steps, you can run and monitor the upgraded Dockerized application in a production-like environment.
