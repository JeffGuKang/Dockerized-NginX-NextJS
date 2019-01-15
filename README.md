# Dockerized-NginX-NextJS

Make your NextJS app in /nextjs and run in real server or aws with nginx without terrible configuration.

## Docker

Start

```sh
cd docker
docker-compose up -d # d for detached
```

Stop

```sh
docker-compose stop
```

## AWS Beanstalk with Multiple Docker Configuration.

You can run `NextJS Server` through `eb deploy` after basic settings for aws elastic beanstalk with Multiple Docker Configuration.

Do not forget set enough memory for `NextJS`. (256 minimum)

### Dockerrun.aws.json

```json
{
  "AWSEBDockerrunVersion": 2,
  "volumes": [
    {
      "name": "nextjs-volume",
      "host": {
        "sourcePath": "/var/app/current/nextjs"
      }
    },
    {
      "name": "nginx-proxy-conf",
      "host": {
        "sourcePath": "/var/app/current/docker/nginx"
      }
    }
  ],
  "containerDefinitions": [
    {
      "name": "nextjs",
      "image": "node:8.15.0-alpine",
      "essential": true,
      "memory": 512,
      "portMappings": [
        {
          "hostPort": 3000,
          "containerPort": 3000
        }
      ],
      "workingDirectory": "/app",
      "command": ["npm", "run", "production-start"],
      "mountPoints": [
        {
          "sourceVolume": "nextjs-volume",
          "containerPath": "/app"
        }
      ]
    },
    {
      "name": "nginx-proxy",
      "image": "nginx:1.15.8-alpine",
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 80
        }
      ],
      "links": [
        "nextjs"
      ],
      "mountPoints": [
        {
          "sourceVolume": "nginx-proxy-conf",
          "containerPath": "/etc/nginx/conf.d",
          "readOnly": true
        },
        {
          "sourceVolume": "awseb-logs-nginx-proxy",
          "containerPath": "/var/log/nginx"
        }
      ]
    }
  ]
}
```

### Docker Compose

```yml
version: '3.7'
services:
  nextjs:
    # container_name: nextjs
    image: node:8.15.0-alpine
    ports:
      - "3000"
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
    image: nginx:1.15.8-alpine
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./logs/nginx:/var/log/nginx
    links:
      - nextjs
    restart: always
```