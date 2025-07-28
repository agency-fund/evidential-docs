# Enabling OIDC

When running in [airplane mode](airplane-mode.md), you are logged in automatically and skip any authentication steps.
This saves developers time! Sometimes, though, you'll want to log in and log out as you implement features. This
document describes how to enable Evidential's integration
with [Google OIDC Connect](https://developers.google.com/identity/openid-connect/openid-connect).

!!! note

    Note: This process requires some familiarity with OAuth and Google Cloud. If you're unfamiliar, contact us and we
    can help you.

1. Log in to the Google Cloud Console.

1. Navigate to [OAuth Overview](https://console.cloud.google.com/auth/overview).

1. If you haven't set up OAuth before, you may be prompted to configure Google Auth Platform. If so, provide the
    following information:

    | Setting             | Value              |
    | ------------------- | ------------------ |
    | App name            | My Evidential      |
    | User Support Email  | your email address |
    | Audience            | Internal           |
    | Contact Information | your email address |

1. Navigate to [Google Auth Platform > Clients](https://console.cloud.google.com/auth/clients).

1. Click "Create Client"

1. Configure the client as follows:

    | Setting                           | Value                                       |
    | --------------------------------- | ------------------------------------------- |
    | Application Type                  | Web Application                             |
    | Name                              | evidential development                      |
    | Authorized JavaScript Origins     | `http://localhost:3000`                     |
    | Authorized Redirect URIs (1 of 2) | `http://localhost:3000/` (note: trailing /) |
    | Authorized Redirect URIs (2 of 2) | `http://localhost:8000/v1/a/oidc/callback`  |

1. Click the "Create" button.

1. You will be shown a "Client ID" and a "Client Secret" value. Keep these for later.

1. In your backend repository, add or edit your .env file to include these lines:

    ```shell
    GOOGLE_OIDC_CLIENT_ID=[value from previous step]
    GOOGLE_OIDC_CLIENT_SECRET=[value from previous step]
    ```

1. In your frontend repository, add or edit your .env file to include this line::

    ```shell
    NEXT_PUBLIC_XNGIN_GOOGLE_CLIENT_ID=[value from previous step]
    ```

1. Instead of using `task start-airplane` to start the services, use `task start` instead.

!!! note

    Note: If your development environment requires you to use a different port number, you may also need to set
    the `GOOGLE_OIDC_REDIRECT_URI` variable on the backend.
