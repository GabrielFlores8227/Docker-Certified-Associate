### Which docker run option can you use to publish a port so that an application is accessible externally?

- [x] `docker run --publish`
- [ ] `docker run --expose`
- [ ] `docker run --publish-port`
- [ ] `docker run --open-port`

#### Explanation

To publish a port on Docker, you need to use the `-p` (or `--publish` flag) when you run a Docker container. This flag maps a port on your host system to a port inside the Docker container.

```bash
docker run --publish <HOST_POST>:<CONTAINER_PORT> <IMAGE>
```

### What is the difference between the ADD and COPY Dockerfile instructions? (Choose two)

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

### A user’s attempts to set the system time from inside a Docker container are unsuccessful. Could SELinux be blocking this operation? 

- [x] Yes.
- [ ] No.

#### Explanation

SELinux is a security architecture for Linux systems that allows administrators to have more control over who can access the system. It enforces mandatory access controls that can restrict what processes can and cannot do, based on defined policies.

SELinux could be blocking this operation because it imposes strict access controls that can prevent Docker containers from performing certain privileged operations, such as setting the system time. Even if the container is running with elevated privileges, SELinux policies might still restrict the ability to change the system time from within the container.
