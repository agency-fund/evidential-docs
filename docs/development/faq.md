# FAQs

## How do I hide the generated SQL from the logs or console output?<a name="how-do-i-hide-the-generated-sql-from-the-logs-or-console-output"></a>

See the environment variables in [Taskfile.yml](https://github.com/agency-fund/evidential-be/blob/main/Taskfile.yml) and customlogging.py for documentation on how to
control logging.

## psycopg2 module does not install correctly.<a name="psycopg2-module-does-not-install-correctly"></a>

You might see this error:

> Error: pg_config executable not found.
>
> pg_config is required to build psycopg2 from source.

The fix will depend on your operating system.

### Linux

1. Run: `sudo apt install -y libpq-dev python3-dev build-essential` and then re-install dependencies.
1. See [https://www.psycopg.org/docs/install.html](https://www.psycopg.org/docs/install.html).
1. See [https://www.psycopg.org/docs/faq.html](https://www.psycopg.org/docs/faq.html).

### OSX<a name="osx"></a>

Run `brew install postgresql@17`.
