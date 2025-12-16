# Getting Started

The Evidential suite has a backend API server (FastAPI) and a frontend web-app (NextJS).

!!! note

    Note: See our [Reading List](reading-list.md) for links to documentation on our open-source tech stack.

## Prerequisites

1. Install [Task](https://taskfile.dev/).

1. Install Docker.

1. Install [Git LFS](https://git-lfs.com/).

1. Install [NodeJS](https://nodejs.org/en/download) version 22.

## Backend

Follow the steps below to get a local development environment running.

1. Check out the [https://github.com/agency-fund/evidential-be](https://github.com/agency-fund/evidential-be) repository:

    ```shell
    gh repo clone agency-fund/evidential-be
    cd evidential-be
    ```

1. Install dependencies (Atlas, uv, Python dependencies) by running:

    ```shell
    git lfs install
    task install-dependencies
    ```

1. Run the unit tests:

    ```shell
    task test-airplane
    ```

1. Get familiar with the task runner. Most of the commands you will run are defined in Taskfile.yml. Run:

    ```shell
    task --list
    ```

1. Start the backend server:

    ```shell
    task start-airplane
    ```

    This will start the server at `http://localhost:8000`. It stores its state in a local Postgres instance, running in
    Docker, on `localhost:5499`.

1. Visit the local interactive OpenAPI docs page: `http://localhost:8000/docs`

1. Now set up the pre-commit hooks in your local git with:

    ```shell
    uv run pre-commit install
    ```

## Frontend

1. Check out the [https://github.com/agency-fund/evidential-fe](https://github.com/agency-fund/evidential-fe) repository:

    ```shell
    gh repo clone agency-fund/evidential-fe
    cd evidential-fe
    ```

1. Switch to node version 22:

    ```shell
    nvm use 22
    ```

1. Start the frontend server:

    ```shell
    task start-airplane
    ```
