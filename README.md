## Chapter 1 - Introduction

### Namespaces

Namespaces are Linux technologies that allows processes to be isolated in terms of the resources that they see. They
can be used to prevent different processes from interfering or interacting with one another.

Docker uses namespaces to isolate containers. This technology allows containers to operate independently and securely.

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
optionally enforce limits and constraints. For example, you can limit the memory available to a specific container. The types or resources managed by cgroups are:

- `CPU`: Controls CPU time allocation for processes.
- `Memory`: Limits the amount of memory processes can use.
- `Block I/O`: Manages and limits disk I/O.
- `Network`: Controls network bandwidth.
- `Devices`: Restricts access to device nodes.
- `Freezer`: Pauses and resumes execution of processes.
- `PIDs`: Limits the number of processes that can be created.

### Namespaces vs. Control Groups

Namespaces create isolated instances of system resources like process IDs and network interfaces, allowing containers to operate independently. Meanwhile, cgroups allocate and regulate system resources such as CPU, memory, and I/O, ensuring equitable distribution among containerized processes. In essence, namespaces provide isolation, while cgroups enforce resource constraints, collectively facilitating robust containerization with secure and efficient resource utilization.

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
Once you have a Dockerfile, you are ready to build your image

```bash
docker build -t <TAG> .
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

First you need to start the container.

```bash
docker run --name <CONTAINER_NAME> <IMAGE>
```

Here, the container is exported into a TAR archive file named <CONTAINER_NAME>.tar. This file will contain the file system of the container.

```bash
docker export <CONTAINER_NAME> > <CONTAINER_NAME>.tar
```

Using cat, the content of the TAR archive is piped into docker import, which creates a new Docker image tagged as flat:latest. This new image effectively represents the original container's file system flattened into a single layer.

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

You can override individual values in the default registry configuration by supplying enviroment variables with `docker run -e`.

Name the variable REGISTRY_ followed by each configuration key, all uppercase and separated by underscores.

For example, to change the config:

```yaml
log:
  level: info
```

Set the enviroment variable:

```bash
docker run -d -p 5000:5000 --restart=always --name registry -e REGISTRY_LOG_LEVEL=debug registry:2
```

#### Securing a Registry

By default, the registry is completely unsecured. It does not use TLS and does not require authentication.

You can take some basic steps to secure your registry:

- Use TLS with a certificate.
- Require user authentication.

### Using Docker Registries

Download an image from a registry to the local system.

```bash
docker pull <IMAGE>
```

Authenticate with a remote registry. When working with Docker registries, if the registry url is not specified, the default registry url will be used (Docker Hub).

```bash
docker login <REGISTRY_URL>
```

To push and pull images from your private registry, tag the images with the registyry hostname (and optionally, port).

```bash
docker pull <REGISTRY_URL>:<PORT>/<IMAGE>
```

```bash
docker push  <REGISTRY_URL>:<PORT>/<IMAGE>
```

#### Untrusted Certificate

When you're trying to log in to a Docker registry and encounter an error due to an untrusted certificate, it usually means that Docker cannot verify the authenticity of the registry server because the certificate presented by the server is not recognized or trusted by Docker. This often happens when the certificate is self-signed or issued by an authority that Docker doesn't recognize by default.

To resolve this issue, you typically need to add the certificate authority (CA) that issued the registry's certificate to Docker's list of trusted CAs. You can do this by configuring Docker to trust the CA by adding it to the insecure-registries or registry-mirrors section in the `/etc/docker/daemon.json` file.

```json
{
  "insecure-registries": ["your.registry.example.com"]
}
```

Restart the Docker service to apply the changes:

```bash
systemctl restart docker
```

After making these changes, Docker should trust the registry's certificate, and you should be able to log in without encountering certificate errors. However, keep in mind that adding insecure registries may pose security risks, as communication with these registries is not encrypted. Only use this approach with registries you trust and when encryption is not a concern.

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

### Introduction to Docker Services

A Service is used to run an application on Docker swarm. A service specifies a
set of one or more replica tasks. These tasks will be distributed automatically
across the nodes in the cluster and executed as containers.

```bash
docker service create <IMAGE>
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
docker service update <SERVICE>
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
docker service create --env NODE_HOSTNAME="{{.Node.Hostname}}" <IMAGE>
```

#### Replicated Services vs. Global Services

Replicated Services run the requests number of replica tasks across the swarm cluster.

```bash
docker service create --replicas 3 <IMAGE>
```

Global Services run one task on each node in the cluster.

```bash
docker service create --mode global <IMAGE>
```

#### Scaling Services

Scaling Services means changing the number of replica tasks. There are two ways to scale a service.

Update the service with a new number of replicas.

```bash
docker service update --replicas <REPLICAS> <SERVICE>
```

Use docker service scale syntax.

```bash
docker service scale <SERVICE>=<SERVICE>
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

