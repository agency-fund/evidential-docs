Documentation repository for `evidential`.

We use [MkDocs](https://www.mkdocs.org/getting-started/) to build our project docs, along with the
[Material](https://squidfunk.github.io/mkdocs-material/reference/) theme and [custom
overrides](https://squidfunk.github.io/mkdocs-material/customization/#extending-the-theme).  To test
and changed docs locally, you can run:
```
uv sync
uv run mkdocs serve [-a localhost:8001]
```