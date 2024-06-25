### 1. Which docker run option can you use to publish a port so that an application is accessible externally?

- [x] `docker run --publish`
- [ ] `docker run --expose`
- [ ] `docker run --publish-port`
- [ ] `docker run --open-port`

#### Explanation

To publish a port on Docker, you need to use the `-p` (or `--publish` flag) when you run a Docker container. This flag maps a port on your host system to a port inside the Docker container.

```bash
docker run --publish <HOST_POST>:<CONTAINER_PORT> <IMAGE>
```

### 2. What is the difference between the ADD and COPY Dockerfile instructions? (Choose two)

- [ ] ADD supports regular expression handling while COPY does not.
- [ ] COPY supports compression format handling while ADD does not.
- [x] ADD supports compression format handling while COPY does not.
- [ ] COPY supports regular expression handling while ADD does not.
- [x] ADD supports remote URL handling while COPY does not.

#### Explanation

The `ADD` instruction can automatically extract compressed files. For instance, if you have a tar archive named `archive.tar.gz`, `ADD` will extract its contents into the specified directory in the Docker image.

```Dockerfile
FROM ubuntu:latest
ADD archive.tar.gz /app
```

In this example, the `ADD archive.tar.gz /app` will extract the contents of `archive.tar.gz` into the /app directory inside the container.

The `ADD` instruction can also download files from remote URLs directly into the Docker image.

```Dockerfile
FROM ubuntu:latest
ADD https://example.com/file.zip /app/
```

Here, the `ADD https://example.com/file.zip /app/` will download `file.zip` from the provided URL and place in the `/app` directory inside the container.

### 3. A user’s attempts to set the system time from inside a Docker container are unsuccessful. Could SELinux be blocking this operation? 

- [x] Yes.
- [ ] No.

#### Explanation

SELinux is a security architecture for Linux systems that allows administrators to have more control over who can access the system. It enforces mandatory access controls that can restrict what processes can and cannot do, based on defined policies.

SELinux could be blocking this operation because it imposes strict access controls that can prevent Docker containers from performing certain privileged operations, such as setting the system time. Even if the container is running with elevated privileges, SELinux policies might still restrict the ability to change the system time from within the container.

### 4. One of the several containers in a pod has been marked as unhealthy after repeatedly failing its liveness probe. Does the orchestrator take action to fix the unhealthy container by restarting it?

- [x] Yes.
- [ ] No.

#### Explanation

In Kubernetes, a liveness probe is used to determine if a container is still running. If the liveness probe fails, the orchestrator (Kubernetes) considers the container unhealthy and takes action to correct this. The typical action is to restart the unhealthy container.

This behavior is designed to ensure that any transient issues or errors that cause a container to become unresponsive can be resolved by restarting the container, Thereby maintaining the overall health and stability of the application running within the pod.

Therefore, the correct action taken by the orchestrator when a container fails its liveness probe many times it to restart the unhealthy container.