### Docker Compose

Docker Compose is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.

Compose simplifies the control of your entire application stack, making it easy to manage services, networks, and volumes in a single, comprehensible YAML configuration file. Then, with a single command, you create and start all the services from your configuration file.

Compose works in all enviroments; production, staging, development, testing, as well as CI workflows. It also has commands for managing the whole lifecycle of your application:

- Start, stop, and rebuild services.
- View the status of running services.
- Stream the log output of running services.
- Run a one-off command on a service.

#### Setting up Docker Compose

Make a directory to contain your Docker Compose project.

```bash
mkdir my-docker-project
cd my-docker-project
```

Add a `docker-compose.yml` file to the directory and define your application within it. Here's an example:

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html

  app:
    image: myapp:latest
    build:
      context: ./app
    ports:
      - "3000:3000"
    depends_on:
      - db

  db:
    image: postgres:latest
    environment:
      POSTGRES_USER: example
      POSTGRES_PASSWORD: example
      POSTGRES_DB: exampledb
```

Create and run the resources defined in `docker-compose.yml`. The `-d` flag runs the application in detached mode.

```bash
docker-compose up -d
```

Display the containers/services currently running under Docker Compose.

```bash
docker-compose ps
```

Stop and remove all resources that were created using `docker-compose up`.

```bash
docker-compose down
```

### Docker Stacks

Services are capable of running a single, replicated application across nodes in the cluster, but what if you need to deploy a more complex application consisting of multiple services?

A Stack is a collection of interrelated services that can be deployed and scaled as a unit.

Docker Stacks are similar to the multi-container applications created using Docker Compose. However, they can be scaled and executed across the swarm just like normal swarm services.

#### Setting up Docker Stacks

Initialize a Docker Swarm if not already done.

```bash
docker swarm init
```

Docker Stacks uses the same docker-compose.yml file format, but with added features for Swarm mode. Here's an example:

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
      restart_policy:
        condition: on-failure

  app:
    image: myapp:latest
    build:
      context: ./app
    ports:
      - "3000:3000"
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: any
    depends_on:
      - db

  db:
    image: postgres:latest
    environment:
      POSTGRES_USER: example
      POSTGRES_PASSWORD: example
      POSTGRES_DB: exampledb
    volumes:
      - db-data:/var/lib/postgresql/data
    deploy:
      placement:
        constraints: [node.role == manager]

volumes:
  db-data:
```

Deploy the stack using the docker stack deploy command.

```bash
docker stack deploy -c docker-compose.yml <STACK>
```

View the stacks deployed in the Swarm.

```bash
docker stack ls
```

Display the services running in a specific stack.

```bash
docker stack services <STACK>
```

Remove the stack and all its associated services.

