## Chapter 1 - Introduction

### Namespaces

Namespaces are Linux technologies that allows processes to be isolated in terms of the resources that they see. They
can be used to prevent different processes from interfering or interacting with one another.

Docker uses namespaces to isolate containers. This technology allows containers to operate independently. and securely.

Docker uses namespaces as the following to isolate resources for containers:

- `pid`: Process isolation
- `net`: Network interfaces
- `ipc`: Inter-process communication
- `mnt`: Filesystem mounts
- `uts`: Kernel and version identifiers
- `user namespaces`: Requires special configuration. Allows container processes to run as root inside the container while mapping that user to an unprivileged user on the host.

### Control Groups

Docker Engine on Linux also relies on another technology called control groups (cgroups). A cgroup limits an application
to a specific set of resources. Control groups allow Docker Engine to share available hardware resources to containers and
optionally enforce limits and constraints. For example, you can limit the memory available to a specific container.

## Chapter 2 - Image Creation, Management, and Registry

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

Here is an example of an efficient Dockerfile utilizing multi-stage builds:

```Dockerfile
# Use the official Golang image as the base image for the compiler stage
FROM golang:latest AS compiler

# Set the working directory within the container for the compiler stage
WORKDIR /compiler

# Copy the source code (helloworld.go) to the working directory
COPY ./helloworld.go .

# Initialize Go module to manage dependencies
RUN go mod init helloworld

# Build the Go application for Linux (GOOS=linux) with CGO disabled (-installsuffix cgo)
# and output the binary as 'helloworld' in the working directory
RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .

# Start a new stage using the lightweight Alpine Linux as the base image
FROM alpine:latest

# Set the working directory within the container for the runtime stage
WORKDIR /root

# Copy the built executable 'helloworld' from the compiler stage to the working directory of the runtime stage
COPY --from=compiler /compiler/helloworld ./

# Define the command to run the executable when the container starts
CMD ["./helloworld"]
```

### Managing Images

Download an image from a remote registry to the local machine:

```bash
docker image pull <IMAGE>
```

List the layers used to build an image:

```bash
docker image history <IMAGE>
```

List images:

```bash
docker image ls
```

Add the -a flag to include intermediate images:

```bash
docker image ls -a
```

Get detailed information about an image:

```bash
docker image inspect <IMAGE>
```

Use `--format` flag to get only a subset of the information (use Go templates):

```bash
docker image inspect <IMAGE> --format TEMPLATE
```

These commands can both be used to delete an image. Note that if an image has other
tags, they must be deleted first:

```bash
docker image rm <IMAGE_ID>
```

```bash
docker rmi <IMAGE_ID>
```

Delete unused images from the system:

```bash
docker image prune
```

### Flattening an Image

Sometimes, images with fewer layers can perform better. In a few cases, you may want to take an image
with many layers and flatten them into a single layer.

Docker does not provide an official method for doing this, but you can accomplish it by doing the
following:

```bash
docker run --name <CONTAINER_NAME> <IMAGE>
```

```bash
docker export <CONTAINER_NAME> > <CONTAINER_NAME>.tar
```

```bash
cat <CONTAINER_NAME>.tar | docker import - flat:latest
```

### Docker Registries

A Docker Registry is responsible for storing and distributing Docker images.

You can also create your own registries using Docker's open source registry software, or Docker Trusted Registry,
the non-free enterprise solution.

To create a basic registry, simply run a container using registry image and publish port 5000.

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

## Chapter 3 - Orchestration

### Docker Swarm

Docker includesa feature called swarm mode, which allows you to build a distributed cluster of docker machines to
run your containers.

Docker swarm provides many useful features, and can help facilitate orchestration, high-availability, and scaling.

### Configuring a Swarm Manager

Setting up a new swarm is relatively simple. All we have to do is create the first swarm manager:

- Install Docker CE on the Swarm Manager server.

- Initialize the swarm with `docker swarm init --advertise-addr <IP>`.

- If necessary, leave the swarm with `docker swarm leave`.

Once the swarm is initialized, you can see some info about the swarm with `docker info`.

You can list the current node in the swarm with `docker node ls`

### Configuring a Swarm Node

With a manager set up, we can add some worker nodes to the swarm.

