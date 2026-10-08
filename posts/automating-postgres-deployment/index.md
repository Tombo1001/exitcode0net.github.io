# Automating Postgres Deployment with Docker Compose and Init Scripts

> Deploy PostgreSQL with Docker Compose and an init script that creates databases and users automatically. Portable, isolated Postgres with Compose V2 and Bash.

Source: https://exitcode0.net/posts/automating-postgres-deployment/
Author: Tom Cocking (https://tomcocking.com)
Published: 2024-01-06
Updated: 2026-10-08
Tags: docker, docker-compose, postgresql, bash


## Introduction to Running PostgreSQL in a Docker Container using Docker Compose

In this blog post, we will explore how to run [PostgreSQL](https://www.postgresql.org/) in a Docker container using [Docker Compose](https://docs.docker.com/compose/). We will also break down and explain the `init-user-db.sh` script that is executed at startup to initialize the PostgreSQL tables. Running PostgreSQL in a Docker container provides several benefits, including ease of deployment, portability, and isolation. So let's dive in and understand the process!

![PostgreSQL in a Docker container](https://exitcode0.net/images/postgres-on-docker.png)

> This post uses Docker Compose V2, if you are still using V1, consider upgrading otherwise use `docker-compose` instead of `docker compose`.

## Why Run PostgreSQL in a Docker Container?

Running PostgreSQL in a Docker container offers numerous advantages. Here are a few key benefits:

- **Easy Deployment**: Docker simplifies the deployment process by encapsulating PostgreSQL and its dependencies into a container, making it easy to set up and manage.

- **Portability**: Docker containers are self-contained and can be run on any system that supports Docker, ensuring consistent behavior across different environments.

- **Isolation**: Running PostgreSQL in a container provides isolation from the host system, preventing potential conflicts with existing installations or dependencies.

### Setting Up PostgreSQL in a Docker Container with Docker Compose

Before we start, ensure that you have Docker and Docker Compose installed on your system. Once both are set up, you can proceed with the following steps:

1. **Create a Docker Compose file**: Open a text editor and create a file called `docker-compose.yml`. Copy and paste the following contents into the file:

```yaml
version: '3.3'
services:
  postgres:
    container_name: my-postgres
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      PGDATA: /var/lib/postgresql/data/pgdata
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data/pgdata
      - ./init-user-db.sh:/docker-entrypoint-initdb.d/init-user-db.sh
```

In this Docker Compose configuration:

- The `postgres` service is defined with the specified container name (`my-postgres`), the latest PostgreSQL image (`postgres:latest`), and the `restart` policy set to `always` to ensure that the container restarts automatically if it stops.

- The `environment` section sets the PostgreSQL environment variables (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`) using the provided values from the environment. In this example I have a `.env` file in the same directory as the `docker-compose.yml` file, which means that there is no need for sensitive data such as passwords to be stored in our infrastructure code. This `.env` file could be replaced by [git secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) in part of a pipeline deployment.

- The `ports` section maps the host machine's port `5432` to the container's port `5432`, allowing you to access the PostgreSQL database from your host machine.

- The `volumes` section mounts two directories:
  - `./data:/var/lib/postgresql/data/pgdata`: This maps the `./data` directory on your host machine to the `/var/lib/postgresql/data/pgdata` directory inside the container. It allows you to persist the PostgreSQL data files across container restarts.
  - `./init-user-db.sh:/docker-entrypoint-initdb.d/init-user-db.sh`: This mounts the `init-user-db.sh` script from your host machine to the `/docker-entrypoint-initdb.d/init-user-db.sh` path inside the container. This script will be executed during container startup to initialize the PostgreSQL tables.

2. **Start the PostgreSQL container**: Open your terminal or command prompt, navigate to the directory containing the `docker-compose.yml` file, and run the following command:

```shell
docker compose up -d
```

The `-d` flag runs the containers in detached mode, allowing

 them to run in the background.

### Breaking Down the `init-user-db.sh` Script

The `init-user-db.sh` script is executed at startup to initialize the PostgreSQL tables. Let's examine the script and understand its purpose:

```shell
#!/bin/bash

set -e
set -u

export PGDATABASE=${POSTGRES_DB}
export PGUSER=${POSTGRES_USER}
export PGPASSWORD=${POSTGRES_PASSWORD}

RUN_PSQL="psql -X --set AUTOCOMMIT=on --set ON_ERROR_STOP=on "

${RUN_PSQL} <<SQL
CREATE TABLE public.mytable
(
    id uuid NOT NULL,
    sensor_name text NOT NULL,
    battery double precision,
    humidity double precision,
    link_quality double precision,
    temperature double precision,
    voltage integer,
    "time" timestamp with time zone NOT NULL
)
TABLESPACE pg_default;
ALTER TABLE IF EXISTS public.mytable
    OWNER to dbadm;
SQL
```

Here's a breakdown of the script:

- The `set -e` and `set -u` commands ensure that the script exits immediately if any command fails or encounters an unset variable.

- The `export` statements set environment variables to configure the PostgreSQL connection. The variables `PGDATABASE`, `PGUSER`, and `PGPASSWORD` are assigned values based on the corresponding Docker environment variables (derived from the `.env` file).

- The `RUN_PSQL` variable defines the `psql` command with specific options. `-X` disables transaction management, `--set AUTOCOMMIT=on` ensures that each command is executed in its own transaction, and `--set ON_ERROR_STOP=on` stops the script execution if any error occurs.

- The `${RUN_PSQL} <<SQL` syntax starts a here-document that allows us to provide SQL commands inline.

- The SQL commands enclosed within the `SQL` delimiter create a table named `mytable` in the `public` schema with the specified columns and data types. The `ALTER TABLE` statement sets the owner of the table to `dbadm` if the table already exists.

With this updated Docker Compose configuration, you can easily manage and deploy your PostgreSQL container along with the initialization script. Running `docker compose up -d` will start the containers and execute the `init-user-db.sh` script to initialize the PostgreSQL tables.

## Conclusion

In this blog post, we have learned how to run PostgreSQL in a Docker container using Docker Compose. We have seen the advantages of running PostgreSQL in a container and how to set it up with the `docker-compose.yml` file. Additionally, we have explored the `init-user-db.sh` script and its role in initializing the PostgreSQL tables.

By leveraging Docker and Docker Compose, you can easily deploy and manage PostgreSQL in a portable and isolated environment. This approach brings flexibility and scalability to your PostgreSQL deployments, making it an ideal choice for various applications.

Hopefully this blog post has been helpful in understanding the process of running PostgreSQL in a Docker container using Docker Compose. Stay tuned for more exciting tutorials and guides. Consider [buying me a coffee](https://bmc.link/exit) to feed the addiction and help me solve more automation headaches! 

---
Markdown version of https://exitcode0.net/posts/automating-postgres-deployment/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
