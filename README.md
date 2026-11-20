# RepoArmor

A CLI tool for managing [TruffleHog](https://github.com/trufflesecurity/trufflehog) secret scanning results with a rich terminal interface.

RepoArmor is built on top of the open-source [TruffleHog](https://github.com/trufflesecurity/trufflehog) scanner. It wraps the scanning engine with a structured workflow for triaging, tracking, and reporting on exposed secrets such as API keys, tokens, and credentials, and adds auditing commands for CI coverage and repository access.

## Features

- Scan individual repositories or entire GitHub organizations for secrets
- Scan **Azure DevOps**, **Bitbucket Cloud**, and **Bitbucket Server** repositories, including whole Azure DevOps projects, Bitbucket Cloud workspaces, and Bitbucket Server projects
- Scan multiple repositories in a single command (any mix of providers)
- Scan public repositories of individual GitHub users or of all members of GitHub organizations
- Scan Docker images for exposed secrets
- Scan locally cloned repositories
- Audit GitHub repositories and organizations for TruffleHog CI/pull-request coverage, including organization-level required workflows
- Search GitHub Actions workflows across repositories or organizations for specific tools and patterns (literal or regex)
- Branch-scoped pipeline and workflow checks: specific branches, branch patterns, or all branches
- Audit GitHub organization members and repository collaborators to identify internal, external, and unknown identities
- Store and manage findings in a local SQLite database
- Track public vs private repository findings
- Export reports to table, JSON, CSV, or Markdown, and export/import results as JSON to share with a team
- Natural language interface via LLM (OpenAI)
- Statistics dashboard

## Demo

https://github.com/user-attachments/assets/275fe111-b975-4e5d-98d5-dd69295fc747

## Installation

```bash
# Clone the repository
git clone https://github.com/progress/repoarmor.git
cd repoarmor

# Basic installation
pip install -e .          # or: pip3 install -e .

# With LLM chat support (OpenAI)
pip install -e ".[llm]"   # or: pip3 install -e ".[llm]"
```

Verify the installation:

```bash
repoarmor --help
trufflehog --version
```

## Prerequisites

### TruffleHog

```bash
# macOS
brew install trufflehog

# Or using Go
go install github.com/trufflesecurity/trufflehog/v3@latest
```

### GitHub Token (for GitHub Scanning and Audits)

Create a Personal Access Token at https://github.com/settings/tokens with scopes:
- `repo` - Full control of private repositories
- `read:org` - Read organization membership

```bash
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
```

### Azure DevOps Token (for Azure DevOps Scanning)

Create a Personal Access Token at `https://dev.azure.com/{org}/_usersSettings/tokens` with scope:
- `Code` → Read

```bash
export AZURE_DEVOPS_TOKEN="your-pat"
```

### Bitbucket Cloud Credentials (for Bitbucket Cloud Scanning)

Create an App Password at https://bitbucket.org/account/settings/app-passwords/ with scope:
- `Repositories` → Read

```bash
export BITBUCKET_USERNAME="your-username"
export BITBUCKET_APP_PASSWORD="your-app-password"
```

### Bitbucket Server Credentials (for Self-Hosted Bitbucket Scanning)

Create an HTTP Access Token in your Bitbucket Server instance at:
`https://your-server.com/plugins/servlet/access-tokens/manage`

Set permission: **Repository read**

```bash
export BITBUCKET_SERVER_USERNAME="your.email@company.com"
export BITBUCKET_SERVER_TOKEN="your-http-access-token"
```

### OpenAI API Key (Optional — for Chat Feature)

```bash
export OPENAI_API_KEY="sk-xxxxxxxxxxxx"
```

## Supported Providers

| Provider | Single Repo | Org/Project Scan | Environment Variables |
|----------|-------------|------------------|-----------------------|
| **GitHub** | `https://github.com/owner/repo` | `--org mycompany` | `GITHUB_TOKEN` |
| **Azure DevOps** | `https://dev.azure.com/org/project/_git/repo` | `--azure-org org/project` | `AZURE_DEVOPS_TOKEN` |
| **Bitbucket Cloud** | `https://bitbucket.org/workspace/repo` | `--bb-workspace myworkspace` | `BITBUCKET_USERNAME`, `BITBUCKET_APP_PASSWORD` |
| **Bitbucket Server** | `https://server.com/projects/PROJ/repos/repo/browse/` | `--bb-server-project PROJ --bb-server-url URL` | `BITBUCKET_SERVER_USERNAME`, `BITBUCKET_SERVER_TOKEN` |

## Commands Reference

### secret-scan - Scan Repositories

| Option | Short | Description |
|--------|-------|-------------|
| `REPO_URL ...` | | One or more repository URLs or local paths to scan (GitHub, Azure DevOps, Bitbucket Cloud, Bitbucket Server) |
| `--org` | `-o` | GitHub organization to scan all repos |
| `--user` | `-u` | GitHub user whose public repos are scanned |
| `--user-org` | | GitHub org whose members' public repos are scanned |
| `--user-orgs` | | Repeat for multiple orgs (`--user-orgs org1 --user-orgs org2`) |
| `--azure-org` | | Azure DevOps org/project to scan (format: `org/project`) |
| `--bb-workspace` | | Bitbucket Cloud workspace to scan all repos |
| `--bb-server-project` | | Bitbucket Server project key to scan all repos |
| `--bb-server-url` | | Bitbucket Server base URL (required with `--bb-server-project`) |
| `--token` | `-t` | Access token (or use the provider's environment variable) |
| `--batch` | `-b` | File containing repository URLs (one per line) |
| `--depth` | `-d` | Number of commits to scan (defaults to `scanner.default_depth` in the config) |
| `--branch` | | Specific branch to scan |
| `--only-verified` | | Only include secrets verified as active by TruffleHog |
| `--include-archived` | | Include archived repos when scanning an org |
| `--public-only` | | Only scan public repos (excludes private) |
| `--verbose` | `-v` | Verbose output |

Examples:

```bash
# Single repository (GitHub, Azure DevOps, Bitbucket Cloud, Bitbucket Server)
repoarmor secret-scan https://github.com/owner/repo
repoarmor secret-scan https://dev.azure.com/org/project/_git/repo
repoarmor secret-scan https://bitbucket.org/workspace/repo
repoarmor secret-scan https://your-server.com/projects/PROJECT/repos/repo/browse/

# Multiple repositories at once (any mix of providers)
repoarmor secret-scan https://github.com/owner/repo1 https://dev.azure.com/org/project/_git/repo2

# With custom depth and branch
repoarmor secret-scan https://github.com/owner/repo --depth 100 --branch main

# Entire GitHub organization
repoarmor secret-scan --org mycompany
repoarmor secret-scan --org mycompany --public-only
repoarmor secret-scan --org mycompany --include-archived
repoarmor secret-scan --org mycompany --only-verified

# Bitbucket Cloud workspace / Bitbucket Server project
repoarmor secret-scan --bb-workspace myworkspace
repoarmor secret-scan --bb-server-project PLATFORM --bb-server-url https://code.myserver.com

# Batch scan from file
repoarmor secret-scan --batch repos.txt

# Local repository (absolute path, relative path, or file:// URL)
repoarmor secret-scan ~/repos/my-project
repoarmor secret-scan ./my-project
repoarmor secret-scan file:///path/to/my-project
```

### Personal Public Repositories & Docker Image Scanning

Personal repository scans target an individual GitHub user, or the public repositories of members across one or more GitHub organizations. Repositories discovered this way are tracked as **personal repositories** so they can be filtered separately from standard organization scans.

```bash
# A single GitHub user's public repositories
repoarmor secret-scan --user alice

# Public repositories of all members of a GitHub organization
repoarmor secret-scan --user-org mycompany

# Public repositories across multiple GitHub organizations
repoarmor secret-scan --user-orgs org1 --user-orgs org2
```

### secret-scan-docker - Scan Docker Images

```bash
# A single image
repoarmor secret-scan-docker --image nginx:latest

# Multiple images
repoarmor secret-scan-docker --image nginx:latest --image redis:alpine

# Comma-separated list
repoarmor secret-scan-docker --images nginx:latest,redis:alpine
```

Docker image results are stored in the same database and can be queried separately from repository findings.

### db - Retag Existing Database Entries

Existing records can be retagged without scanning again.

```bash
# Retag repositories belonging to a GitHub user as personal
repoarmor db retag-personal --user alice

# Retag repositories belonging to members of an organization
repoarmor db retag-personal --user-org mycompany

# Retag repositories across multiple organizations
repoarmor db retag-personal --user-orgs org1 --user-orgs org2

# Retag existing Docker entries
repoarmor db retag-docker
```

### check-repo-pipeline / check-org-pipeline - TruffleHog Pipeline Coverage

Verify whether **TruffleHog secret scanning is configured in CI and runs on pull requests**. Detection covers both repository-local GitHub Actions workflows and **organization-level required workflows** (using GitHub's per-repository effective rules), and distinguishes workflows that run on pull requests from those that only run on push or on a schedule.

```bash
# Single repository, multiple repositories, or by name within an organization
repoarmor check-repo-pipeline owner/repo
repoarmor check-repo-pipeline owner/repo1 owner/repo2
repoarmor check-repo-pipeline repo1 repo2 --org mycompany
repoarmor check-repo-pipeline https://github.com/owner/repo

# Every repository in an organization
repoarmor check-org-pipeline mycompany
repoarmor check-org-pipeline mycompany --missing-only      # only repos missing PR coverage
repoarmor check-org-pipeline mycompany --include-archived
repoarmor check-org-pipeline mycompany --public-only
```

#### Branch Targeting

By default, checks run against each repository's default branch. Use the options below to check other branches.

| Option | Short | Description |
|--------|-------|-------------|
| `--branch` | `-b` | Check a specific branch. Can be specified multiple times. |
| `--branch-pattern` | | Check branches matching a glob pattern, such as `release/*` |
| `--all-branches` | | Check all branches in the repository |

`--branch-pattern` and `--all-branches` can significantly increase GitHub API calls and require confirmation before proceeding.

```bash
repoarmor check-repo-pipeline owner/repo -b main -b release/2.0
repoarmor check-repo-pipeline owner/repo --branch-pattern 'release/*'
repoarmor check-repo-pipeline owner/repo --all-branches
repoarmor check-repo-pipeline repo-a,repo-b --org myorg -b release/2.0
repoarmor check-org-pipeline myorg --missing-only -b release/2.0
repoarmor check-org-pipeline myorg --branch-pattern 'release/*'
repoarmor check-org-pipeline myorg --all-branches --include-archived
```

### search-repo-workflows / search-org-workflows - Workflow Pattern Search

Search GitHub Actions workflow files (`.github/workflows/*.yml`) for a string or regular expression, for example to find repositories that reference **Trivy, Snyk, TruffleHog**, or other tools. Results show the repository, visibility, whether the pattern was found, the number of workflow files inspected, and the matching workflow file(s).

`search-repo-workflows` options:

| Option | Short | Description |
|--------|-------|-------------|
| `repos` | | One or more repositories as `owner/repo`, bare names with `--org`, or full GitHub URLs (space- or comma-separated) |
| `--pattern` | `-p` | String or regular expression to search for (required) |
| `--regex` | | Treat the pattern as a regular expression |
| `--org` | `-o` | Default organization/owner for bare repository names |
| `--token` | `-t` | GitHub Personal Access Token |

`search-org-workflows` options:

| Option | Short | Description |
|--------|-------|-------------|
| `org` | | GitHub organization to search (required) |
| `--pattern` | `-p` | String or regular expression to search for (required) |
| `--regex` | | Treat the pattern as a regular expression |
| `--found-only` | | Only display repositories where the pattern was found |
| `--include-archived` | | Include archived repositories (excluded by default) |
| `--public-only` / `--include-private` | | Search public repositories only, or include private ones (default) |
| `--token` | `-t` | GitHub Personal Access Token |

Both commands accept the same `--branch` / `-b`, `--branch-pattern`, and `--all-branches` options described under [Branch Targeting](#branch-targeting). By default only the default branch is searched.

```bash
# Repositories
repoarmor search-repo-workflows myorg/myrepo --pattern trivy
repoarmor search-repo-workflows repo-a repo-b --org myorg --pattern trivy
repoarmor search-repo-workflows myorg/myrepo --pattern "snyk|trivy|trufflehog" --regex
repoarmor search-repo-workflows https://github.com/myorg/myrepo --pattern trivy
repoarmor search-repo-workflows owner/repo --pattern trivy -b main -b release/2.0
repoarmor search-repo-workflows owner/repo --pattern trivy --all-branches

# Organizations
repoarmor search-org-workflows mycompany --pattern trivy
repoarmor search-org-workflows mycompany --pattern trivy --found-only
repoarmor search-org-workflows mycompany --pattern "snyk|trivy" --regex --public-only
repoarmor search-org-workflows mycompany --pattern trivy --include-archived
repoarmor search-org-workflows mycompany --pattern trivy --branch-pattern 'release/*'
```

### check-org-members / check-repo-collaborators - Identity Checks

Audit GitHub organization members and repository collaborators to identify **internal, external, and unknown** users. Classification is based on discovered email addresses from the user's public GitHub profile, SCIM data, and Git commit author information. Each discovered address is evaluated against your internal email domains. Users for whom no email address can be found are classified as **unknown**, not external.

Internal email domains are **not configured by default**. Provide at least one, otherwise the command stops with an error:

- On the command line with `-d` / `--internal-domain` (repeatable), or
- Globally in the config file:

```yaml
identity:
  internal_email_domains:
    - example.com
    - example.org
```

`check-org-members` options:

| Option | Short | Description |
|--------|-------|-------------|
| `org` | | GitHub organization to check (required) |
| `--internal-domain` | `-d` | Internal email domain. Can be specified multiple times. |
| `--external-only` | | Display only members classified as external |
| `--token` | `-t` | GitHub PAT with `read:org` scope |

`check-repo-collaborators` options (includes outside collaborators who may not be organization members):

| Option | Short | Description |
|--------|-------|-------------|
| `repo` | | Repository in `owner/repo` format |
| `--internal-domain` | `-d` | Internal email domain. Can be specified multiple times. |
| `--external-only` | | Display only collaborators classified as external |
| `--affiliation` | `-a` | Filter collaborators by `all` (default), `direct`, or `outside` |

```bash
repoarmor check-org-members myorg -d example.com -d example.org
repoarmor check-org-members myorg --internal-domain example.com --external-only

repoarmor check-repo-collaborators myorg/myrepo -d example.com
repoarmor check-repo-collaborators myorg/myrepo -d example.com --affiliation direct
repoarmor check-repo-collaborators myorg/myrepo -d example.com --affiliation outside
```

To simply list the members of an organization (no classification):

```bash
repoarmor list org-users mycompany
```

### list - List Repositories

| Option | Short | Description |
|--------|-------|-------------|
| `--with-stats` | `-s` | Include finding statistics |
| `--personal` | | Only personal repos (scanned via `--user` / `--user-org`) |
| `--org-only` | | Only org repos (scanned via `--org`) |
| `--docker` | | Only Docker image scans |
| `--image` | `-i` | Filter by exact Docker image name |
| `--user` | `-u` | Filter by GitHub username |

```bash
repoarmor list
repoarmor list --with-stats
repoarmor list --personal
repoarmor list --docker
repoarmor list --image nginx:latest
repoarmor list --user alice
```

### secret-findings - View Findings

| Option | Short | Description |
|--------|-------|-------------|
| `--repo` | `-r` | Filter by repository URL or name |
| `--org` | `-o` | Filter by organization/owner name |
| `--type` | `-t` | Filter by detector type (partial match, e.g. `aws`, `github`) |
| `--verified` / `--unverified` | | Only TruffleHog-verified active secrets, or only unverified ones |
| `--limit` | `-n` | Maximum findings to show |
| `--id` | | Show detailed view of a specific finding |
| `--current-only` | | Only secrets still present in the current code (HEAD) |
| `--personal` / `--org-only` | | Only findings from personal repos, or only from org repos |
| `--docker` | | Only findings from Docker image scans |
| `--image` | `-i` | Filter by exact Docker image name |
| `--user` | `-u` | Filter by GitHub username |

```bash
repoarmor secret-findings
repoarmor secret-findings --repo owner/repo
repoarmor secret-findings --org mycompany
repoarmor secret-findings --type aws
repoarmor secret-findings --verified
repoarmor secret-findings --org mycompany --type aws --verified
repoarmor secret-findings --id 42
repoarmor secret-findings --personal
repoarmor secret-findings --docker
repoarmor secret-findings --image nginx:latest
repoarmor secret-findings --user alice
```

### dashboard - Statistics Overview

```bash
repoarmor dashboard
```

Shows total repos, total findings, findings by org, by repo, and by detector type.

### report - Generate Reports

| Option | Short | Description |
|--------|-------|-------------|
| `--format` | `-f` | Output format: `table`, `json`, `csv`, `markdown` |
| `--output` | `-o` | Output file path |
| `--repo` | `-r` | Filter by repository |
| `--org` | | Filter by organization/owner name |
| `--current-only` | | Only secrets still present in the current code (HEAD) |

```bash
repoarmor report
repoarmor report --format json --output report.json
repoarmor report --format csv --output findings.csv
repoarmor report --format markdown --output report.md
```

### export / import - Share Results with Your Team

```bash
# Export results
repoarmor export all-results.json
repoarmor export mycompany-results.json --org mycompany
repoarmor export repo-results.json --repo owner/repo

# Import results from a teammate
repoarmor import team-results.json
repoarmor import team-results.json --dry-run   # preview first
```

Import uses a smart merge and skips findings that already exist, to avoid duplicates.

### chat - Natural Language Interface

Interact with the scanner using natural language via OpenAI (requires the `llm` extra and `OPENAI_API_KEY`).

| Option | Short | Description |
|--------|-------|-------------|
| `PROMPT` | | Natural language command |
| `--model` | `-m` | OpenAI model (default: `gpt-4o`) |
| `--interactive` | `-i` | Interactive chat session |

```bash
# Single command
repoarmor chat "show me all findings"
repoarmor chat "how many AWS keys were found?"
repoarmor chat "show me all findings from private repos"
repoarmor chat "get details on finding 5"
repoarmor chat "give me statistics"
repoarmor chat "scan https://github.com/owner/repo"
repoarmor chat "scan the mycompany organization"   # requires GITHUB_TOKEN

# Specify a model
repoarmor chat -m gpt-4-turbo "summarize the findings"

# Interactive chat mode
repoarmor chat -i
```

### interactive - Interactive Shell

```bash
repoarmor interactive        # prompts for mode
repoarmor interactive llm    # start directly in LLM mode
repoarmor interactive cli    # start directly in CLI mode
```

### config - Configuration

```bash
repoarmor config init   # create default config
repoarmor config show   # display current config
repoarmor config path   # show config file location
```

## Configuration File

Location: `~/.repoarmor/config.yaml`

```yaml
database:
  path: ~/.repoarmor/scanner.db

scanner:
  trufflehog_path: trufflehog
  default_depth: 50
  parallel_scans: 2
  timeout: 300
  only_verified: false

ignore_patterns:
  - "*.test.js"
  - "vendor/*"
  - "node_modules/*"

severity_mapping:
  AWS: high
  GitHub: high
  PrivateKey: high

output:
  default_format: table
  date_format: "%Y-%m-%d %H:%M"

identity:
  internal_email_domains: []

llm:
  provider: openai
  model: null
  api_key: null
```

## Workflows

### Initial Organization Scan

```bash
export GITHUB_TOKEN="ghp_your_token"
repoarmor secret-scan --org mycompany
repoarmor dashboard
repoarmor secret-findings --verified
repoarmor report --format markdown --output security-report.md
```

### Bitbucket Server Project Scan

```bash
export BITBUCKET_SERVER_USERNAME="your.email@company.com"
export BITBUCKET_SERVER_TOKEN="your-token"
repoarmor secret-scan --bb-server-project PLATFORM --bb-server-url https://code.myserver.com
repoarmor dashboard
repoarmor secret-findings --verified
```

### Share Results with Team

```bash
# User A exports
repoarmor export mycompany-scan.json --org mycompany

# User B imports
repoarmor import mycompany-scan.json
```

## Output Colors

| Color | Meaning |
|-------|---------|
| **Red** | Private repository |
| **Yellow** | Internal repository (GitHub Enterprise) |
| **Green** | Public repository |

Findings verified by TruffleHog as active are marked with ✓.

## Data Storage

| Item | Location |
|------|----------|
| Database | `~/.repoarmor/scanner.db` |
| Configuration | `~/.repoarmor/config.yaml` |
| Chat Memory | `~/.repoarmor/chat_memory.json` |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "TruffleHog not found" | Install: `brew install trufflehog` |
| "GitHub API error: 401" | Token invalid/expired - generate new token |
| "GitHub API error: 403" | Rate limit or insufficient permissions |
| Azure DevOps "401 Unauthorized" | PAT expired or wrong org — regenerate at `dev.azure.com/{org}/_usersSettings/tokens` |
| Bitbucket Server clone hangs | Auth token likely needed — set `BITBUCKET_SERVER_USERNAME` and `BITBUCKET_SERVER_TOKEN` |
| Bitbucket Server "Authentication failed" | Verify username and token are correct; `@` in username is auto-encoded |
| "No internal domains" from `check-org-members` / `check-repo-collaborators` | Pass `-d <domain>` or set `identity.internal_email_domains` in the config |
| Database errors after upgrade | Delete `~/.repoarmor/scanner.db` and re-scan |

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md). Commits must be signed off (`git commit -s`).

## Security

For private vulnerability reporting instructions, see [SECURITY.md](./SECURITY.md). Do not open a public issue.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