- Install Docker CE on both worker nodes.

- Get a join command from the manager: Run `docker swarm join-token worker` on the manager node to get a join command.

- Run the join command on both workers: Copy the join command from the manager and run it on both workers.

### Swarm Backup and Restore

In a production enviroment, it's always a good idea to backup critical data.

Backing up Docker swarm data is fairly simple. To backup, do the following on a swarm manager.

- Stop the Docker service.

- Backup all data in the directory `/var/lib/docker/swarm`.

- Start the Docker service.

To restore the previous backup:

- Stop the Docker service.

- Delete any existing files or directories under `/var/lib/docker/swarm`.

- Copy the backed-up files to `/var/lib/docker/swarm`.

- Start the Docker service.

- Verify both workers have successfully joined the swarm: Run `docker node ls` on the manager and verify that you can see the two worker nodes listed.

### Locking and Unlocking a Swarm Cluster

Docker swarm encrypts sensitive data for security reasons, such as:

- Raft logs on swarm managers.

- TLS communication between swarm nodes.

By default, Docker manages the keys used for this encryption automatically, but they are
stored unencrypted on the managers's disks.

Autolock is a feature that automatically locks the swarm, allowing you to manage the encryption keys
yourself. This gives you control of the keys and can allow for greater security.

However, it requires you to unlock the swarm everytime Docker is restarted on one of your managers.

#### Enable and Disable Autolock

You can enable autolock when you initialize a new swarm with the `--autolock` flag.

```bash
docker swarm init --autolock
```

Enable autolock on a running swarm:

```bash
docker swarm update --autolock=true
```

Disable autolock on a running swarm:

```bash
docker swarm update --autolock=true
```

#### Working with Autolock

Whenever Docker restarts on a manager you must unlock the swarm:

```bash
docker swarm unlock
```

Get the current unlock key for a running swarm:

```bash
docker swarm unlock-key
```

Rotate the unlock key:

```bash
docker swarm unlock-key --roate
```

### High-Availability in a Swarm Cluster

#### Multiple Managers

In order to build a highly-available and fault-tolerant Swarm, it is a good idea to
have multiple swarm managers.

Docker uses the Raft consensus algorithm to maintain a consistent cluster state
across multiple managers.

More manager nodes means better fault tolerance. However, there can be a decrease
in performance as the number of managers grows, since more managers means more network traffic as managers afree to updates in the cluster state.

#### Quorum

A Quorum is the majority (more than half) of the managers in a swarm. For example,
for a swarm with 5 managers, the quorum is 3.

A quorum must be maintained in order to make changes to the cluster state. If a
quorum is not available, nodes cannot be added or removed, new tasks cannot be added, and existing tasks cannot be changed or moved.

Note that since a quorum requires more than half of the manager nodes, it is
recommended to have an odd number of managers.

#### Availability Zones

Docker recommends that you distribute your manager nodes across at least 3 availability zones. Distribute your managers across these zones so that you can maintain a quorum if one of them goes down.

| Manager Nodes | AZ Distribution |
| ------------- | --------------- |
| 3             | 1-1-1           |
| 5             | 2-2-1           |
| 7             | 3-2-2           |
| 9             | 3-3-3           |

### Introoduction to Docker Services

A Service is used to run an application on Docker Swarm. A service specifies a
set of one or more replica tasks. These tasks will be distributed automatically
across the nodes in the cluster and executed as containers.

```bash
docker service create [OPTIONS] <IMAGE> [ARGS]
```

- `--replica`: Specify the number of replica tasks to create for the service.

- `--name`: Specify a name for the service.

- `p`: Publish a port so the service can be accessed externally. The port
  is published on every node in the swarm

#### Managing Services

List current services:

```bash
docker service ls
```

List a service's tasks

```bash
docker service ps <SERVICE>
```

Get more information about a service

```bash
docker service inspect <SERVICE>
```

Make changes to a service:

```bash
docker service update [OPTIONS] <SERVICE>
```

Delete an existing service:

```bash
docker service rm <SERVICE>
```

#### Templates

Templates can be used to give somewhat dynamic values to some flags with `docker service create`

The following flags accept templates:

- `--hostname`

- `--mount`

- `--env`

