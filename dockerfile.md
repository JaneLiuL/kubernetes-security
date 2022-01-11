

推荐阅读

- [Docker best practices by Google](https://cloud.google.com/blog/products/containers-kubernetes/7-best-practices-for-building-containers)
- [Docker tips](https://learnk8s.io/blog/smaller-docker-images)

题目

Given a Dockerfile, analyse it and update it based on security best practices.

```
vi ~/Dockerfile

FROM ubuntu:latest

ENV CI=true

RUN apt get update
RUN apt get install -y wget
RUN apt get install -y curl

USER root

WORKDIR /code
COPY package.json package-lock.json /code/
RUN npm ci
COPY src /code/src

CMD [ "npm", "start" ]
```

答案：把`USER root`注释即可