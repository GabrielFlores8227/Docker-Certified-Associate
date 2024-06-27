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

### 5. Is this a way to configure the Docker engine to use a registry without a trusted TLS certificate? Set INSECURE_REGISTRY in the ‘/etc/docker/default’ configuration file.

- [ ] Yes.
- [x] No.

#### Explanation

To configure the Docker engine to use a registry without a trusted TLS certificate, you should set the `insecure-registries` option in the Docker daemon configuration file, typically located at `/etc/docker/daemon.json`. Here's how you can do it: 

Open the `/etc/docker/daemon.json` file in a text editor. If the file doesn't exist, create it. Add or modify the `ìnsecure-registries` key with the appropriate registry URL.

```json
{
  "insecure-registries": [ "your.registry.com" ]
}
```

### 6. Is this an advantage of multi-stage builds? Better caching when building Docker images

- [ ] Yes.
- [x] No.

#### Explanation

The primary advantages of multi-stage builds in Docker are:

- **Smaller Final Image Size**: By copying the necessary artifacts from intermediate stages, the final image is much smaller and does not include build tools or dependencies that are only needed during the build process.

- **Improved Security**: Reducing the final image size also means fewer vulnerabilities since the final image does not include unnecessary components.

- **Simplified Build Process** Multi-stage builds allow for more complex build processes to be encapsulated within a single Dockerfile, making it easier to manage dependencies and build steps.

Better caching is generally related to how Docker layers are structured and reused, but it is not a direct advantage of multi-stage builds.

### 7. You add a new user to the engineering organization in DTR. Will this action grant them read/write access to the engineering/api repository? Add the user directly to the list of users with read/write access under the repository’s Permissions tab.

- [x] Yes.
- [ ] No.

#### Explanation

In Docker Trusted Registry (DTR), an organization serves as a fundamental organizational unit for managing users, teams, and repositories within a Docker environment. Organizations are pivotal for structuring access control and collaboration, particularly in large-scale deployments where multiple teams or departments need to securely manage and share Docker images.

Within an organization, administrators can create teams and assign users to these teams based on their roles and responsibilities. This hierarchical approach simplifies permission management by allowing permissions to be applied at the team level. For example, an organization might have separate teams for development, QA, and operations, each with distinct access requirements to different sets of Docker repositories.

Each organization in DTR can host multiple repositories, each serving as a container for Docker images related to specific applications, services, or projects. This structure not only organizes Docker images logically but also allows administrators to enforce fine-grained access controls at the repository level. Permissions can be set to control who can view, pull, push, or manage images within each repository, ensuring that sensitive or critical images are safeguarded appropriately.

By utilizing organizations in DTR, companies can streamline their Docker image management workflows. This includes maintaining security by limiting access to authorized personnel, facilitating collaboration by enabling teams to share and iterate on Docker images seamlessly, and enhancing overall operational efficiency through centralized management of Docker resources. Organizations thus play a crucial role in ensuring that Docker-based deployments are secure, scalable, and well-managed across diverse teams and projects within an organization.

### 8. Is this a function of UCP? Image role-based access control

- [x] Yes.
- [ ] No.

#### Explanation

In container environments like UCP, enforcing the use of signed images helps ensure the authenticity and integrity of the software being deployed, enhancing security by preventing unauthorized or tampered images from being used in production environments.

### 9. Is this the purpose of Docker Content Trust? Enable mutual TLS between the Docker client and server.

- [ ] Yes.
- [x] No.

#### Explanation

Enabling mutual TLS (Tansport Layer Security) between the Docker client and server is not the primary purpose of Docker Content Trust (DCT).

Docker Content Trust is primarily designed to provide a way to verify the authenticity and integrity of Docker images. It achieves this by using digital signatures to ensure that only trusted publishers can push images to a registry and that only trusted images can be pulled and deployed on Docker hosts. This helps prevent tampering with images and ensures that only verified content is used in your Docker deployments.
