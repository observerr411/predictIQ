# Contributing to PredictIQ

## Branch Protection Rules

The `main` branch is protected. The following rules are enforced:

- **Pull request required** — direct pushes to `main` are not allowed; all changes must go through a PR.
- **CI must pass** — all status checks in `.github/workflows/` must succeed before a PR can be merged.
- **At least 1 approval required** — a PR must receive at least one approving review from a team member.
- **No force pushes** — `git push --force` to `main` is disabled.
- **No branch deletion** — `main` cannot be deleted.

These rules are configured in the repository settings under **Settings → Branches → Branch protection rules**.

## Code Ownership

Sensitive paths have designated reviewers defined in [`.github/CODEOWNERS`](.github/CODEOWNERS). GitHub automatically requests a review from the relevant owner when a PR touches those paths.

## Development Workflow

1. Create a feature branch from `main`: `git checkout -b feat/your-feature`
2. Make your changes and commit with a descriptive message.
3. Open a pull request against `main`.
4. Ensure all CI checks pass and at least one reviewer approves.
5. Merge using **Squash and merge** to keep the history clean.

## Secrets and Environment Variables

- Never commit real secrets or credentials.
- Copy `services/api/.env.example` to `services/api/.env` and fill in real values locally. The `.env` file is gitignored.
- All placeholder values in `.env.example` are intentionally empty or clearly fake.
- Gitleaks runs on every push to detect accidental secret commits (see `.gitleaks.toml`).
