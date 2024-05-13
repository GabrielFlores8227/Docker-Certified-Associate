## Chapter 1 - Image Creation, Management, and Registry

### Docker Images

A Docker image is a file containing the code and components needed to run software in a container.
Containers and images use a layered file system. Each layer contains only the differences from the 
previous layer. The image consists of one or more read-only layers, while the container adds one 
additionall writable layer.

The layered file system allows multiple images and containers to share the same layers. This 
results in:

  - Smaller overall storage footprint.

  - Faster image transfers.

  - Faster image build.

```Dockerfile
docker image pull IMAGE[:TAG]
```

Download an image from a remote registry to the local machine.

```Dockerfile
docker image history IMAGE[:TAG]
```

List the layers used to build an image.

### Dockerfiles

If you want to create your own images. you can do so with a Dockerfile. A Dockerfile is a set of instructions
which are used to construct a Docker image. These instructions are called directives.

- `ADD`: Add local or remote files and directories.
  
- `ARG`: Use build-time variables.
  
- `CMD`: Specify default commands.
  
- `COPY`: Copy files and directories.
  
- `ENTRYPOINT`: Specify default executable.
  
- `ENV`: Set environment variables.
  
- `EXPOSE`: Describe which ports your application is listening on.
  
- `FROM`: Create a new build stage from a base image.
  
- `HEALTHCHECK`: Check a container's health on startup.
  
- `LABEL`: Add metadata to an image.
  
- `MAINTAINER`: Specify the author of an image.
  
- `ONBUILD`: Specify instructions for when the image is used in a build.
  
- `RUN`: Execute build commands.
  
- `SHELL`: Set the default shell of an image.
  
- `STOPSIGNAL`: Specify the system call signal for exiting a container.
  
- `USER`: Set user and group ID.
  
- `VOLUME`: Create volume mounts.
  
- `WORKDIR`: Change working directory.

### Efficient Docker Images

When working with Docker in real world, it is important to create Docker images that are as
efficient as possible. This means that they are as small as possible and result in ephemeral 
containers that can be started, stopped, and destroyed easily.

Docker supports the ability to perform multi-stage builds. Multi-stage builds have more than
one `FROM` diirective in the Dockerfile, with each `FROM` directive starting a new stage.
Each stage begins a completely new set of file system layers, allowing you to
selectively copy only the files you need from previous layers.
