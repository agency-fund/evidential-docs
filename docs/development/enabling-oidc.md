# Enabling OIDC

When running in [airplane mode](airplane-mode.md), you are logged in automatically and skip any authentication steps.
This saves developers time! Sometimes, though, you'll want to log in and log out as you implement features. This
document describes how to connect Evidential to an OpenID Connect (OIDC) identity provider.

!!! note

    Note: This process requires some familiarity with OAuth and your identity provider's administration console. If
    you're unfamiliar, contact us and we can help you.

## How it works

Evidential implements an OIDC RP using the authorization code flow with PKCE. The SPA initiates the PKCE and the backend
does the final token exchange. The ID token issued by the IDP is verified cryptographically and we issue an encrypted
session token to the SPA, which it uses as a bearer token when invoking Evidential APIs.

The SPA acquires the OIDC parameters by requesting configuration data from `/v1/a/oidc/config1`.

Users must be invited to Evidential in order to authenticate. This can be done via the web UI, or via the command line
(run `xngin-cli add-user --help` for details). On first login, the user is bound to the identity provider's issuer and
subject identifier, and only that identity provider can sign them in afterwards.

Session tokens are bound to the configured issuer and client ID, so changing `XNGIN_OIDC_ISSUER` or
`XNGIN_OIDC_CLIENT_ID` will invalidate existing sessions.

The backend requires that the `email_verified` claim indicate that the user's email address has been verified. If this
claim is not present in the ID token, Evidential will fall back to the IDP's userinfo endpoint to find it.

## Backend environment variables

| Variable                   | Required | Description                                                                                                                                                                                                                                                                                                                       |
| -------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `XNGIN_OIDC_ISSUER`        | yes      | The issuer URL. The backend fetches `{issuer}/.well-known/openid-configuration` and requires the `issuer` in that document to match. Must be an `https://` URL (`http://` is accepted only in development) with a hostname and no credentials, query, or fragment. Copy it exactly from the provider, including any trailing `/`. |
| `XNGIN_OIDC_CLIENT_ID`     | yes      | The client ID registered with the identity provider.                                                                                                                                                                                                                                                                              |
| `XNGIN_OIDC_CLIENT_SECRET` | no       | The client secret. Leave it unset for public clients, which authenticate with PKCE alone.                                                                                                                                                                                                                                         |
| `XNGIN_OIDC_REDIRECT_URI`  | yes      | The URL the identity provider redirects to after login: the public URL of the frontend, such as `http://localhost:3000/`. Must be an absolute URL without a fragment, using `https://` outside development. Register the same value with the identity provider.                                                                   |
| `XNGIN_OIDC_CLAIM_MAP`     | no       | Comma-separated `claim:field` pairs that copy ID token claims onto the session principal. The only supported field is `hd`. Example: `hd:hd`.                                                                                                                                                                                     |

The backend always requests the `openid` and `email` scopes; they are not configurable.

The frontend needs only `NEXT_PUBLIC_XNGIN_OIDC_BASE_URL`, the base URL of the backend's OIDC API (for example
`http://localhost:8000/v1/a/oidc`). The frontend Taskfile sets it for development.

## Google

