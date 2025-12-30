# Forgejo Actions Runner with Docker Compose

This repository provides a Docker Compose setup for a self-hosted [Forgejo Actions Runner](https://forgejo.org/docs/latest/admin/actions/), enabling you to run CI/CD workflows on your own infrastructure.

This setup is based on the guide [Setting up a Self-Hosted Forgejo Actions Runner with Docker Compose](https://linus.dev/posts/setting-up-a-self-hosted-forgejo-actions-runner-with-docker-compose/) by Linus Groh, adapted for a pre-built image and specific network configuration.

## Prerequisites

-   [Docker](https://docs.docker.com/get-docker/)
-   [Docker Compose](https://docs.docker.com/compose/install/)
-   A Forgejo instance (e.g., Codeberg.org or your own self-hosted instance)
-   A registration token from your Forgejo instance (Settings -> Actions -> Runners -> Create new Runner)

## Directory Structure

```
.
├── docker-compose.yml    # The service definition
├── README.md             # This file
└── data/
    └── config.yml        # Runner configuration
```

## Setup Instructions

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd forgejo_runner
```

### 2. Register the Runner

Before starting the daemon, you need to register the runner with your Forgejo instance to obtain credentials.

1.  Run a temporary container to perform the registration:

    ```bash
    docker compose run --rm --entrypoint sh forgejo_runner
    ```

2.  Inside the container, run the registration command. **Important:** Ensure the generated `.runner` file is saved to the `/data` directory so it persists on your host.

    ```bash
    forgejo-runner register --no-interactive --instance <instance_url> --token <registration_token> --name <runner_name> --labels <labels>
    ```
    
    *   Replace `<instance_url>` with your Forgejo URL (e.g., `https://codeberg.org`).
    *   Replace `<registration_token>` with the token you obtained.
    *   Replace `<runner_name>` with a name for your runner.
    *   Replace `<labels>` with the labels you want to support (e.g., `docker:docker://node:16-bullseye,ubuntu-latest:docker://node:16-bullseye`).

    **Alternatively**, you can run the interactive command and move the file afterwards:
    
    ```bash
    forgejo-runner register
    # Follow the prompts...
    mv .runner /data/.runner
    ```

3.  Exit the container:

    ```bash
    exit
    ```

### 3. Configuration

The configuration file is located at `data/config.yml`. It is already pre-configured with sensible defaults for this Docker Compose setup:

-   **Capacity**: Set to `4` concurrent jobs.
-   **Caching**: Enabled and configured to use the internal network.
-   **Network**: Configured to use `forgejo_runner_net` so the runner can communicate with the cache server and spawned job containers.

You can modify this file to adjust settings like capacity or labels.

### 4. Start the Runner

Once registered, start the runner in the background:

```bash
docker compose up -d
```

Check the logs to ensure it started correctly:

```bash
docker compose logs -f
```

## Caching

This setup includes configuration for caching, which speeds up workflows by reusing data between runs.

-   The runner acts as its own cache server on port `7080`.
-   The `docker-compose.yml` exposes this port.
-   The `config.yml` is set to use `host: forgejo_runner` and `port: 7080`.
-   A custom bridge network `forgejo_runner_net` is used to ensure connectivity between the runner and the workflow containers.

## Credits

-   Based on [Setting up a Self-Hosted Forgejo Actions Runner with Docker Compose](https://linus.dev/posts/setting-up-a-self-hosted-forgejo-actions-runner-with-docker-compose/) by Linus Groh.
-   [Forgejo Runner Documentation](https://forgejo.org/docs/latest/admin/actions/)
