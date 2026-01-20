# TypeDB Development Meta-Repository

This repository orchestrates development across all TypeDB repositories using git submodules.

## Development Philosophy

We put a huge emphasis on **architectural correctness**, **cohesion** with other code in the same repository, **readability**, and **correctness**. Always aim for **maximum simplicity**, even if it takes multiple iterations of work, cleanup, and attempts - the first solution typically isn't the best one!

### Guidelines for AI Agents

1. **Read documentation first** - Before starting work in any repository, read the `README.md` and `architecture.md` files to understand the codebase structure and conventions. Update these files if your changes affect the documented architecture or setup.

2. **Plan first** - Before implementing, explore existing patterns in the repository and validate your design against them. When sensible, feel free to introduce or suggest other accepted best-practices.

3. **Iterate on design** - Don't settle for the first working solution. Review, refactor, and simplify until the code feels natural alongside existing patterns.

4. **Review completeness** - When a task seems done, review the complete change for correctness, clarity, and cohesion with existing patterns. We don't want ad-hoc patterns to arise in different parts of the codebase.

5. **Verify with tests** - Where possible, run tests in a subagent to verify the solution works correctly.

6. **Use working memory for complex tasks** - When working on a problem with multiple steps and components, create a `<feature-name>-plan.md` file in the repository root. Record progress, design decisions, and open questions there as high-level working memory.

7. **Keep documentation current** - If your changes affect architecture, APIs, or setup procedures, update the relevant `README.md` and `architecture.md` files as part of the change.

### Cross-Repository Validation

When implementing a feature or change, always check if other repositories may need updates. Flag any cross-repo impacts to the developer once the feature is complete.

**Always validate:**

1. **Behaviour tests** (`typedb-behaviour`) - Do BDD test specifications need new scenarios or updates?

2. **Documentation** (`typedb-docs`) - **Critical**: Check corresponding documentation for correctness and completeness, even if behavior hasn't changed. Our docs may not be updated or complete yet. Consider edge cases and ramifications of the feature.

3. **Protocol consumers** - If gRPC or HTTP behaviors/protocols change, validate whether these need updates:
   - `typedb-driver` (all language bindings)
   - `typedb-console`
   - `typedb-studio`
   - `typedb-examples`

4. **Downstream dependencies** - Check the dependency graph and consider if changes propagate.

### Quality Checklist

Before considering a change complete:
- [ ] Does it follow existing architectural patterns in the repository?
- [ ] Is it the simplest solution that solves the problem?
- [ ] Does naming match conventions used elsewhere?
- [ ] Are error cases handled consistently with other code?
- [ ] Would a reviewer unfamiliar with the change understand it easily?
- [ ] Have cross-repository impacts been identified and flagged?

### Meta-Repository Portability

When modifying this `typedb-dev` repository itself (scripts, tooling, configuration):

- **Cross-platform**: All scripts and tools must work on both **Linux** and **macOS**
- **Location-agnostic**: Never hardcode absolute paths. Anyone should be able to clone this repository to any location and have it work immediately
- **Self-contained**: Use paths relative to the repository root (e.g., `$SCRIPT_DIR`, `$(dirname "$0")`)
- **Standard tools**: Prefer POSIX-compatible shell constructs; avoid bash-specific features where possible
- **No environment assumptions**: Don't assume specific environment variables are set (except standard ones like `$HOME`)
- **Claude settings**: Keep `.claude/settings.local.json` path and user agnostic - use relative paths (e.g., `./tool/script`) so settings work for any team member cloning the repository

## Quick Start

```bash
# Initialize submodules and configure remotes (first time setup)
tool/repo init

# Check status of all repositories
tool/repo status

# Start working on a feature across multiple repos
tool/repo checkout my-feature typedb typeql typedb-driver

# Switch to an existing feature branch
tool/repo switch existing-feature typedb typeql

# Reset repos back to master when done
tool/repo reset typedb typeql typedb-driver

# Fetch latest from all repos
tool/repo fetch
```

## Repository Tool

The `tool/repo` script manages submodules and branches.

### Commands

