# Getting Started

The Evidential suite has a backend API server (FastAPI) and a frontend web-app (NextJS).

!!! note

    Note: See our [Reading List](reading-list.md) for links to documentation on our open-source tech stack.

!!! tip "Contributing from outside the team?"

    The clone commands below use the upstream repositories. If you're an external
    contributor, [fork first](contributing-dev.md) and clone your fork instead — you
    won't have push access to the upstream repos.

!!! tip "Setting up on Windows?"

    Windows needs a few extra steps first. Start with [Windows Setup](#windows-setup) — it sets up WSL and installs
    everything in Prerequisites for you — then continue with [Setup](#setup) inside WSL.

## Prerequisites

1. Install [Task](https://taskfile.dev/).

1. Install Docker.

1. Install [Git LFS](https://git-lfs.com/).

1. Install [NodeJS](https://nodejs.org/en/download) version 26.

1. Install [pnpm](https://pnpm.io/installation). pnpm will select the Node.js version required by the frontend.

1. Install [prek](https://prek.j178.dev/installation/).

## Setup

Run these steps on macOS, on Linux, or inside WSL on Windows.

### Backend

Follow the steps below to get a local development environment running.

1. Clone the [https://github.com/agency-fund/evidential-be](https://github.com/agency-fund/evidential-be) repository:

    ```shell
    gh repo clone agency-fund/evidential-be
    cd evidential-be
    ```

1. Set up the [prek](https://prek.j178.dev/) git hooks in your local checkout:

    ```shell
    prek install
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

### Frontend

1. Clone the [https://github.com/agency-fund/evidential-fe](https://github.com/agency-fund/evidential-fe) repository:

    ```shell
    gh repo clone agency-fund/evidential-fe
    cd evidential-fe
    ```

1. If you installed Node.js with nvm (the Windows setup script does), switch to version 26:

    ```shell
    nvm use 26
    ```

1. Set up the [prek](https://prek.j178.dev/) git hooks in your local checkout:

    ```shell
    prek install
    ```

1. Confirm that pnpm selects and runs the required Node.js version:

    ```shell
    pnpm exec node --version
    ```

1. Start the frontend server:

    ```shell
    task start-airplane
    ```

## Windows Setup

1. From Command Prompt, install WSL 2:

    ```shell
    wsl --set-default-version 2
    wsl --install -d ubuntu-24.04
    ```

1. From Command Prompt, configure WSL networking so that servers running in WSL are reachable from Windows.
    Open the WSL config file:

    ```shell
    notepad %USERPROFILE%\.wslconfig
    ```

    Add the following, then save and close the file:

    ```ini
    [wsl2]
    networkingMode=mirrored
    dnsTunneling=true
    firewall=true
    autoProxy=true
    ```

    Restart WSL to apply the change, then reopen WSL when you're ready to continue.

1. Install the prerequisites by running
    [`tools/windows_setup.sh`](https://github.com/agency-fund/evidential-be/blob/main/tools/windows_setup.sh) from the
    backend repo:

    ```shell
    curl -fsSL https://raw.githubusercontent.com/agency-fund/evidential-be/main/tools/windows_setup.sh -o /tmp/windows_setup.sh
    chmod +x /tmp/windows_setup.sh
    /tmp/windows_setup.sh
    ```

1. Go back to [Setup](#setup) and follow the remaining steps from inside WSL.
