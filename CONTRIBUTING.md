# Contributing to PredictIQ

Thank you for your interest in contributing! This guide covers everything you need to get started.

## Table of Contents

- [Development Setup](#development-setup)
- [Branch Naming](#branch-naming)
- [Commit Conventions](#commit-conventions)
- [Pull Request Process](#pull-request-process)
- [Running Tests](#running-tests)
- [Code Style](#code-style)

---

## Development Setup

### Prerequisites

- [Rust](https://rustup.rs/) (stable toolchain)
- [Node.js](https://nodejs.org/) 18+
- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- [PostgreSQL](https://www.postgresql.org/) 15+ (or use the provided Docker Compose stack)

### Getting Started

```bash
# Clone the repository
git clone https://github.com/solutions-plug/predictIQ.git
cd predictIQ

# Start backing services (Postgres, Redis, etc.)
docker compose up -d

# API service
cd services/api
cp .env.example .env          # fill in required values
cargo build

# Frontend
cd frontend
cp .env.example .env.local    # fill in required values
npm install
npm run dev

# TTS service
cd services/tts
npm install
npm run dev
```

---

## Branch Naming

Use the following prefixes:

| Prefix | Purpose |
|--------|---------|
| `feat/` | New feature |
| `fix/` | Bug fix |
| `chore/` | Maintenance, dependency updates |
| `docs/` | Documentation only |
| `refactor/` | Code refactoring without behaviour change |
| `perf/` | Performance improvement |
| `ci/` | CI/CD changes |

Examples: `feat/market-resolution`, `fix/rate-limit-header`, `docs/contributing`

---

## Commit Conventions

This project uses **[Conventional Commits](https://www.conventionalcommits.org/)**.  
The CHANGELOG is **auto-generated** from commit messages via [git-cliff](https://git-cliff.org/) — do not edit `CHANGELOG.md` manually.

### Format

```
<type>(<scope>): <short description>

[optional body]

[optional footer(s)]
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | A new feature (triggers a minor version bump) |
| `fix` | A bug fix (triggers a patch version bump) |
| `docs` | Documentation changes only |
| `chore` | Build process, dependency updates, tooling |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `ci` | CI/CD configuration changes |

Append `!` after the type/scope for **breaking changes** (triggers a major version bump):

```
feat(api)!: remove deprecated /v0 endpoints
```

### Examples

```
feat(markets): add oracle result caching
fix(newsletter): handle duplicate subscription gracefully
docs(api): document rate-limit response headers
chore(deps): bump axum to 0.7.5
```

---

## Pull Request Process

1. Fork the repository and create your branch from `main`.
2. Ensure all tests pass locally (see [Running Tests](#running-tests)).
3. Keep commits focused — one logical change per commit.
4. Open a PR against `main` with a clear title following the commit convention.
5. Fill in the PR description:
   - **What** changed and **why**
   - How to test the change
   - Any breaking changes or migration steps
6. Link related issues using `Closes #<issue>` in the PR description.
7. At least one approval is required before merging.
8. Squash-merge is preferred to keep the history clean.

### PR Checklist

- [ ] Branch is up to date with `main`
- [ ] Commit messages follow Conventional Commits
- [ ] Tests added or updated for the change
- [ ] Documentation updated if behaviour changed
- [ ] No secrets or credentials committed
- [ ] `CHANGELOG.md` **not** manually edited

---

## Running Tests

### API (Rust)

```bash
cd services/api
cargo test
```

### Frontend (Next.js)

```bash
cd frontend
npm test              # unit tests (Jest)
npm run test:e2e      # end-to-end tests (Playwright)
```

### TTS Service

```bash
cd services/tts
npm test
```

### Smart Contracts

```bash
cd contracts/predict-iq
make test
```

---

## Code Style

### Rust

- Follow `rustfmt` defaults — run `cargo fmt` before committing.
- Lint with `cargo clippy -- -D warnings`.

### TypeScript / JavaScript

- ESLint and Prettier are configured in the `frontend/` directory.
- Run `npm run lint` and `npm run format` before committing.

### YAML / Markdown

- Keep line length reasonable (80–100 characters).
- Use 2-space indentation for YAML.

---

## Questions?

Open an issue or start a discussion on [GitHub](https://github.com/solutions-plug/predictIQ/issues).
