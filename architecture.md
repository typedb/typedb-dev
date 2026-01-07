# TypeDB Development Architecture

Technical documentation for the multi-repo orchestration system.

## Directory Structure

```
typedb-dev/
├── .git/
├── .gitmodules              # Submodule configuration
├── CLAUDE.md                # AI agent instructions
├── architecture.md          # This file
├── tool/
│   └── repo                 # Repository management script
│
└── repositories/            # All submodules
    │ # Primary repositories (releasable)
    ├── typedb/              # Core database server
    ├── typedb-driver/       # Multi-language drivers
    ├── typeql/              # Query language parser
    ├── typedb-protocol/     # gRPC protocol definitions
    ├── typedb-console/      # CLI client
    ├── typedb-studio/       # GUI application
    │
    │ # Private repositories (commercial)
    ├── typedb-cluster/      # Clustered/distributed TypeDB
    ├── typedb-cloud/        # Cloud platform management
    ├── typedb-cloud-infrastructure/  # Cloud infra configs
    │
    │ # Secondary repositories (supporting)
    ├── dependencies/        # Shared Bazel config
    ├── bazel-distribution/  # Deployment rules
    ├── typedb-behaviour/    # BDD test specs
    ├── typedb-docs/         # Documentation
    ├── typedb-examples/     # Examples
    └── typedb-web/          # Website
```

## Repository Dependencies

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
       │
       ▼
┌─────────────────────────────────────────┐
│ typedb-cloud (updated infrequently)     │
│   └─► typedb-cloud-infrastructure       │
└─────────────────────────────────────────┘
```

### Dependency Details

| Repository | Depends On |
|------------|------------|
| `typedb` | dependencies, bazel-distribution, typeql, typedb-protocol, typedb-behaviour |
| `typedb-driver` | dependencies, typedb-protocol, typedb-behaviour |
| `typedb-console` | dependencies, typedb-driver, typeql |
| `typeql` | dependencies, typedb-behaviour |
| `typedb-protocol` | dependencies |
| `typedb-behaviour` | dependencies |
| `typedb-studio` | dependencies |
| `typedb-cluster` | dependencies, bazel-distribution, typedb |
| `typedb-cloud` | dependencies, typedb-driver (Maven), typeql (Maven), typedb-cloud-infrastructure |
| `typedb-cloud-infrastructure` | (standalone infrastructure configs) |

## Git Configuration

### Remote Setup

All submodules use:
- **Remote name**: `typedb`
- **URL format**: `git@github.com:typedb/<repo>.git`

This differs from the default `origin` to clearly indicate the upstream source.

### Submodule Mode

Submodules use detached HEAD mode (git default). The `tool/repo` script manages branch switching for active development.

## Tool Implementation

### `tool/repo` Script

Bash script that operates on submodules. Key behaviors:

1. **Branch validation**: Ensures repos are either on feature branch or master
2. **Remote fetching**: `switch` command fetches from remote if branch not found locally
3. **Color output**: TTY-aware colored output for terminal readability
4. **Error handling**: Clear error messages with usage hints

### Command Details

#### `checkout <feature> <repos...>`

Creates feature branch if it doesn't exist, switches to it if it does.

```
Input: checkout my-feature typedb typeql
Output:
  ✓ typedb - created and switched to my-feature
  ✓ typeql - created and switched to my-feature
```

#### `switch <feature> <repos...>`

Switches to existing branch. Fetches from remote if not found locally.

```
Input: switch existing-feature typedb
Output:
  ✓ typedb - fetched and switched to existing-feature
```

#### `status [--feature <name>]`

Shows tabular status of all repos. Optional feature filter shows only repos on that feature or master.

```
REPO                 BRANCH               CLEAN    AHEAD/BEHIND
typedb               my-feature           yes      +2/-0
typeql               master               yes      -
```

#### `reset <repos...>`

Switches repos back to master branch.

```
Input: reset typedb typeql
Output:
  ✓ typedb - reset to master (was: my-feature)
  ✓ typeql - reset to master (was: my-feature)