| Command | Description |
|---------|-------------|
| `init [repos...]` | Initialize submodules, configure remotes, and add user's fork |
| `checkout <feature> <repos...>` | Create/checkout feature branch in specified repos |
| `switch <feature> <repos...>` | Switch to existing feature branch (fetches from remote if needed) |
| `status [--feature <name>]` | Show status of all repos |
| `commit <feature> "<message>"` | Commit changes in all repos on the feature branch |
| `push <fork> <feature>` | Push feature branch to fork and show PR links |
| `reset <repos...>` | Reset repos back to master |
| `fetch [repos...]` | Fetch from remote (all repos if none specified) |
| `list` | List all available repos |

### Examples

```bash
# Start a new feature touching typedb and typeql
tool/repo checkout add-new-query-type typedb typeql

# Check which repos are on the feature branch
tool/repo status --feature add-new-query-type

# Commit changes across all repos on the feature branch
tool/repo commit add-new-query-type "Add new query type support"

# Push to your fork and get PR links
tool/repo push myusername add-new-query-type

# Fetch latest changes
tool/repo fetch typedb typeql

# Done with feature, reset to master
tool/repo reset typedb typeql
```

**Note:** The `commit` command only commits to submodule repositories on the specified feature branch. The meta-repository (`typedb-dev`) should be committed manually to avoid accidental submodule pointer updates.

## Slash Commands

Custom Claude Code commands available in this repository (invoke with `/command-name`):

| Command | Description |
|---------|-------------|
| `/review [files]` | Review changes for architectural correctness, simplicity, clarity, and edge cases |
| `/cross-repo-impact [repo]` | Analyze changes and identify cross-repository impacts |
| `/sync-local-deps <repo>` | Generate local Bazel dependency overrides for development |

### Review Command

The `/review` command performs a comprehensive code review based on TypeDB development principles:

- **Architectural correctness** - Follows existing patterns and module boundaries
- **Simplicity** - No over-engineering, premature abstractions, or unnecessary complexity
- **Clarity** - Readable, well-named, self-documenting code
- **Correctness** - Proper error handling and edge case coverage:
  - Empty/null inputs
  - Boundary conditions
  - Concurrent access
  - Resource cleanup
  - Invalid state transitions
- **Cross-repo impact** - Identifies changes that may affect other repositories

```bash
# Review all uncommitted changes
/review

# Review specific files
/review src/query/executor.rs
```

## Repositories

### Primary (Releasable)

| Repository | Description |
|------------|-------------|
| `typedb` | Core database server (Rust) |
| `typedb-driver` | Multi-language client drivers |
| `typeql` | Query language parser |
| `typedb-protocol` | gRPC protocol definitions |
| `typedb-console` | Interactive CLI client |
| `typedb-studio` | GUI desktop application |

### Private (Commercial)

| Repository | Description |
|------------|-------------|
| `typedb-cluster` | Clustered/distributed TypeDB (Rust) |
| `typedb-cloud` | Cloud platform management (Kotlin) |
| `typedb-cloud-infrastructure` | Cloud infrastructure configs (Terraform/K8s) |

### Secondary (Supporting)

| Repository | Description |
|------------|-------------|
| `dependencies` | Shared Bazel build configuration |
| `bazel-distribution` | Package deployment rules |
| `typedb-behaviour` | BDD test specifications |
| `typedb-benchmark` | Performance benchmarks |
| `typedb-docs` | Documentation site |
| `typedb-examples` | Example projects |
| `typedb-web` | Website |

## Multi-Repo Feature Workflow

When implementing a feature that spans multiple repositories:

1. **Identify affected repos** - Determine which repos need changes
2. **Create feature branches** - Use `tool/repo checkout <feature> <repos...>`
3. **Implement changes** - Work in each repo as needed
4. **Test integration** - Ensure changes work together
5. **Create PRs** - Submit PRs in dependency order (see below)
6. **Reset when done** - Use `tool/repo reset <repos...>`

### Dependency Order for PRs

When changes span repos, merge PRs in this order:

1. `dependencies` / `bazel-distribution` (if changed)
2. `typedb-behaviour` (if test specs changed)
3. `typeql` (parser changes)
4. `typedb-protocol` (protocol changes)
5. `typedb` (server changes)
6. `typedb-cluster` (cluster server, depends on typedb)
7. `typedb-driver` (driver changes)
8. `typedb-console` / `typedb-studio` (client changes)
9. `typedb-cloud` (cloud platform, updated infrequently, depends on driver)

