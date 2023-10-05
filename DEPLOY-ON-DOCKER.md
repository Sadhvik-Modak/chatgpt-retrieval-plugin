# Run Gpt Plugin with Redis datastore on Docker

1. Clone the repo `https://github.com/openai/chatgpt-retrieval-plugin.git`

2. cd into /path/to/chatgpt-retrieval-plugin/

3. Before running this, we need to below keys
    1. BEARER_TOKEN
        * We can generate a random bearer token using [jwt.io](jwt.io).
        * Example value: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`

    2. OPENAI_API_KEY
        * This can be generated from [https://beta.openai.com/account/api-keys](https://beta.openai.com/account/api-keys)
        * Example Value: `sk-3cQ1Gjjgnajngajkl2giop2e1dk1R`

4. Create a file with name `docker-compose.yml` and add the below content

```
version: "3.9"

services:
  redis:
    image: redis/redis-stack-server:latest
    ports:
      - "6379:6379"
    volumes:
        - redis_data:/data
    # Add a Docker volume bind here for your specific path where the ssd disk is attached to the host machine
    # volumes:
    #   - /path/on/host:/path/in/container
    healthcheck:
      test: ["CMD", "redis-cli", "-h", "localhost", "-p", "6379", "ping"]
      interval: 2s
      timeout: 1m30s
      retries: 5
      start_period: 5s

  gpt_api:
    build:
      context: .
      dockerfile: Dockerfile  
    ports:
      - "8089:8089"
    environment:
      - PORT=8089
      - DATASTORE=redis
      - BEARER_TOKEN=<paste bearer token>
      - OPENAI_API_KEY=<paste openapi key>
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - REDIS_PASSWORD=
      - REDIS_INDEX_NAME=index
      - REDIS_DOC_PREFIX=doc
      - REDIS_DISTANCE_METRIC=COSINE
      - REDIS_INDEX_TYPE=FLAT
    # Add a Docker volume bind here for your specific path where the ssd disk is attached to the host machine
    # volumes:
    #   - /path/on/host:/path/in/container
    depends_on:
      redis:
        condition: service_healthy

volumes:
  redis_data:
```

5. Paste the values of *BEARER_TOKEN* and *OPENAI_API_KEY* in the above docker-compose.yml

6. Now, run the command `docker-compose up -d`. This will first the run the redis container on port 389. Once the redis container is healthy, docker image for gpt plugin will be built (one time process) and will start on PORT `8089`. For the first time, it took approximately 7min to build the image and start both redis and gpt plugin.

7. We can access the plugin API at [http://localhost:8089/docs](http://localhost:8089/docs).

8. To stop the services, run the command `docker-compose down`. To stop and delete the volumes, run the command `docker-compose down -v`.