This command sets an enviroment variable for each container that contains the hostname of the node that container is running on:

```bash
docker service create --env NODE_HOSTNAME="{{.Node.Hostname}}" nginx
```

#### Replicated Services vs. Global Services

Replicated Services run the requests number of replica tasks across the swarm cluster.

```bash
docker service create --replicas 3 nginx
```

Global Services runn one task on each node in the cluster.

```bash
docker service create --mode global nginx
```

#### Scaling Services

Scaling Services means changing the number of replica tasks. There are two ways to scale a service.

Update the service with a new number of replicas.

```bash
docker service update --replicas REPLICAS SERVICE
```

Use docker service scale

```bash
docker service scale SERVICE=REPLICAS
```

### Using docker inspect

Docker inspect is a command that allows you to get information about Docker objects, such as containers, images, services, etc.

```bash
docker inspect <OBJECT_ID>
```

If you know what kind of object you are inspecting, you can also use an allternate form of the command:

```bash
docker container inspect <CONTAINER>
```

```bash
docker service inspect <CONTAINER>
```

This form allows you to specify an object name instead of an ID.

For some object types, you can also supply `--pretty` flag to get a more
readable output.

Use the `--format` flag to retrieve a specific subsection of the data using a Go template.

```bash
docker service inspect --format '{{.ID}}' <SERVICE>
```

## Chapter 4 - Storage and Volumes

### Docker Storage in Depth

Storage drivers are sometims known as Graph drivers. The proper storage driver to use often depends ib your operating system and other local configuration factors.

- `overlay2`: Current Ubuntu and CentOS/RHEL versions.

- `aufs`: Ubuntu 14.04 and older.

- `devicemapper`: CentOS 7 and earlier.

#### Storage models

Persistent data can be managed using several storage models.

##### Filesystem storage

- Data is stored in the form of a file system.

- Used by `overlay2` and `aufs`.

- Efficient use of memory.

- Inefficient with write-heavy workloads.

##### Block storage

- Stores data in blocks.

- Used by `devicemapper`.

- Efficient with write-heavy workloads.

### Configuring DeviceMapper

Device Mapper is one of the Docker storage drivers available for some Linux distributions. It is the default storage driver for CentOS 7 and earlier.

You can customize your DeviceMapper configuration using the daemon config file.

DeviceMapper supports two modes:

#### loop-vm mode

- Loopback mechanism simulates an additional physical disk using files on the local disk.

- Minimal setup, does not require an additional storage device.

- Bad performance, only use for testing.

#### direct-lvm

- Stores data on a separate device.

- Requires an additional storage device

- Good performance, use for prooduction

To configure Docker to use the Device Mapper storage driver via the `/etc/docker/daemon.json` configuration file, you need to specify the storage driver in the JSON format within this file. Here's how you can do it:

```bash
sudo vim /etc/docker/daemon.json
```

Add the following JSON content to the file

```json
{
  "storage-driver": "devicemapper"
}
```

Restart the Docker service for the changes to take effect:

```bash
sudo systemctl restart docker
```

### Docker Volumes

Docker volumes are a wa to persist data generated by and used by Docker containers. They provide a means for sharing data between containers, as well as between the host machine and containers.

#### Bind mounts vs. Volumes

When mounting external storage to a container, you can use either a bind mount or a volume mount.

##### Bind mounts

- Mount specific path on the host machine to the container.

- Not portable, depends on the host machine's file system and directory structure.

##### Volumes

- Stores data on the host file system, but the storage location is managed by Docker.

- More portable.

- Can mount the same volume to multiple containers.

- Work in more scenarios.

#### Working with Volumes

##### --mount syntax

```bash
docker run --mount [key=value]...
```

When you need to connect a location from your host machine directly into your Docker container, you utilize the bind mount mode. Additionally, you can specify the `readonly` parameter to enforce read-only access within the container. In the provided command

```bash
docker run --mount type=bind,source=/home/private,destination=/private,readonly
```

Conversely, if you prefer to use Docker-managed volumes for data persistence and sharing among containers, you can employ the volume mount mode. This mode provides greater control and flexibility over data management. Additionally, you can specify the `readonly` parameter to enforce read-only access within the container. In the provided command

