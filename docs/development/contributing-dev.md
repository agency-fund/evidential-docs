# Contributing to Development

Thanks for your interest in improving Evidential! This page walks through the full
contribution workflow, from fork to merged pull request.

## Before you start: small vs. large changes

**Small changes** — typo fixes, documentation improvements, or small bug fixes with
tests — are welcome as direct pull requests. No need to ask first.

**Larger changes** — new features, API changes, refactors, or anything touching the
experiment statistics — please [reach out to the team](../contact.md) *before* you start
coding. We’re happy to brainstorm use‑cases, sanity‑check your design, and make sure your
effort fits the project’s roadmap before you invest serious time in it. Opening an early
**draft pull request** is another good way to get direction while the change is still
cheap to steer.

## Community‑driven vision

This project will only thrive in the long run if it remains community‑driven.
Your ideas, code, and feedback are essential to its success.

## The repositories

| Repository | What it is |
| --- | --- |
| [evidential-be](https://github.com/agency-fund/evidential-be) | The backend: a FastAPI API server, stats engine, and task queue (Python) |
| [evidential-fe](https://github.com/agency-fund/evidential-fe) | The frontend: a Next.js admin web app (TypeScript) |
| [evidential-docs](https://github.com/agency-fund/evidential-docs) | This documentation site (MkDocs) — see [Contributing to Documentation](../contributing-docs.md) |

## 1. Fork and clone

Click **Fork** on the repository you want to change, then clone *your fork* (you won’t
have push access to the upstream repos):

```bash
git clone git@github.com:<your-github-handle>/evidential-be.git
cd evidential-be
```

## 2. Set up your development environment

Follow [Getting Started](getting-started-dev.md) to install prerequisites, run the test
suite, and start a local server. Don’t skip the last step there — installing the
pre‑commit hooks (`uv run pre-commit install`) keeps formatting and lint issues from ever
reaching your pull request.

## 3. Make your changes

Create a feature branch from `main`:

```bash
git checkout -b feature/my-awesome-idea
```

Write code that matches the style of the surrounding code, and update or add tests for
any behavior you change. Commit with clear, descriptive messages.

## 4. Verify before you push

**Backend (`evidential-be`):**

```bash
task test-airplane              # unit tests, no external services needed
uv run pre-commit run --all-files
uv run mypy src                 # type checking
```

**Frontend (`evidential-fe`):**

```bash
corepack pnpm run lint          # eslint
npx tsc --noEmit                # type checking
```

GitHub Actions runs the same checks on every pull request, so running them locally first
saves you a review round‑trip.

## 5. Open a pull request

!!! tip "Keep pull requests focused"

    One logical change per PR. Small, reviewable PRs get merged much faster than
    grab‑bags — if you find yourself fixing an unrelated bug along the way, split it
    into its own PR.

1. Push the branch to your fork: `git push -u origin feature/my-awesome-idea`
1. Open a **pull request** against `main` on the upstream repository.
1. Fill in the PR template — including the checklist — and reference any related issues.
1. A maintainer will review your PR, request changes if needed, and merge when ready.
   CI must be green before merge.

## Request a feature

If you don't feel like contributing, simply request a feature by [contacting us](../contact.md)

## Reporting security issues

Please **don’t** report potential security vulnerabilities in public pull requests or
issues. [Contact us](../contact.md) privately instead so we can fix the problem before
it’s disclosed.

## Licensing

The backend ([evidential-be](https://github.com/agency-fund/evidential-be)) is licensed
under Apache 2.0. By contributing, you agree that your contributions will be licensed
under the same terms as the repository you contribute to.

Thanks for helping us build something better—together!
