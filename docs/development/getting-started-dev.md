# Getting Started

The Evidential suite has a backend API server (FastAPI) and a frontend web-app (NextJS).

!!! note

    Note: See our [Reading List](reading-list.md) for links to documentation on our open-source tech stack.

!!! tip "Contributing from outside the team?"

    The clone commands below use the upstream repositories. If you're an external
    contributor, [fork first](contributing-dev.md) and clone your fork instead — you
    won't have push access to the upstream repos.

## Prerequisites

1. Install [Task](https://taskfile.dev/).

1. Install Docker.

1. Install [Git LFS](https://git-lfs.com/).

1. Install [NodeJS](https://nodejs.org/en/download) version 22.

## macOS and Linux Setup

On Windows, start with [Windows Setup](#windows-setup), then follow these steps inside WSL.

### Backend

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

### Frontend

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

## Windows Setup

1. From Command Prompt, install WSL 2 and Debian:

    ```shell
    wsl --set-default-version 2
    wsl --install -d Debian
    ```

1. From Command Prompt, enable mirrored networking mode so that servers running in WSL are reachable from Windows.
    Open the WSL config file:

    ```shell
    notepad %USERPROFILE%\.wslconfig
    ```

    Add the following, then save and close the file:

    ```ini
    [wsl2]
    networkingMode=mirrored
    ```

    Restart WSL to apply the change, then reopen WSL when you're ready to continue:

    ```shell
    wsl --shutdown
    ```

1. Install the prerequisites:

    ```shell
    curl -fsSL https://gist.githubusercontent.com/Snehaaa18/f17e639c90751c2acd4fbd93f04c1608/raw/e3a85bffc762c3f4b73edba91a3301e83e70c151/evid-setup.sh -o /tmp/evid-setup.sh
    chmod +x /tmp/evid-setup.sh
    /tmp/evid-setup.sh
    ```

1. Clone the repositories:

    ```shell
    mkdir -p ~/src
    cd ~/src
    git clone https://github.com/agency-fund/evidential-be.git
    git clone https://github.com/agency-fund/evidential-fe.git
    ```

1. Run Evidential. Start the frontend server:

    ```shell
    cd ~/src/evidential-fe
    task start-airplane
    ```

    Then, in a second shell, start the backend server:

    ```shell
    cd ~/src/evidential-be
    task start-airplane
    ```