## Git Configuration

- **Remote name**: `typedb` (not `origin`)
- **URLs**: SSH format (`git@github.com:typedb/<repo>.git`)
- **Default branch**: `master`
- **Branch base**: Feature branches fork from `typedb/master`. When looking for diffs, use `typedb/master` as the base (e.g., `git diff typedb/master...HEAD`), or find the nearest ancestor branch if the feature branch is based on another branch.

### Pushing Changes

**Always push to your fork remote, never directly to `typedb`.**

1. Add your fork as a remote (e.g., `git remote add <username> git@github.com:<username>/<repo>.git`)
2. Push feature branches to your fork: `git push <username> <branch-name>`
3. Create PRs from your fork to the `typedb` remote

Use `tool/repo push <fork-remote> <feature>` to push all repos on a feature branch and get PR links.

### Base Branches

Each repository should specify its base branch in its `CLAUDE.md` file. If a repo's `CLAUDE.md` doesn't specify a base branch, flag this to the user so it can be updated.

| Repository | Base Branch |
|------------|-------------|
| Most repos | `master` |
| `typedb-docs` | `3.x-development` |

### Merge Strategy

Always **squash, then rebase** before merging PRs:
1. Squash all commits into a single logical commit
2. Rebase onto the latest `master`
3. Merge (fast-forward when possible)

### PR Description Format

Write meaningful PR descriptions with two sections:

**Change & Motivation**
- Why the change was needed (problem/context)
- What changed from a product/user perspective
- Impact on behavior or functionality

**Implementation**
- Technical approach taken
- Key files/modules modified
- Notable design decisions or trade-offs

Example:
```markdown
## Change & Motivation
Users were experiencing slow query performance when filtering by multiple attributes.
This PR optimizes the query planner to use index intersection for multi-attribute filters.

## Implementation
- Added `IndexIntersectionPlanner` in `query/planner/`
- Modified `QueryOptimizer::plan()` to detect intersectable index patterns
- Updated cost model to prefer intersection over sequential scans
- Benchmarks show 3-5x improvement for 2+ attribute filters
```


## Local Bazel Overrides

When developing across multiple repos, you can override Bazel's git dependencies to use local paths. This allows testing changes across repos without pushing to git.

### Modifying Repository Rules

In most repositories, the `dependencies/typedb/repositories.bzl` file, change `git_repository` to `native.local_repository`:

**Before (git):**
```python
def typeql():
    git_repository(
        name = "typeql",
        remote = "https://github.com/typedb/typeql",
        tag = "3.7.0",
    )
```

**After (local):**
```python
def typeql():
    native.local_repository(
        name = "typeql",
        path = "../typeql"
    )
```

The relative path `../typeql` works because it's relative to the WORKSPACE root of the repo being built.

### Common Override Patterns

| Building | Repository | Local Path (relative to WORKSPACE) |
|----------|------------|-----------------------------------|
| `typedb` | `typeql` | `../typeql` |
| `typedb` | `typedb_protocol` | `../typedb-protocol` |
| `typedb` | `typedb_behaviour` | `../typedb-behaviour` |
| `typedb-driver` | `typedb_protocol` | `../typedb-protocol` |
| `typedb-driver` | `typedb_behaviour` | `../typedb-behaviour` |
| `typedb-console` | `typeql` | `../typeql` |
| `typedb-console` | `typedb_driver` | `../typedb-driver` |
| `typedb-cluster` | `typedb` | `../typedb` |
| `typedb-cluster` | `typedb_dependencies` | `../dependencies` |

## Build Commands

**Installing Bazel:**

All repositories use Bazel via Bazelisk (version manager). Install with:
```bash
npm install -g @bazel/bazelisk
```
Or see https://github.com/bazelbuild/bazelisk for other installation methods.

**Common patterns:**

```bash
# TypeDB server
cd repositories/typedb && bazel build //:assemble-typedb-all

# TypeQL parser
cd repositories/typeql && bazel build //rust:typeql

# TypeDB driver
cd repositories/typedb-driver && bazel build //rust:typedb_driver
```

## Code Style & Development Principles

These principles are extracted from the TypeDB codebases and should be followed when contributing.

