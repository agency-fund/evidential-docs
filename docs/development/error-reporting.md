# Error Reporting

Evidential has optional support for using [Sentry](https://sentry.io/) for capturing operational errors and metrics
from the backend service.

These environment variables are used to configure Sentry:

| Variable      | Description                                                                                    |
| ------------- | ---------------------------------------------------------------------------------------------- |
| `SENTRY_DSN`  | The Sentry DSN. If unset, Sentry will not be enabled.                                          |
| `ENVIRONMENT` | The "environment" tag reported with the Sentry events. If unset, the default value is "local". |
