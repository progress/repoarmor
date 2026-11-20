# Contributing to RepoArmor

Thank you for your interest in contributing! Contributions of all kinds are welcome, including bug reports, feature requests, documentation, and code.

## Reporting issues

- Search existing issues before opening a new one.
- Include the command you ran, the output or error, your OS, and your Python and TruffleHog versions.
- **Do not report security vulnerabilities as issues.** See [SECURITY.md](./SECURITY.md).

## Development setup

```bash
git clone <your-fork-url>
cd repoarmor
pip install -e ".[dev]"
pytest
```

Python 3.10 or newer is required. [TruffleHog](https://github.com/trufflesecurity/trufflehog) must be installed separately to run scans, but it is not needed to run the unit tests, which need no GitHub token, other credentials, or network access.

## Project layout

| Path | Purpose |
|---|---|
| `repoarmor/cli/` | Typer commands and input validation |
| `repoarmor/scanner/` | TruffleHog runner and one API client per provider (GitHub, Azure DevOps, Bitbucket Cloud and Server) |
| `repoarmor/db/` | SQLAlchemy models and queries for the local SQLite database |
| `repoarmor/reports/` | Report generation (table, JSON, CSV, Markdown) |
| `repoarmor/ui/` | Rich console output, prompts, and tables |
| `repoarmor/llm/` | Optional natural language interface |
| `tests/` | Unit tests |

To add a provider, add an API client under `repoarmor/scanner/` modeled on the existing ones, wire it into the scan commands, and add tests.

## Testing

- Tests must not call real provider APIs or run TruffleHog. Mock HTTP calls (for example `monkeypatch.setattr("repoarmor.scanner.github_api.requests.request", ...)`, as in `tests/test_github_api.py`).
- Tests must not read or write your real RepoArmor data directory. Use `tmp_path` or monkeypatching.
- The checked-in workflows run TruffleHog, Black Duck and Polaris on pull requests, and CI runs `pytest` on Python 3.10 to 3.13 and builds the package. Black Duck and Polaris run only for branches in the main repository, so they appear as skipped on fork and Dependabot pull requests. Run `pytest` locally after installing the dev extras.

## Contribution process

1. Fork the repository and create a branch from `main`.
2. Make your change and add or update tests. New code should have unit test coverage.
3. Run `pytest` and make sure it passes.
4. Commit with a sign-off (see below).
5. Rebase on the latest `main`, push to your fork, and open a pull request describing what changed and why.
6. A maintainer will review your pull request. CI must pass before it can be merged.

Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages and PR titles (`feat:`, `fix:`, `docs:`, `ci:`, `chore:`; use `!` for breaking changes, for example `feat!:`). Pull requests are squash-merged; the sign-offs on your individual commits are still checked.

## Developer Certificate of Origin (DCO)

All commits must be signed off to certify that you wrote the code or otherwise have the right to submit it under this project's license (Apache 2.0). Add the sign-off with `-s`:

```bash
git commit -s -m "fix: describe your change"
```

This adds a line to the commit message:

```
Signed-off-by: Your Name <your.email@example.com>
```

If you forgot, fix the latest commit with `git commit --amend -s`, or sign off several commits with `git rebase --signoff main`, then force-push your branch.

By signing off you agree to the [Developer Certificate of Origin](https://developercertificate.org/).

## License

By contributing, you agree that your contributions are licensed under the [Apache License 2.0](./LICENSE).
