# Secrets<a name="secrets"></a>

<!-- mdformat-toc start --slug=github --maxlevel=3 --minlevel=1 -->

- [Secrets](#secrets)
    - [Overview](#overview)
    - [Supported Encryption Backends](#supported-encryption-backends)
    - [Algorithm Details](#algorithm-details)
    - [Configuration](#configuration)
        - [Selecting an Encryption Provider](#selecting-an-encryption-provider)
        - [Enabling NaCl Encryption](#enabling-nacl-encryption)
        - [Enabling Google Cloud KMS](#enabling-google-cloud-kms)
        - [Debugging GCP KMS](#debugging-gcp-kms)

<!-- mdformat-toc end -->

## Overview<a name="overview"></a>

Evidential encrypts customer-provided database credentials to enhance security. This encryption reduces the risk of
credential leakage through API server vulnerabilities, database backups, or other operational practices.

We implement encryption-at-rest for:

- BigQuery service account credentials
- Postgres/Redshift connection passwords

## Supported Encryption Backends<a name="supported-encryption-backends"></a>

To accommodate various deployment scenarios, we offer these encryption options:

| Backend                     | Use Case                                         | Description                                                                     |
|-----------------------------|--------------------------------------------------|---------------------------------------------------------------------------------|
| No-op (`noop`)              | Local development only                           | No encryption is applied                                                        |
| NaCl (`nacl`)               | Traditional web hosts, self-hosting, development | Uses symmetric encryption with a key accessible to the API server               |
| Google Cloud KMS (`gcpkms`) | GCP deployments                                  | Uses Google Cloud Key Management Service (KMS) to implement envelope encryption |

## Algorithm Details<a name="algorithm-details"></a>

Our "nacl" backend uses XChaCha20-Poly1305 as provided by
the [PyNaCl/libsodium](https://pynacl.readthedocs.io/en/latest/) library. We employ authenticated data (AAD) to bind
ciphertexts to server-generated database identifiers.

Our KMS backends use [Envelope Encryption](https://cloud.google.com/kms/docs/envelope-encryption). Every ciphertext
is encrypted with a unique key, and that key is encrypted by the cloud provider.

In both of these scenarios, customer data is always encrypted at rest.

## Configuration<a name="configuration"></a>

### Selecting an Encryption Provider<a name="selecting-an-encryption-provider"></a>

The system automatically initializes backends based on environment variables specific to each provider.

Set the `XNGIN_SECRETS_BACKEND` environment variable to specify which backend encrypts values. The system can decrypt
values from any configured backend.

At server startup, you'll see log entries like these informing you of which encryption providers were configured
successfully:

```
Secrets: GCP credentials and key URI configured from environment: gcp-kms://projects/.../locations/.../keyRings/.../cryptoKeys/...
Secrets: Using 'gcpkms' for encryption (available: gcpkms, nacl)
```

This multi-provider capability enables migration between encryption providers. To migrate, change the
`XNGIN_SECRETS_BACKEND` value and re-encrypt all values.

Choose **one** of these settings for your deployment:

```
XNGIN_SECRETS_BACKEND=noop
XNGIN_SECRETS_BACKEND=nacl
XNGIN_SECRETS_BACKEND=gcpkms
```

### Enabling NaCl Encryption<a name="enabling-nacl-encryption"></a>

1. Set the encryption backend:

   ```
   XNGIN_SECRETS_BACKEND=nacl
   ```

1. Generate a new keyset for your deployment with the CLI tool:

   ```shell
   uv run xngin-cli create-nacl-keyset
   ```

   Store the output in the `XNGIN_SECRETS_NACL_KEYSET` environment variable.

### Enabling Google Cloud KMS<a name="enabling-google-cloud-kms"></a>

Setting up Google Cloud KMS requires these steps:

1. Enable Google Cloud KMS in your account
1. Ensure your Google Cloud principal has KMS administration privileges
1. Create a service account for Evidential
1. Create a Google Cloud KMS keyring and key
1. Grant the service account permission to encrypt/decrypt with the key
1. Configure Evidential with the credentials and key

> **Note:** We don't currently support Google Cloud instance profiles. Please submit a feature request if this is
> important for your deployment.

Configure the environment variables:

```bash
XNGIN_SECRETS_BACKEND=gcpkms
XNGIN_SECRETS_GCP_CREDENTIALS=$SA_CREDENTIALS   # base64 encoded JSON credentials file
XNGIN_SECRETS_GCP_KMS_KEY_URI=$KEY_URI          # GCP KMS Key URI starting with gcp-kms://
```

To test your configuration, use the CLI tools:

```shell
# Encrypt and decrypt a test value
echo secretvalue | \
  uv run --env-file .env xngin-cli encrypt | \
  tee /tmp/ciphertext | \
  uv run --env-file .env xngin-cli decrypt
```

### Debugging GCP KMS<a name="debugging-gcp-kms"></a>

Adjust gRPC logging verbosity with these environment variables:

```
# Enable debug tracing
GRPC_VERBOSITY=debug

# Choose one of these trace configurations
GRPC_TRACE=channel,subchannel,call_stream
GRPC_TRACE=tcp,http,secure_endpoint,transport_security
GRPC_TRACE=call_error,connectivity_state,pick_first,round_robin,glb
```

For more details, see
the [gRPC environment variables documentation](https://github.com/grpc/grpc/blob/master/doc/environment_variables.md).
