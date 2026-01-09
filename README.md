# TypeDB Development

This repository orchestrates development across all TypeDB repositories using git submodules.

## Setup

```bash
# Clone the repository
git clone git@github.com:typedb/typedb-dev.git
cd typedb-dev

# Initialize submodules and configure remotes
tool/repo init

# Or initialize only specific repos (e.g., if you don't have access to private repos)
tool/repo init typedb typeql typedb-driver
```

The `init` command will:
1. Initialize git submodules
2. Rename `origin` remote to `typedb` (upstream)
3. Add your GitHub fork as a remote named after your username

Your GitHub username is detected via `gh` CLI or `git config github.user`.

## Quick Start

```bash
# Check status of all repositories
tool/repo status

# Start working on a feature across multiple repos
tool/repo checkout my-feature typedb typeql typedb-driver

# Switch to an existing remote feature branch
tool/repo switch existing-feature typedb typeql

# Reset repos back to master when done
tool/repo reset typedb typeql typedb-driver

# Fetch latest from all repos
tool/repo fetch
```

## Repository Structure

```
typedb-dev/
├── tool/repo              # Multi-repo management script
├── CLAUDE.md              # AI agent instructions
├── architecture.md        # Technical documentation
│
└── repositories/
    ├── typedb/            # Core database server (Rust)
    ├── typedb-driver/     # Multi-language client drivers
    ├── typeql/            # Query language parser
    ├── typedb-protocol/   # gRPC protocol definitions
    ├── typedb-console/    # Interactive CLI client
    ├── typedb-studio/     # GUI desktop application
    ├── typedb-cluster/    # Clustered TypeDB (private)
    ├── typedb-cloud/      # Cloud platform (private)
    ├── dependencies/      # Shared Bazel build config
    ├── bazel-distribution/ # Package deployment rules
    ├── typedb-behaviour/  # BDD test specifications
    ├── typedb-docs/       # Documentation site
    └── typedb-examples/   # Example projects
```

## Repository Tool

The `tool/repo` script manages submodules and branches.

| Command | Description |
|---------|-------------|
| `init [repos...]` | Initialize submodules, configure remotes, and add user's fork |
| `checkout <feature> <repos...>` | Create/checkout feature branch in specified repos |
| `switch <feature> <repos...>` | Switch to existing feature branch (fetches from remote) |
| `status [--feature <name>]` | Show status of all repos |
| `commit <feature> "<message>"` | Commit changes in all repos on the feature branch |
| `push <fork> <feature>` | Push feature branch to fork and show PR links |
| `reset <repos...>` | Reset repos back to master |
| `fetch [repos...]` | Fetch from remote (all repos if none specified) |
| `list` | List all available repos |

**Note:** The `commit` and `push` commands only operate on submodule repositories. The meta-repository should be committed/pushed manually.

## Dependency Graph

```
                    ┌─────────────────┐
                    │  dependencies   │
                    └────────┬────────┘
                             │
    ┌────────────────────────┼────────────────────────┐
    ▼                        ▼                        ▼
┌──────────────────┐    ┌─────────┐    ┌──────────────────┐
│bazel-distribution│    │ typeql  │    │ typedb-behaviour │
└──────────────────┘    └────┬────┘    └────────┬─────────┘
                             │                  │
┌────────────────┐           │                  │
│ typedb-protocol│           │                  │
└───────┬────────┘           │                  │
        │                    │                  │
        ├────────────────────┼──────────────────┤
        ▼                    ▼                  ▼
┌──────────────┐        ┌──────────┐    ┌──────────────┐
│ typedb-driver│        │  typedb  │    │typedb-console│
└──────┬───────┘        └────┬─────┘    └──────────────┘
       │                     │
       │                     ▼
       │              ┌──────────────┐
       │              │typedb-cluster│
       │              └──────────────┘
       ▼
┌─────────────┐
│ typedb-cloud│
└─────────────┘
```

## Local Development with Bazel Overrides

When developing across multiple repos, override Bazel's git dependencies to use local paths:

**Modify `dependencies/typedb/repositories.bzl`:**

```python
# Before (fetches from git)
def typeql():
    git_repository(
        name = "typeql",
        remote = "https://github.com/typedb/typeql",
        tag = "3.7.0",
    )

# After (uses local path)
def typeql():
    native.local_repository(
        name = "typeql",
        path = "../typeql"
    )
```

**Or use command-line overrides:**

```bash
bazel build //target \
  --override_repository=typeql=$(pwd)/../typeql \
  --override_repository=typedb_protocol=$(pwd)/../typedb-protocol
```

## Git Configuration

- **Remote name**: `typedb` (not `origin`)
- **Default branch**: `master`
- **Merge strategy**: Squash, then rebase before merging PRs

## PR Merge Order

When changes span repos, merge PRs in dependency order:

1. `dependencies` / `bazel-distribution`
2. `typedb-behaviour`
3. `typeql`
4. `typedb-protocol`
5. `typedb`
6. `typedb-cluster`
7. `typedb-driver`
8. `typedb-console` / `typedb-studio`
9. `typedb-cloud`

## Documentation

- **[CLAUDE.md](CLAUDE.md)** - Detailed development guidelines and code conventions
- **[architecture.md](architecture.md)** - Technical documentation for the orchestration system
