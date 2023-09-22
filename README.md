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