---
title: Docker Commands
slug: docker-commands
last_modified_at: 2019-11-01T16:20:02-05:00
category: Certifications
tags: Docker
Authors: Daniel Haggerty
---

### Docker

```bash
docker run dans-container
```

Is equivalent too

```bash
docker create dans-container
docker start dans-container
```

If:

```bash
docker run busybox ping google.com
```

To view that container still running:

```bash
docker ps
docker ps --all
```

To stop container: `docker stop` tries to cleanup, `docker kill` is more brutal.

```bash
docker stop containerid
docker kill containerid
```

To get a shell inside a running container:

```bash
docker run redis
# Then in a second Docker Terminal
docker exec -it containerid sh
```

To get straight into the shell at container startup:

```bash
docker run -it busybox sh
```


Stop and remove all containers. 

```bash
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

# NODE APP

**index.js**

```js
const express = require('express');

const app = express();

app.get('/', (req, res) => {
    res.send('Docker deployment of minimal node app');
});

app.listen(8080, () => {
    console.log('listening on port 8080');
});
```

**package.json**

```json
{
    "dependencies":{
        "express": "*"
    },
    "scripts": {
        "start": " node index.js"
    }
}
```

**Dockerfile**

```dockerfile
# Specify a base image, node alpine is stripped down node.
FROM node:alpine

# put app into a dedicated folder in container
WORKDIR /usr/app

# package.json has to be available before npm install
COPY ./package.json ./

# Install some dependencies
RUN npm install

# copy rest of files after npm install
COPY ./* ./

# Default command
CMD ["npm", "start"]
```

Any incoming request made to my port 8080 that request is mapped to the container.

Port forwarding is a runtime constraint `-p 8080:8080` and not required in Dockerfile

```bash
docker build -t danhagg/node_app:latest .

docker run -p 8080:8080 danhagg/node_app
```

Visit http://192.168.99.100:8080/ and view the app.


Get a shell inside the running docker containing. Note that we enter into WORKDIR delcared in Dockerfile.
```bash
docker exec -it b24326768f29 sh
/usr/app #
```

# Use docker-compose for multiple containers

**index.js**

```js
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient({
    host: 'redis-server',
    port: 6379
});
client.set('visits', 0);

app.get('/', (req, res) => {
    client.get('visits', (err, visits) => {
        res.send("number of visits is " + visits);
        client.set('visits', parseInt(visits) + 1)
    });
});

app.listen(8081, () => {
    console.log('Listening on port 8081');
});
```

**dockerfile**

```
FROM node:alpine

WORKDIR '/app'

COPY package.json .

RUN npm install

COPY . .

CMD ["npm", "start"]
```

**docker-compose.yml**

```yml
version: '3'
services:
  redis-server:
    image: 'redis'
  node-app:
    restart: always
    build: .
    ports:
      - "4001:8081"
```

The commands are different if we have a docker-compose.yml.

```bash
# To build and run
docker-compose up --build

# To run already built container
docker-compose up
```

More docker-cmpose commands

```bash
# Launch in background
docker-compose up -d

#stop&remove containers started above
docker-compose down

# View running containers. 
# Must be in same folder as docker-compose.yml
docker-compose ps
```