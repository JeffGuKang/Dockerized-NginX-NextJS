# Dockerized-NginX-NextJS

Make your NextJS app in /nextjs and run in real server through nginx without terrible configuration.

Start

```sh
cd docker
docker-compose up -d # d for detached
```

Stop

```sh
docker-compose stop
```

I am working to apply letsencrypt.

## Docker Compose

```yml
version: '3.7'
services:
  nextjs:
    # container_name: nextjs
    image: node:8.15.0
    build: ../nextjs
    ports:
      - "3000:3000"
    volumes:
      - ../nextjs:/app/nextjs
      - ./logs/nextjs/npm:/root/.npm/_logs
    working_dir: /app/nextjs
    command: >
      /bin/sh -c "npm install -f&&
        npm run build &&
        npm run start"
  nginx:
    # container_name: nginx
    image: nginx:latest
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./logs/nginx:/var/log/nginx
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
    links:
      - nextjs
restart: always
```