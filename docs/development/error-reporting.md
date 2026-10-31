# Error Reporting

Evidential has optional support for using [Sentry](https://sentry.io/), [Glitchtip](https://glitchtip.com/) (open source), or any
other Sentry- compatible service for capturing operational errors and metrics from the backend service.

These environment variables are used to configure Sentry:

| Variable      | Description                                                                                    |
| ------------- | ---------------------------------------------------------------------------------------------- |
| `SENTRY_DSN`  | The Sentry DSN. If unset, Sentry SDK will not be enabled.                                      |
| `ENVIRONMENT` | The "environment" tag reported with the Sentry events. If unset, the default value is "local". |

In some cases, we will send additional detail along with the error reports, such as Organization ID, Datasource ID,
User ID, Experiment ID, or a request correlation ID.
