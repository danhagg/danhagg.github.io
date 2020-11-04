---
title: Docker Certified Associate
slug: docker-certified-associate
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
