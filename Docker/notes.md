# Docker

## Container

What is a container?

Container is a way to package an application with all the necessary dependencies and configuration. And that package is portable and can be easily shared and moved around.


**Container VS Image**:
##TODO
Container is a running environment for image.


## Docker Architecture 

When we install docker we install **docker engine** which comes with 3 parts:

- Docker Server - Which is responsible for pulling images, managing images & containers, running and stopping containers.
- Docker API - This is an API for interacting with the docker server.
- Docker CLI - Which is a client to execute docker commands against the server.

Docker Server also have some important parts:

- Docker Runtime - Which is responsible for pulling images, managing container lifecycle( starting and stopping containers)
- Volumes - Which is responsible for persisting data in the containers.
- Network - configure network for container communication.
- Build images - build our own docker images, to build artifact that we can then run as containers.

**If we need only a container runtime then we have *containerd*, *cri-o*, and to build an image we have *buildah*.**


## Docker Network

Docker creates it isolated docker network. When we deploy two container in same docker network, they can communicate just by using their container name.

`docker network ls`

create docker network: `docker network create mongo-network`

Running container in our network: `docker run --net mongo-network mongo`


```yaml
volumes:
    mongo-data:
        driver: local
```

## Pushing images to nexus

Create a repository for docker. To push to the repository we need to first login to nexus using a user account that has access to docker repository.

Creating a new role and give privilege `nx-repository-view-docker-docker-hosted-*` and then add this role to a user.

Give the repo a http port so that docker can reach access the repo.

After this we need to config in nexus is Realms. When we login do docker login we get a token of authentication from nexus docker repository for client and that token will be stored on our local machine in `~/.docker/config.json` file. This file contains all the authentication that we have made all the different docker repositories.

Lastly we need to configure docker to allow to request to insecure repository, by default docker only allow client requests (docker login , pull, push) to go to https endpoints of a docker registry. This is done in linux by editing `/etc/docker/daemon.json` file  and adding
```json
{
    "insecure-registries": ['url']
}
```

`docker login ip:port (192.168.56.118:8080)`

**Pushing image**:
`docker build -t  my-app:1.0`

`docker tag my-app:1.0 192.168.56.118:8080/my-app:1.0`

`docker push 192.168.56.118:8080/my-app:1.0`

retrieve info using REST API: `curl -u darq:1324 -X GET 'http://192.168.56.118:8081/service/rest/v1/components?repository=docker-hosted'`

`docker volume create --name nexus-data`

### Best practices

**Optimize Caching Image layers**:
In dockerfile each layer creates an image layer. We can also see the image layer of our image we built using `docker history myapp:1.0`.

What does caching an Image Layer means?

Docker caches each layer, saved on local filesystem. If we build our image and dockerfile has not changed then docker will re-use the cached layer to build the image. It makes building the image faster.

```dockerfile
FROM node:17.0.1-alpine

WORKDIR /app

COPY myapp /app

RUN npm install --production

CMD ["node", "src/index.js"]
-----
# optimized 
FROM node:17.0.1-alpine

WORKDIR /app

COPY package.json package-lock.json .

RUN npm install --production

COPY myapp /app

CMD ["node", "src/index.js"]
```

**Order Dockerfile commands from least to most frequently changing**.

## using built artifacts 

Contents, that you need for building the image and don't need them in the final image to run the app. e.g. package.json is needed to install packages but after installing it is no longer required or JDK is required to compile the source code and not needed to run the Java application or tools like maven or gradle to build the application but are not needed to run Java application.

So how do we separate **build stage** from **runtime stage**, or how do we exclude the build dependencies from the final image?

For that we can use **Multi-Stage Builds**. Multi-Stage Build feature allows us to use multi temporary images during the build process but keep only the latest/last image as final artifact.

```Dockerfile
# Build Stage
FROM maven AS build  # you can name your stages with "AS <name>"
WORKDIR /app
COPY myapp /app
RUN mvn package
# here we build java app

# RUN Stage
FROM tomcat  # each FROM instruction starts a new build stage
COPY --from=build /app/target/file.war  /usr/local/tomcat/ # we use file generated in previous build stage and copy them in the final image.

# all the files and tools we use are discarded once we build the final image.
```

Which Os user will be used to start the application?

If Dockerfile does not specify an user then it uses root user. It is not a good practice to use container with root privileges and it is also a bad security practice. 

This creates a security issue because when container starts on the host, it could potentially have root access on the docker host and will make it easier for an attacker to have privilege escalation on the host.

Best practice is to create a dedicated user and group in docker image to run the container.

```Dockerfile
# create group and user
RUN groupadd -r tom && useradd -g tom tom

# set ownership and permissions 
RUN chown -R tom:tom /app

# switch to user
USER tom

CMD node index.js
```
**Some base images have a generic user bundled in like for node image it has node user.**

### How to validate the built image for security vulnerabilities?

Scan you image for security vulnerabilities using `docker scan myapp:1.0`. We need to login into docker hub to scan our images. Docker uses Snyk services for vulnerability scan.
### Docker login to nexus

