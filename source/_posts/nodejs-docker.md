---
title: Running Node.js Apps with Docker/Docker Compose
date: 2018-08-21 22:08:55
categories:
- DevOps
tags:
- Docker
- Nodejs
---
# Checklist of dockerizing an application
## Choose a base image
Consider to use Alpine images.

## Install the necessary packages
* You need to write `apt-get update` and `apt-get install` on the same line (same if you are using apk on Alpine).
* Double check if you are installing ONLY what you really need (assuming you will run the container on production).

## Add your custom files
* Understand the different between COPY and ADD
* Follow File System conventions on where to place your files
* Check the attributes of the files you are adding. If you need execution permission, there is no need to add a new layer on your image (`RUN chmod +x`). Just fix the original attributes on your code repository.

## Define which user will (or can) run your container
* Without any other option provided, processes in containers will execute as root (unless a different uid was supplied in the Dockerfile).
* You only need to run your container with a specific (fixed ID) user if your application need access to the user or group tables (`/etc/passwd` or `/etc/group`).
* Avoid running your container as root as much as possible.
* Note some applications require you to run them with specific ids (e.g. Elastic Search with uid:gid = 1000:1000).

## Define the exposed ports

## Define the entrypoint
Create a `docker-entrypoint.sh` script where you can hook things like configuration using environment variables.

Examples
* [elasticsearch](https://github.com/elastic/elasticsearch-docker/tree/master/build/elasticsearch/bin)
* [postgres](https://github.com/docker-library/postgres/tree/de8ba87d50de466a1e05e111927d2bc30c2db36d/10)

## Define a Configuration method
Every application requires some kind of parametrization. There are basically two paths you can follow:
* Use an application specific configuration file: them you will need to document the format, fields, location and so on (not good if you have a complex environment, with applications spanning different technologies).
* Use (operating system) Environment variables: Simple and efficient.

## Externalize your data
The golden rule is: do not save any persistent data inside the container.

The container file system is supposed and intended to be temporary, ephemeral. So any user generated content, data files, process output should be saved either on a mounted volume or on a bind mounts (that is, on a folder on the Base OS linked inside the container).

## Make sure you handle the logs as well
If you are creating a new app and want it to stick to docker conventions, no logs files should be written at all. The application should use stdout and stderr as an event stream.

Docker will automatically capture everything you are sending to stdout and make it available through “docker logs” command.

## Rotate logs and other append only files
If your application is writing log files or appending any files that can grow indefinitely, you need to worry about file rotation.

This is critical for you to prevent the server running out of space, apply data retention policies (which is critical when it comes to GDPR and other data regulations).

[Example of log rotation using logrotate](https://www.aerospike.com/docs/operations/configure/log/logrotate.html)

# Lean image with Docker multi-stage build

Docker 17.05 extends Dockerfile syntax to support new multi-stage build, by extending two commands: FROM and COPY.

The multi-stage build allows using multiple FROM commands in the same Dockerfile. The last FROM command produces the final Docker image, all other images are intermediate images (no final Docker image is produced, but all layers are cached).

The FROM syntax also supports AS keyword. Use AS keyword to give the current image a logical name and reference to it later by this name.

To copy files from intermediate images use `COPY --from=<image_AS_name|image_number>`, where number starts from 0 (but better to use logical name through AS keyword).

```dockerfile
#
# ---- Base Node ----
FROM alpine:3.5 AS base
# install node
RUN apk add --no-cache nodejs-current tini
# set working directory
WORKDIR /root/chat
# Set tini as entrypoint
ENTRYPOINT ["/sbin/tini", "--"]
# copy project file
COPY package.json .

#
# ---- Dependencies ----
FROM base AS dependencies
# install node packages
RUN npm set progress=false && npm config set depth 0
RUN npm install --only=production 
# copy production node_modules aside
RUN cp -R node_modules prod_node_modules
# install ALL node_modules, including 'devDependencies'
RUN npm install

#
# ---- Test ----
# run linters, setup and tests
FROM dependencies AS test
COPY . .
RUN  npm run lint && npm run setup && npm run test

#
# ---- Release ----
FROM base AS release
# copy production node_modules
COPY --from=dependencies /root/chat/prod_node_modules ./node_modules
# copy app sources
COPY . .
# expose port and define CMD
EXPOSE 5000
CMD npm run start
```

# Docker Compose for NodeJS Development

An important concept to understand is that Docker Compose spans “buildtime” and “runtime”. Up until now, we have been building images using docker build ., which is “buildtime.” This is when our containers are actually built. We can think of “runtime” as what happens once our containers are built and being used.

Compose triggers “buildtime” — instructing our images and containers to build — but it also populates data used at “runtime,” such as env vars and volumes. This is important to be clear on. For instance, when we add things like volumes and command, they will override the same things that may have been set up via the Dockerfile at “buildtime.”

# How to debug a Node.js application in a Docker container

# References
* [How to dockerize any application](https://hackernoon.com/how-to-dockerize-any-application-b60ad00e76da)
* [理解Docker的多阶段镜像构建](https://tonybai.com/2017/11/11/multi-stage-image-build-in-docker/)