Google requires a client secret at the token endpoint even when using PKCE, so the backend must be configured with
one. See [Google's OpenID Connect documentation](https://developers.google.com/identity/openid-connect/openid-connect)
for details.

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

    | Setting                       | Value                                       |
    | ----------------------------- | ------------------------------------------- |
    | Application Type              | Web Application                             |
    | Name                          | evidential development                      |
    | Authorized JavaScript Origins | `http://localhost:3000`                     |
    | Authorized Redirect URIs      | `http://localhost:3000/` (note: trailing /) |

1. Click the "Create" button.

1. You will be shown a "Client ID" and a "Client Secret" value. Keep these for later.

1. In your backend repository, add or edit your .env file to include these lines:

    ```shell
    XNGIN_OIDC_ISSUER=https://accounts.google.com
    XNGIN_OIDC_CLIENT_ID=[value from previous step]
    XNGIN_OIDC_CLIENT_SECRET=[value from previous step]
    XNGIN_OIDC_REDIRECT_URI=http://localhost:3000/
    XNGIN_OIDC_CLAIM_MAP=hd:hd
    ```

    `hd:hd` records the user's Google Workspace domain (the `hd` claim) on their session principal.

## Okta

Okta's [authorization code flow with PKCE](https://developer.okta.com/docs/guides/implement-grant-type/authcodepkce/main/)
is intended for single-page applications, which have no client secret.

1. In the Okta Admin Console, navigate to Applications > Applications and click "Create App Integration".

1. Choose "OIDC - OpenID Connect" as the sign-in method and "Single-Page Application" as the application type.

1. Configure the application as follows:

    | Setting               | Value                                       |
    | --------------------- | ------------------------------------------- |
    | Grant type            | Authorization Code                          |
    | Sign-in redirect URIs | `http://localhost:3000/` (note: trailing /) |
    | Assignments           | the users or groups who may sign in         |

1. Save the application and note its "Client ID".

1. In your backend repository, add or edit your .env file to include these lines:

    ```shell
    XNGIN_OIDC_ISSUER=https://{yourOktaDomain}/oauth2/default
    XNGIN_OIDC_CLIENT_ID=[client ID from previous step]
    XNGIN_OIDC_REDIRECT_URI=http://localhost:3000/
    ```

    The issuer depends on which Okta authorization server issues the tokens: `https://{yourOktaDomain}` for the org
    authorization server, or `https://{yourOktaDomain}/oauth2/{authorizationServerId}` for a custom authorization
    server such as `default`. The `email` scope, which the backend always requests, is what adds the `email` claim to
    Okta ID tokens.

## Clerk

Clerk can act as an [OIDC identity provider](https://clerk.com/docs/advanced-usage/clerk-idp) for an application
registered as an OAuth application in the Clerk Dashboard.

1. In the Clerk Dashboard, open the OAuth Applications page and create an application.

1. Mark the application as public so that it uses PKCE without a client secret, set the redirect URL to
    `http://localhost:3000/`, and enable the `openid`, `email`, and `profile` scopes.

1. Note the "Client ID" and the Frontend API URL of your Clerk instance, for example
    `https://your-app.clerk.accounts.dev` for a development instance or `https://clerk.example.com` for a production
    instance.

1. In your backend repository, add or edit your .env file to include these lines:

    ```shell
    XNGIN_OIDC_ISSUER=[Frontend API URL from previous step]
    XNGIN_OIDC_CLIENT_ID=[client ID from previous step]
    XNGIN_OIDC_REDIRECT_URI=http://localhost:3000/
    ```

## Running

Instead of using `task start-airplane` to start the services, use `task start` instead. The backend Taskfile sets the
Google values used for development; values in the backend `.env` file (such as `XNGIN_OIDC_CLIENT_SECRET`) override
them.

## Troubleshooting

- **The backend fails to start with `XNGIN_OIDC_... environment variable is not set.`** Every variable marked
    required above must be set unless the backend runs in airplane mode.
- **`Discovery document issuer '...' does not match XNGIN_OIDC_ISSUER '...'.`** The issuer must match the `issuer`
    field of the identity provider's discovery document exactly, including any path (such as Okta's
    `/oauth2/default`).
- **`Discovery document's ... does not include '...'.`** The identity provider does not advertise a capability
    Evidential needs (see "How it works"). For `token_endpoint_auth_methods_supported`, the provider does not
    accept a client secret in the token request body; check whether the application should be a public client
    instead.
- **`JWKS response does not contain any usable RS256 public signing keys.`** The provider's JWKS has no RSA public
    key with a key ID, so ID tokens cannot be verified.
- **The identity provider reports a redirect URI mismatch.** `XNGIN_OIDC_REDIRECT_URI` must be registered with the
    identity provider character for character, including the trailing slash. If your development environment uses a
    different port number, change both.
- **Login succeeds at the identity provider, but Evidential asks you to contact support for access.** The email
    address in the ID token does not match an Evidential user, or the user is bound to a different identity provider.
- **`Email address is not verified`.** The identity provider marked the user's email address as unverified, or
    reported no `email_verified` value in either the ID token or the userinfo response. Verify the address with the
    identity provider and try again. If the provider sends `email_verified` in neither place, configure it to
    include the claim. The backend log explains when the userinfo endpoint could not be queried.
