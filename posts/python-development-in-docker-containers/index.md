# Python Development in Docker Containers

> Develop Python scripts inside Docker containers for reproducible, isolated environments, and how that compares with virtual environments for team tooling.

Source: https://exitcode0.net/posts/python-development-in-docker-containers/
Author: Tom Cocking (https://tomcocking.com)
Published: 2024-01-07
Updated: 2026-10-08
Tags: docker, python, docker-compose, development


Docker Compose is a powerful tool that allows developers to define and manage multi-container applications. In this blog post, we will explore a [Docker Compose](https://www.docker.com/) file for developing [Python](https://www.python.org/) applications. Specifically, we will analyze the following Docker Compose file:

```yaml
version: '3'
services:
  mypthontool:
    image: python:3.9
    volumes:
      - ./:/app/
    tty: true
```

Now all that is left to do is bring the container online with `docker compose up -d` and access the bash terminal in the container to run our code in a repeatable environment using `docker exec -it <container name> /bin/bash`.

> This post uses Docker Compose V2, if you are still using V1, consider upgrading otherwise use `docker-compose` instead of `docker compose`.

## Understanding the Docker Compose File

Let's break down the Docker Compose file and understand each component:

- `version: '3'`: This line specifies the version of Docker Compose being used. In this case, we are using version 3 - more information on compose versions can be found [here](https://docs.docker.com/compose/compose-file/compose-versioning/).

- `services`: This section defines the services or containers in the application. In our case, we have a single service named `mypthontool`.

- `mypthontool`: This is the name of our service. You can modify it according to your project's requirements.

- `image: python:3.9`: This line specifies the Docker image that will be used to create the container for our Python application. In this case, we are using the official Python 3.9 image. This can be updated to whatever image meets your needs: [Python Docker Builds](https://hub.docker.com/_/python).

- `volumes: - ./:/app/`: Here, we define a volume mapping between our local directory (the current directory denoted by `./`) and the `/app/` directory within the container. This allows us to share code between our host machine and the container.

- `tty: true`: This line enables a pseudo-TTY for the container, allowing us to interact with it in an interactive manner.

## Benefits of Developing Python Applications in a Container Environment

Developing Python applications in a container environment offers several advantages:

1. **Portability**: With Docker, you can create a containerized environment that encapsulates all the dependencies and configurations required for your Python application. This ensures consistent behavior across different development machines and deployment environments.

2. **Isolation**: Containers provide a sandboxed environment where your application and its dependencies run independently of the host system. This isolation helps avoid conflicts between different applications or libraries.

3. **Reproducibility**: By defining your application's environment in a Docker Compose file, you can easily recreate the exact same environment on any machine. This eliminates the notorious "it works on my machine" problem and makes it easier to collaborate with other developers.

4. **Scalability**: Docker allows you to scale your application by spinning up multiple containers to handle increased workload. This makes it convenient to develop and test distributed systems or microservices architectures.

## Comparison: Development in a Container vs. Python Virtual Environments (venv)

While Python virtual environments (venv) also provide a way to isolate Python environments, there are some notable differences when compared to containerized development:

**1. Dependency Isolation**: In a container environment, you can encapsulate not only your Python dependencies but also the entire operating system and runtime environment. This ensures that your application behaves consistently across different systems. With venv, you can only isolate Python packages.

**2. System-Level Dependencies**: Containers allow you to include system-level dependencies like databases, message queues, or other services required by your application. These dependencies can be easily managed and replicated across different environments. In venv, you would need to install and configure these dependencies separately on each development machine.

**3. Platform Compatibility**: Containers are platform-agnostic, meaning you can develop and

 test your application on one platform (e.g., Windows) and deploy it on another (e.g., Linux) without worrying about compatibility issues. venv, on the other hand, is tied to the specific host system and may not work seamlessly across different platforms.

**4. Development Environment Consistency**: Docker Compose provides a standardized environment for the entire development team. It ensures that everyone is working with the same set of dependencies and configurations. venv relies on developers to create and maintain their own virtual environments, leading to potential inconsistencies and compatibility problems.

In conclusion, developing Python applications in a container environment using Docker Compose offers enhanced portability, isolation, reproducibility, and scalability. While venv provides dependency isolation at the Python package level, it lacks the comprehensive environment encapsulation and platform compatibility offered by Docker containers. Consider using Docker Compose for your Python projects to streamline development, improve collaboration, and ensure consistent application behavior across different environments.

Start containerizing your Python applications today and experience the benefits of a containerized development workflow with Docker Compose!

*Note: This blog post is for informational purposes only and does not cover all aspects of Docker Compose or Python virtual environments. Make sure to refer to [official documentation](https://docs.docker.com/compose/) and best practices for detailed instructions.*

![Python development in a Docker container](https://exitcode0.net/images/containerized-python.jpg)
_Stable Diffusion's summary of this post - have no fear artists of the world!_

---
Markdown version of https://exitcode0.net/posts/python-development-in-docker-containers/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
