# Contributing to Documentation

!!! note

    Cheatsheet for anyone who edits, deploys, or rolls back the Evidential docs site

    **Repo:** `github.com/evidential-org/evidential-docs`

    **Live site:** [https://docs.evidential.dev](https://docs.evidential.dev)

______________________________________________________________________

## Local setup

1. Install [uv](https://docs.astral.sh/uv/getting-started/installation/)

1. Clone and switch to source branch

```bash
git clone git@github.com:evidential-org/evidential-docs.git
cd evidential-docs
git checkout docs
```

1. Install dependencies

```
uv sync
```

2. Preview locally

```bash
uv run mkdocs serve
# Default served at http://127.0.0.1:8000; use -a host:port to customize
```

## Routine content update & deploy

After publishing changes and merging the PR into the `docs` branch, the site will be **automatically deployed** by GitHub Actions via the `deploy-docs.yaml` workflow.

You can monitor deployment progress in the **Actions** tab on GitHub.

The `docs` branch holds the source.
The `gh-pages` branch holds the built site and the CNAME file.

## Safe rollback

Revert a broken deploy. Besides reverting to previous or known commit:

### 🏷️ Tag known-good states

Before deploying a major change, tag the source commit so you can easily roll back:

```bash
git tag -a v1.2 -m "Docs v1.2"
git push origin v1.2
```

You can then revert the live site to that state:

```bash
git push -f origin v1.2:gh-pages
```
