# nodereactmongo

```
docker images
docker run mongo
docker stop zen_wiles
docker rm zen_wiles
# add "--rm" flag to remove container when stopped the container
docker run --rm mongo
docker run --name mongodb -d --rm mongo
```
Create Dockerfile:
```
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 8000
CMD ["npm", "run", "start"]
```

```
docker build -t todoapp .
docker run --name todo-app todoapp
```

### Connecting Our Containers Through Localhost
```
docker run --name mongodb -d --rm -p 27017:27017 mongo
```
mongodb connection url:
`mongodb://host.docker.internal:27017/todos-app`
```
docker build -t todoapp .
docker run --rm --name todo-app todoapp
```
### Connecting Our Containers Utilizing the Container's IP Address
```
docker container inspect mongodb
# see parameter NetworkSettings.IPAddress (for example: "172.17.0.2")
```
mongodb connection url:
`mongodb://172.17.0.2:27017/todos-app`
```
docker stop todo-app
docker build -t todoapp .
docker run --rm --name todo-app todoapp
```

### Docker Networks
```
docker network ls
docker network create todo-net
docker stop mongodb 
docker stop todo-app
docker run --network todo-net --name mongodb -d --rm mongo
```
mongodb connection url:
`mongodb://mongodb:27017/todos-app`
```
docker build -t todoapp .
docker run --rm --network todo-net --name todo-app todoapp
```

## Dockerizing React App
Dockerfile:
```
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "run", "start"]
```

```
docker build -t frontend .
docker run --name frontend-todo -p 3000:3000 --rm -it frontend
```

Rerun todo-app with port mapping:
```
docker run --rm --network todo-net -p 8000:8000 --name todo-app todoapp
```

### Optimizing Our Workflow with Volumes
```
docker run --name frontend-todo -p 3000:3000 -v $(pwd)/src:/app/src -v $(pwd)/public:/app/public --rm -it frontend

docker run --name mongodb --network todo-net -v data:/data/db mongo

docker run --name todo-app --rm -p 8000:8000 --network todo-net -v $(pwd)/src:/app/src todoapp
```

## Use docker-compose
### Configuration for mongodb container
Create docker-compose.yaml file:
```
version: "3.8"
services:
  mongodb:
    image: mongo
    volumes:
      - mongodbdata:/data/db
    env_file:
      - ./mongo/.env
    container_name: mongodb

volumes:
  mongodbdata:
```
Create ./mongo/.env file:
```
MONGO_INITDB_ROOT_USERNAME=mymongoadmin
MONGO_INITDB_ROOT_PASSWORD=mysecretpassword
```
Command to start container: `docker-compose up`
### Configuration for mongo + backend
Update docker-compose.yaml file:
```
version: "3.8"
services:
  mongodb:
    image: mongo
    volumes:
      - mongodbdata:/data/db
    env_file:
      - ./mongo/.env
    container_name: mongodb
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    volumes:
      - ./backend/src:/app/src
    ports:
      - 8000:8000
    env_file:
      - ./backend/.env
    depends_on: 
      - mongodb
    container_name: backend

volumes:
  mongodbdata:
```
Create ./backend/.env file:
```
MONGODB_USERNAME=mymongoadmin
MONGODB_PASSWORD=mysecretpassword
```
Update connection url in ./backend/src/index.js
``mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/todos-app?authSource=admin``

Send JSON to POST localhost:8000/todos to check the backend:
```
{
    "name": "another todo"
}
```
### Configuration for mongo + backend + frontend
Update docker-compose.yaml file:
```
version: "3.8"
services:
  mongodb:
    image: mongo
    volumes:
      - mongodbdata:/data/db
    env_file:
      - ./mongo/.env
    container_name: mongodb
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    volumes:
      - ./backend/src:/app/src
    ports:
      - 8000:8000
    env_file:
      - ./backend/.env
    depends_on: 
      - mongodb
    container_name: backend
  frontend:
    build: ./frontend
    volumes:
      - ./frontend/src:/app/src
      - ./frontend/public:/app/public
    ports:
      - 3000:3000
    stdin_open: true
    tty: true
    depends_on:
      - backend

volumes:
  mongodbdata:
```