```bash
docker run --mount type=volume,source=private-volume,destination=/private,readonly
```

##### -v syntax

```bash
docker run -v <SOURCE>:<DESTINATION>:[OPTIONS]
```

For bind mounts, you simply specify the source directory on the host machine followed by the destination within the container. For example:

```bash
docker run -v /home/private:/private
```

For Docker-managed volumes, you specify the volume name followed by the destination within the container. Additionally, you can include the :ro suffix to enforce read-only access. For instance:

```bash
docker run -v private-volume:/private:ro
```

#### Managing with Volumes

Create volume

```bash
docker volume create <VOLUME_NAME>
```

List current volumes.

```bash
docker volume ls
```

Get detailed information about a volume.

```bash
docker volume inspect <VOLUME_NAME>
```

Delete volume.

```bash
docker volume rm <VOLUME_NAME>
```

### Image Cleanup

Get information about disk usage on a system.

```bash
docker system df
```

Get even more information about disk usage on a system.

```bash
docker system df -v
```

Remove dangling images (images not referenced by any tag or container).

```bash
docker image prune
```

Remove all unused images (not used by a container).

```bash
docker image prune -a
```

### Chapter 5 - Networking

#### Docker Networking

Docker uses an architecture called 'Container Networking Model (CNM)' to manage networking for Docker containers. The CNM utilizes the following concepts:

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/ikkltkeemeup.png">

- **Sandbox**: An isolated unit containing all networking components associated with a single container. Usually a Linux Network namespace.

- **Endpoint**: Connects a sandbox to a network. Each sandbox/container can have any number of endpoints, but has exactly one endpoint for each network it is connected to.

- **Network**: A Collection of endpoints connected to one another.

- **Network Driver**: Handles the actual implementation of the CNM concepts.

- **IPAM Driver**: IPAM means IP Address Management. Automatically allocates subnets and IP addresses for networks and endpoints.

#### Networking Drivers

Docker includes several built0in network drivers, knowns as 'Native Network Drivers'.
 (CNM)
These network drivers implement the concept descrived in the 'Container Networking Model'.

The 'Native Network Drivers' are host, bridge, overlay, MACVLAN, none. With `docker run`, you can choose a network driver by using `--net=<DRIVER>`

#### The Host Network Driver

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/glgscpmrqrlm.png">

The Host Network Driver allows containers to use the host's network stack directly.

- Containers use the host's networking resources directly.

- No sandboxes, all containers on the host using the host driver share the same network namespace.

- No two containers can use the same port(s).

- Simple and easy setup, one or only few containers on a single host.

#### The Bridge Network Driver

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/nfzcchkupzbb.png">

The Bridge Network Driver uses Linux bridge networks to provide connectivity between containers on the same host.

- This is the default driver for containers running on a single host (i.e., not in a swarm).

- Creates a Linux Bridge for each Docker network.

- Creates a default Linux bridge network called `bridge0`. Containers automatically connect to this if no other network is specified.

- Isolated networking among containers on a single host.

#### The Overlay Network Driver

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/znzxlgsftsnd.png">

The Overlay Network Driver provides connectivity between containers across multiple Docker hosts, i.e. with Docker swarm.

- Uses a VXLAN data plane, which allows the underlying network infrastructure (underlay) to route data between hosts in a way that is transparent to the containers themselves.

- Automatically configures network interfaces, bridges, etc. on each host as needed.

- Networking between containers in a swarm.

#### The MACVLAN Network Driver

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/mnkkmqnzbopk.png">

The MACVLAN Network Driver offers a more lightweight implementation by connecting container interfaces directly to host interfaces.

- Uses direct association with Linux interfaces instead of a bridge interface.

- Harder to confoigure and greater dependency between MACVLAN and external network.

- More lightweight and less latency.

- When there is a need for extremely low latency, or a need for containers with IP addresses in the external subnet.

#### The None Network Driver

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/feveisqnojhu.png">

The None Network Driver does not provide any networking implementation.

- Container is completely isolated from other containers and the host.

- If you want networking with the None driver, you must set everything up manually.

- None does create a separate networking namespace for each container, but no interfaces or endpoints.

- When there is no need for container networking or you want to set all of the networking up yourself.
