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