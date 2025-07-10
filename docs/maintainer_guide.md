# Maintainer Guide

> Cheatsheet for anyone who edits, deploys, or rolls back the Evidential docs site  
> **Repo:** `github.com/evidential-org/evidential-docs`  
> **Live site:** [https://docs.evidential.dev](https://docs.evidential.dev)

---

## 1. Local setup

Install mkdocs
```bash
pip install mkdocs-material
```

Clone and switch to source branch
```bash
git clone git@github.com:evidential-org/evidential-docs.git
cd evidential-docs
git checkout docs
```

Preview locally
```bash
mkdocs serve
# Served at http://127.0.0.1:8000
```

## 2. Routine content update & deploy

After publishing changes and merging the PR into the `docs` branch, the site will be **automatically deployed** by GitHub Actions via the `deploy-docs.yaml` workflow.

You can monitor deployment progress in the **Actions** tab on GitHub.

The `docs` branch holds the source.  
The `gh-pages` branch holds the built site and the CNAME file.
--force overwrites the previous site build but never touches source content.

## 3. Safe rollback

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