### General Guidelines (All Languages)

**Formatting:**
- Maximum line width: 120 characters
- Explicit imports (no wildcards)
- Group imports: standard library → third-party → internal

**Naming:**
- Classes/Types: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Boolean methods: `is_*` / `has_*` prefix (language-appropriate casing)

**Error Handling:**
- Structured error codes with component prefix (e.g., `SRO-001`, `JDR-008`)
- Error chaining to preserve root cause
- Descriptive error messages with context

**Documentation:**
- License headers on all files (see License section below)
- Minimal inline docs; code should be self-documenting
- TODO comments with context for incomplete features

**License Headers:**
- **MPL-2.0**: High-IP repositories (`typedb`, `typeql`)
- **Apache 2.0**: Embeddable/user-consumable repositories (`typedb-driver`)
- **Commercial**: Private repositories (`typedb-cloud`, `typedb-cluster`)

**Testing:**
- BDD tests via Cucumber/Gherkin specs in `typedb-behaviour`
- Unit tests alongside source code
- Integration tests as separate binaries/modules

**Key Principles:**
1. **Type safety over convenience**
2. **Errors are values** with structured codes
3. **Clarity over conciseness**
4. **Composition over inheritance**
5. **Exhaustive pattern matching** - no catch-all fallbacks

---

### Rust

**Compiler Strictness:**
```rust
#![deny(unused_must_use)]
#![deny(elided_lifetimes_in_paths)]
#![deny(rust_2018_idioms)]
#![deny(rust_2021_compatibility)]
```

**Formatting (rustfmt):**
```toml
max_width = 120
imports_granularity = "Crate"
group_imports = "StdExternalCrate"
```

**Running rustfmt via Bazel:**
```bash
# Format all Rust files in a repository
bazel run @rules_rust//:rustfmt --@rules_rust//:rustfmt.toml=//:rustfmt.toml

# For repositories with rustfmt.toml in a /rust subdirectory
bazel run @rules_rust//:rustfmt --@rules_rust//:rustfmt.toml=//rust:rustfmt.toml
```

Common rustfmt.toml locations:
- `typedb`: `//:rustfmt.toml`
- `typeql`: `//rust:rustfmt.toml`
- `typedb-driver`: `//rust:rustfmt.toml`
- `typedb-console`: `//:rustfmt.toml`

**Naming:**
| Category | Convention | Examples |
|----------|------------|----------|
| Functions | `snake_case` | `reserve_write_transaction()` |
| Predicates | `is_`/`has_` | `is_clean()`, `has_explicit_end()` |
| Conversions | `as_`/`to_`/`into_` | `as_position()`, `into_iter()` |
| Modules | `snake_case`, `_` suffix for keywords | `type_`, `match_` |
| Generics | Single semantic letters | `D` (DurabilityClient) |

**Error Handling:**
```rust
typedb_error! {
    pub ServerOpenError(component = "Server open", prefix = "SRO") {
        NotADirectory(1, "Invalid path '{path}': not a directory.", path: String),
        DatabaseOpen(6, "Could not open database.", typedb_source: DatabaseOpenError),
    }
}
```

**Type Design:**
- Prefer enums over trait objects for exhaustive matching
- Use newtypes for type safety: `struct VariablePosition { position: u32 }`
- RAII with drop guards: `SnapshotDropGuard<T>`, `DatabaseDropGuard<D>`

**Concurrency:**
- Tokio runtime for async operations
- `Arc<RwLock<T>>` for shared mutable state
- Explicit transaction types: `TransactionRead`, `TransactionWrite`, `TransactionSchema`

**Architecture:**
```
common/* → encoding/storage → concept → ir/compiler → query/executor → database → server
```

---

### Python

**Style:**
- Follow PEP 8 with 120-char line limit
- Use `from __future__ import annotations` for forward compatibility
- Type hints on all function parameters and return types

**Naming:**
| Category | Convention | Examples |
|----------|------------|----------|
| Functions | `snake_case` | `require_non_null()` |
| Classes (public) | `PascalCase` | `TypeDBDriver` |
| Classes (private) | `_PascalCase` | `_ConceptImpl` |
| Properties | `@property` with `snake_case` | `query_type` |