```bash
docker stack rm <STACK>
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

## Chapter 5 - Networking

### Docker Networking

Docker uses an architecture called 'Container Networking Model (CNM)' to manage networking for Docker containers. The CNM utilizes the following concepts:

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/ikkltkeemeup.png">

- **Sandbox**: An isolated unit containing all networking components associated with a single container. Usually a Linux Network namespace.

- **Endpoint**: Connects a sandbox to a network. Each sandbox/container can have any number of endpoints, but has exactly one endpoint for each network it is connected to.

- **Network**: A Collection of endpoints connected to one another.

- **Network Driver**: Handles the actual implementation of the CNM concepts.

- **IPAM Driver**: IPAM means IP Address Management. Automatically allocates subnets and IP addresses for networks and endpoints.

### Networking Drivers

Docker includes several built-in network drivers, referred to as 'Native Network Drivers', which implement the 'Container Networking Model (CNM)'. These drivers facilitate different networking configurations for containers. The primary network drivers are:

- **Host**: Connects the container directly to the host's network stack, sharing the host's IP address.

- **Bridge**: Creates a private internal network on the host, where containers connected to the same bridge can communicate.

- **Overlay**: Enables communication between containers across multiple Docker hosts, forming a distributed network.

- **MACVLAN**: Assigns a unique MAC address to each container, making them appear as physical devices on the network.

- **None**: Disables all networking for the container, effectively isolating it.

#### The Host Network Driver

The Host Network Driver allows containers to utilize the host's network stack directly, sharing the same network namespace as the host. This configuration bypasses the isolation provided by Docker's default bridge network, enabling containers to use the host's IP address and port range. Consequently, no two containers can use the same port simultaneously. This driver simplifies networking setup and improves performance by reducing overhead, making it ideal for scenarios where low-latency communication is crucial and only a few containers are running on a single host. However, it also increases security risks since all containers share the host's network namespace.

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/glgscpmrqrlm.png">

To run a container using the host network driver in Docker, you can specify the network mode as 'host' when running the container.

```bash
docker run --net host <IMAGE>
```

or

```bash
docker run --network=host <IMAGE>
```

#### The Bridge Network Driver

The Bridge Network Driver uses Linux bridge networks to provide connectivity between containers on the same host. This is the default driver for containers not running in a swarm. It creates a Linux bridge for each Docker network and establishes a default bridge network named bridge0. Containers automatically connect to this default bridge if no other network is specified. The bridge network driver offers isolated networking among containers on a single host, making it suitable for standard applications where container-to-container communication is required without exposing them to the host's network directly.

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/nfzcchkupzbb.png">

When you run a container without specifying a network, Docker automatically connects it to the default bridge network. This default bridge network simplifies container deployment by providing a basic networking setup out of the box. Containers on this network can communicate with each other by default, making it convenient for many use cases.

```bash
docker run <IMAGE>
```

However, in certain scenarios, you may require more control and isolation over your container networking. In such cases, you can create your own custom bridge network. Custom bridge networks allow you to define specific network configurations tailored to your application's needs. You may want to create a custom bridge network to:

- Isolate your containers from other networks or the host network.

- Provide a specific naming convention or network settings.

- Segment your application into different network layers for security or performance reasons.

To create and run containers on a custom bridge network, follow these steps:

Create a custom bridge network using the `docker network create command`, specifying the desired network name.

```bash
docker network create <NETWORK>
```

When running a container, specify the custom bridge network using the `--net` flag along with the name of the network created in the previous step.

```bash
docker run --net <NETWORK> <IMAGE>
```

By creating and utilizing a custom bridge network, you gain greater flexibility and control over your container networking environment, allowing you to tailor it to your specific requirements.

#### The Overlay Network Driver

The Overlay Network Driver facilitates seamless communication between containers across multiple Docker hosts, primarily within Docker swarm environments. It employs a VXLAN data plane, enabling transparent routing of data between hosts within the swarm while abstracting the underlying network infrastructure (underlay) from the containers. This driver automatically sets up network interfaces, bridges, and other necessary components on each host within the swarm, simplifying the networking setup process. It enables effortless networking between containers within a Docker swarm, ensuring they can communicate seamlessly irrespective of their physical host, crucial for distributed applications such as microservices architectures or high availability setups.

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/znzxlgsftsnd.png">

In a Docker swarm environment, when you deploy services without specifying a network, Docker automatically connects them to the default overlay network. The default overlay network simplifies service deployment by providing a built-in networking solution for container communication across multiple hosts within the swarm. Containers and services connected to this network can communicate seamlessly across swarm nodes, making it convenient for distributed applications.

```bash
docker service create --name <SERVICE> <IMAGE>
```

However, in more complex swarm setups or specific application requirements, you may need to create your own custom overlay network. Custom overlay networks allow you to define specific network configurations tailored to your application's needs within the swarm environment. You might create a custom overlay network to:

- Isolate your services from other networks or the host network.

- Configure network settings such as subnet range, ingress routing mesh, or encryption.

- Organize services into logical network segments for better management and security.

To create and deploy services on a custom overlay network, follow these steps:

Create a custom overlay network using the `docker network create` command, specifying the `--driver overlay` option and the desired network name.

```bash
docker network create --driver overlay <NETWORK>
```

When deploying a service in the swarm, specify the custom overlay network using the `--net` flag along with the name of the network created in the previous step.

```bash
docker service create --name <SERVICE> --net <NETWORK> <IMAGE>
```

By creating and utilizing a custom overlay network, you gain greater flexibility and control over your service networking within the Docker swarm, allowing you to tailor it to your specific requirements.

#### The MACVLAN Network Driver

The MACVLAN Network Driver offers a lightweight approach by directly connecting container interfaces to host interfaces. It associates with Linux interfaces directly instead of utilizing a bridge interface, offering less overhead and latency. However, configuration is more complex, and there's a stronger dependency between MACVLAN and the external network. It's ideal for scenarios requiring extremely low latency or containers with IP addresses in the external subnet.

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/mnkkmqnzbopk.png">

When you run a container without specifying a network driver, Docker connects it to the default network, which is typically the bridge network. However, for scenarios where you require containers to directly connect to physical networks with their own MAC addresses, you may opt for the MACVLAN driver.

The MACVLAN driver offers a lightweight solution by directly connecting container interfaces to host interfaces. This approach is beneficial for scenarios requiring containers to appear as physical devices on the network, with their own MAC addresses and potentially separate IP addresses.

To leverage the MACVLAN driver, follow these steps:

Create a custom MACVLAN network using the docker network create command, specifying the `--driver macvlan` option and providing additional configuration such as subnet, gateway, parent interface, etc.

```bash
docker network create --driver macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 <NETWORK>
```

When running a container, specify the custom MACVLAN network using the --network flag along with the name of the network created in the previous step.

```bash
docker run --net <NETWORK> <IMAGE>
```

By creating and utilizing a custom MACVLAN network, you gain the ability to integrate containers directly into your existing physical network infrastructure, enabling them to behave like separate physical devices on the network. This is particularly useful for scenarios where containers require direct access to specific network resources or need to communicate directly with other devices on the network.

#### The None Network Driver

The None Network Driver provides complete isolation for containers without any networking implementation. Containers using this driver are entirely isolated from both other containers and the host. Manual setup is required if networking is desired with the None driver. Although it creates separate networking namespaces for each container, no interfaces or endpoints are established. This driver is suitable when container networking is unnecessary or when custom networking setup is preferred.

<img width="600" src="https://github.com/GabrielFlores8227/Docker-Certified-Associate/blob/main/.assets/feveisqnojhu.png">

To run a container using the None network driver, you simply specify the `--net none` option when launching the container.

```bash
docker run --net none <IMAGE>
```

### Creating a Bridge Network

You can create and manage your own networks with the `docker network` commands. If you do not
specify a network driver, bridge will be used.

Create a network.

```bash
docker network create <NETWORK>
```

Run a new container, connecting it to the specified network.

```bash
docker run --network <NETWORK>
```

Connect a running container to an existing network.

```bash
docker network connect <NETWORK> <CONTAINER>
```

#### Embedded DNS

Docker networks implement an embedded DNS server, allowing containers and services to locate and communicate with one another.

Containers can communicate with other containers and services using the service or container name, or network alias.

```bash
docker run --network-alias <ALIAS> <IMAGE>
```

Provide a network alias to a container.

```bash
docker network connect --alias <ALIAS> <IMAGE>
```

#### Managing Networks

List networks.

```bash
docker network ls
```

Inspect network

```bash
docker network inspect <NETWORK>
```

Disconnect a running container from an existing network.

```bash
docker network disconnect <NETWORK> <CONTAINER>
```

Delete network.

```bash
docker network rm <NETWORK>
```

### Deploy a Service on a Docker Overlay Network

You can create a network with the overlay driver to provide connectivity between services and containers in Docker swarm.

By default, services are attached to a default overlay network called ingress. You can create your own networks to provide isolated communication between services and containers.

Create an overlay network.

```bash
docker network create --driver overlay <NETWORK>
```

Run a service attached to an existing overlay network.

```bash
docker service create --network <NETWORK> <IMAGE>
```

## Chapter 6 - Security

### Signing Images and Enabling Docker Content Trust

Docker Content Trust (DCT) provides aa secure way to verify the integrity of images before you pull or run them on your systems.

With DCT, the image creator signs each image with a certificate, which clients can use to verify the image before running it.

Generate a delegation key pair. This gives users access to sign images for a repository.

```bash
docker trust key generate <SIGNER_NAME>
```

Add a signer (user) to a repo.

```bash
docker trust signer add --key <KEY_FILE> <SIGNER_NAME> <REPO>
```

Sign an image and pish it to registry.

```bash
docker trust sign <REPO>:<TAG>
```

#### Enabling DCT

Docker Content Trust can be enabled by setting the `DOCKER_CONTENT_TRUST` enviroment to 1.

In Docker Enterprise Edition, you can also enable it in `daemon.json`.

When DCT is enabled, Docker will only pull and run signed images. Attempting yo upll and/or run an unsined image will result in an error message.

Note that when `DOCKER_CONTENT_TRUST=1`, `docker push` will automatically sign the image before pushing it.

### Default Docker Engine Security

Namespaces and Control Groups (cgroups) provide isolation to containers.

Isolation means that container processes cannot see or affect other containers or processes running directly on the host system.

This limits the impact of certain exploits or pivilege escalation attacks. If one ccontainer is compromised, it is less likely that it can be used to gain any further access outside the container.

#### Docker Daemon Attack Surface

It is important to note that the Docker daemon itself requires root privileges. Therefore, you should be aware that of the potential attack surface presented by the Docker daemon.

Only allow trusted users to access the daemon. Control of the Docker daemon could allow the entire host to be compromised.

Be aware of this you are building any automation that accesses the Docker daemon, or grating any userrs direct access to it.

#### Linux Kernel Capabilities

Docker uses capabilities to fine-tune what container processes can access.

This means that a process can run as root inside a container, but does not have access to do everything root could normally do on the host.

For example, Docker uses the `net_bind_service` capability to allow container processes to bind to a port below 1024 without running as root.

### Docker MTLS

Docker swarm provides additional security by encrypting communication between various components in the cluster.

Mutually Authenticated Transport Layer Security (MTLS)

- Both participants in communication exchanges certificated and all communication is authenticated and ecrypted.

- When a swarm is initialized, a root certificate authority (CA) is created, which is used to generate certificates for all nodes as they join the cluster.

- Worker and manager tokens are generated using the CA and are used to join new nodes to the cluster.

- Used for all cluster-level communication between swarm nodes.

- Enabled by default, you don't need to doo anything to set it up.

#### Encrypt Overlay Networks

You can encrypt communication between containers on overlay networks in order to provide greater security within your swarm cluster.

Use the `--opt encrypted` flag when creating an overlay network to encrypt it.

```bash
docker network create --opt encrypted --driver overlay <NETWORK>
```

## Chapter 7 - Docker Enterprise

### Universal Control Plane

Docker Universal Control Plane (UCP) is a robust enterprise-level cluster management solution that extends beyond the basic functionalities of Docker Swarm.

While UCP might initially appear to be just "Docker Swarm with a web interface," it offers a comprehensive suite of advanced features, including:

- **Organization and Team Management**: Efficiently manage user groups and streamline collaboration within your organization.

- **Role-Based Access Control (RBAC)**: Implement fine-grained access controls to ensure that users have the appropriate permissions for their roles, enhancing security and compliance.

- **Multi-Orchestrator Support**: Seamlessly orchestrate containers with both Docker Swarm and Kubernetes, giving you the flexibility to choose the best tool for your needs.

These capabilities make UCP a powerful tool for managing complex containerized environments in an enterprise setting.'

### Universal Control Plane (UCP)

Docker Universal Control Plane (UCP) is a robust enterprise-level cluster management solution that extends beyond the basic functionalities of Docker Swarm.

While UCP might initially appear to be just "Docker Swarm with a web interface," it offers a comprehensive suite of advanced features, including:

- **Organization and Team Management:** Efficiently manage user groups and streamline collaboration within your organization.
- **Role-Based Access Control (RBAC):** Implement fine-grained access controls to ensure that users have the appropriate permissions for their roles, enhancing security and compliance.
- **Multi-Orchestrator Support:** Seamlessly orchestrate containers with both Docker Swarm and Kubernetes, giving you the flexibility to choose the best tool for your needs.

These capabilities make UCP a powerful tool for managing complex containerized environments in an enterprise setting.

#### UCP Security

Docker Universal Control Plane (UCP) offers a flexible and robust model for managing access to cluster resources and functionality. Here’s an overview of the key components in UCP's security architecture:

- **User:** An authenticated individual who can access the system.

- **Team:** A group of users that share specific permissions.

- **Organization:** A collection of teams that share certain permissions.

- **Service Account:** A Kubernetes object that enables a container to access cluster resources.

- **Subject:** An entity (user, team, organization, or service account) with the capability to perform actions within the cluster.
- **Resource Set:** A collection of Docker Swarm objects, including containers, services, and nodes.

- **Role:** A set of permissions that allows operations on objects within a resource set.

- **Grant:** The assignment of a specific role to a subject concerning a resource set.

Additionally, UCP supports LDAP integration, enabling streamlined management of users and teams through an LDAP-enabled user directory. This integration enhances security by centralizing user authentication and access control.

### Docker Enterprise Edition (DEE) Trusted Registry

Docker Enterprise Edition (DEE) Trusted Registry (DTR) is a secure, scalable, and robust solution for storing and managing Docker images within your enterprise. DTR provides several advanced features that make it an essential tool for managing containerized applications at scale.

#### Key Features of DTR

- **Security and Access Control:** DTR allows you to define detailed access control policies, ensuring that only authorized users can access specific images.

- **Image Scanning:** Automatically scan images for vulnerabilities, ensuring that only secure images are deployed in your environment.

- **Content Trust:** Use digital signatures to verify the authenticity and integrity of Docker images.

- **Automated Image Promotion:** Define policies to automatically promote images through different stages of your CI/CD pipeline based on predefined criteria.

- **High Availability:** DTR can be deployed in a highly available configuration to ensure that your registry is always accessible.

- **LDAP Integration:** Integrate with LDAP for centralized user management and authentication.

#### DTR Security

DTR incorporates several features to enhance the security of your Docker images and registry operations:

- **Role-Based Access Control (RBAC):** Implement fine-grained access controls, assigning specific permissions to users, teams, and organizations.

- **Image Scanning and Vulnerability Detection:** Regularly scan images to detect vulnerabilities and ensure compliance with security policies.

- **Notary Integration for Content Trust:** Leverage Docker Content Trust to sign and verify images, ensuring their authenticity and integrity.

- **Audit Logs:** Maintain detailed logs of all actions performed in the registry for auditing and compliance purposes.

By using DEE Trusted Registry, organizations can ensure that their container images are securely stored, managed, and deployed, reducing the risk of security breaches and enhancing operational efficiency.
