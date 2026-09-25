# Developing Offline

"Airplane Mode" allows the frontend and backend to start up without relying on any internet resources. This allows
development when disconnected from the internet.

When in airplane mode:

- You will be logged in automatically as testing@example.com.
- The backend Admin API calls and the frontend will authenticate you as testing@example.com.
- You cannot develop login/logout features when airplane mode is enabled.
- testing@example.com will have an organization and sample experiments created automatically.
- OIDC integration is disabled.

To start the [frontend](https://github.com/agency-fund/evidential-fe) in airplane mode, run:

```shell
cd evidential-fe
task start-airplane
```

To start the [backend](https://github.com/agency-fund/evidential-be) in airplane mode, run:

```shell
cd evidential-be
task start-airplane
```

To run the backend tests in airplane mode, use the `task test-airplane` command.