**Imports:**
```python
from __future__ import annotations
from typing import TYPE_CHECKING, Optional, Generic, TypeVar

if TYPE_CHECKING:
    from typedb.api.concept import Concept  # Forward references
```

**Error Handling:**
```python
class TypeDBDriverException(RuntimeError):
    pass

# Error messages as module constants
INVALID_CASTING = DriverErrorMessage("JDR", 4, "Invalid casting from {0} to {1}")
raise TypeDBDriverException(INVALID_CASTING, (from_type, to_type))
```

**Patterns:**
- `NativeWrapper[T]` for wrapping native/JNI objects
- Context managers (`__enter__`/`__exit__`) for resource management
- `is_*()` / `as_*()` method pairs for type checking and casting

**Documentation:**
```python
def method(self, param: str) -> Result:
    """Short description.

    :param param: Parameter description
    :return: Return value description

    Examples::

        result = obj.method("value")
    """
```

---

### Java

**Style:**
- Standard Java conventions with 120-char line limit
- Explicit imports (no wildcards)
- Separate API interfaces from implementation classes

**Naming:**
| Category | Convention | Examples |
|----------|------------|----------|
| Methods | `camelCase` | `getPropertyName()`, `isOpen()` |
| Interfaces | `PascalCase` (no `I` prefix) | `Transaction`, `Concept` |
| Implementations | `*Impl` suffix | `TransactionImpl`, `ConceptImpl` |

**Annotations:**
```java
@Override
@CheckReturnValue  // For fluent APIs
@Nullable          // For nullable parameters/returns
```

**Error Handling:**
```java
public class TypeDBDriverException extends RuntimeException {
    public static final Driver POSITIVE_VALUE_REQUIRED =
        new Driver(8, "Value must be positive, was: %s");
}
throw new TypeDBDriverException(POSITIVE_VALUE_REQUIRED, value);
```

**Patterns:**
- `AutoCloseable` for resource management (try-with-resources)
- `NativeObject<T>` base class for JNI wrappers
- Static factory methods over constructors
- `is*()` / `as*()` method pairs for type checking and casting

**Documentation:**
```java
/**
 * Short description.
 *
 * <h3>Examples</h3>
 * <pre>
 * Result result = obj.method("value");
 * </pre>
 *
 * @param param Parameter description
 * @return Return value description
 */
```

---

### Kotlin

**Style:**
- Kotlin 1.7+ language features
- Explicit imports (no wildcards)
- Prefer immutability (`val` over `var`)

**Naming:**
| Category | Convention | Examples |
|----------|------------|----------|
| Functions | `camelCase` | `loadStorageCost()` |
| Properties | `camelCase` | `principalIdentifier` |
| Constants | `UPPER_SNAKE_CASE` in companion | `PING_PERIOD` |

**Null Safety:**
```kotlin
val email: String? = null           // Nullable type
email?.let { sendTo(it) }           // Safe call with let
val name = email ?: "anonymous"     // Elvis operator
```

**Data Classes & Sealed Types:**
```kotlin
data class StorageType(val vendorID: String, val features: List<Feature>)

sealed class ErrorMessage(val code: String, val description: String) {
    class Server(code: String, desc: String) : ErrorMessage(code, desc)
    class General(code: String, desc: String) : ErrorMessage(code, desc)
}
```

**Error Handling:**
```kotlin
class CloudException(
    val error: ErrorMessage,
    val statusCode: HttpStatusCode = HttpStatusCode.BadRequest
) : Exception(error.toString()) {
    companion object {
        fun invalidAuthToken() = CloudException(General.invalidAuthToken(), HttpStatusCode.Unauthorized)
    }
}
```

**Coroutines:**
```kotlin
suspend fun loadData(): Result { ... }

scope.launchAndHandle(logger) {
    val data = loadData()
    process(data)
}
```

**Extension Functions:**
```kotlin
object Util {
    fun LocalDateTime.toEpochMillisecondUTC() = toInstant(ZoneOffset.UTC).toEpochMilli()
    fun UUID.toProto() = ByteString.copyFromUtf8(this.toString())
}
```

**Patterns:**
- Sealed classes/interfaces for exhaustive `when` expressions
- Companion object factory methods (smart constructors)
- Extension functions in utility objects
- `let`, `also`, `run`, `apply` for scoped operations