```

#### `fetch [repos...]`

Fetches from remote. All repos if none specified.

#### `list`

Lists all known repos grouped by primary/secondary.

## Worktree Usage

Git worktrees enable concurrent work on different features:

```
/path/to/dev/
├── typedb-dev/              # Main worktree (master)
├── typedb-dev-feature-a/    # Worktree for feature-a
└── typedb-dev-feature-b/    # Worktree for feature-b
```

### Creating a Worktree

```bash
# From main repo
git worktree add ../typedb-dev-feature-a -b feature-a

# Initialize submodules
cd ../typedb-dev-feature-a
git submodule update --init

# Configure remotes (submodules default to 'origin')
for repo in */; do
  if [ -d "$repo/.git" ] || [ -f "$repo/.git" ]; then
    git -C "$repo" remote rename origin typedb 2>/dev/null || true
  fi
done
```

### Worktree Workflow

1. Create worktree for the meta-repo
2. Initialize submodules in new worktree
3. Rename remotes to `typedb`
4. Use `tool/repo checkout` to switch submodules to feature branches
5. Work on feature
6. Remove worktree when done: `git worktree remove ../typedb-dev-feature-a`

## Build System

All repositories use Bazel with language-specific tooling:

| Language | Build Stack |
|----------|-------------|
| Rust | Bazel + Cargo (Cargo.toml auto-generated) |
| Java | Bazel + Maven |
| Python | Bazel + pip |
| TypeScript | Bazel + pnpm |

### Local Repository Overrides

Bazel repositories normally fetch dependencies from git. For local development, modify the repository rules to use local paths.

#### Modifying `repositories.bzl`

Repository definitions live in `dependencies/typedb/repositories.bzl`. Change `git_repository` to `native.local_repository`:

**Before (fetches from git):**
```python
def typeql():
    git_repository(
        name = "typeql",
        remote = "https://github.com/typedb/typeql",
        tag = "3.7.0",
    )
```

**After (uses local path):**
```python
def typeql():
    native.local_repository(
        name = "typeql",
        path = "../typeql"
    )
```

The path is relative to the WORKSPACE root of the consuming repository. Since all repos are siblings under `repositories/`, the path `../typeql` resolves correctly.

#### Repository Name Mapping

| Local Directory | Bazel Repository Name | Relative Path |
|-----------------|----------------------|---------------|
| `repositories/typeql` | `typeql` | `../typeql` |
| `repositories/typedb-protocol` | `typedb_protocol` | `../typedb-protocol` |
| `repositories/typedb-behaviour` | `typedb_behaviour` | `../typedb-behaviour` |
| `repositories/typedb-driver` | `typedb_driver` | `../typedb-driver` |
| `repositories/dependencies` | `vaticle_dependencies` | `../dependencies` |
| `repositories/bazel-distribution` | `vaticle_bazel_distribution` | `../bazel-distribution` |

#### Alternative: `--override_repository` Flag

For temporary overrides without modifying files (requires absolute paths):

```bash
bazel build //target --override_repository=typeql=/abs/path/to/repositories/typeql
```

### Common Build Patterns

```bash
# TypeDB server
cd repositories/typedb
bazel build //:assemble-typedb-all
cargo test

# TypeQL parser
cd repositories/typeql
bazel build //rust:typeql
bazel test //rust:typeql_unit_tests

# TypeDB driver (Rust)
cd repositories/typedb-driver
bazel build //rust:typedb_driver

# TypeDB console
cd repositories/typedb-console
bazel build //:console-native
cargo build --release
```

## Testing

### BDD Tests

Shared test specifications in `typedb-behaviour` repository. Used by:
- `typedb` - Server behavior tests
- `typedb-driver` - Driver behavior tests
- `typeql` - Parser behavior tests

### Running Tests

```bash
# TypeDB server tests
cd repositories/typedb
cargo test
bazel test //...

# TypeDB driver tests
cd repositories/typedb-driver
bazel test //rust/tests/behaviour/driver:test_query
```

## Release Process

Releases are coordinated across repositories in dependency order:

1. `typedb-behaviour` (if changed)
2. `typeql`
3. `typedb-protocol`
4. `typedb`
5. `typedb-driver`
6. `typedb-console`

See individual repository documentation for release procedures.
