Generate local Bazel repository overrides for cross-repo development.

## Instructions

**Target repository:** $ARGUMENTS (e.g., `typedb`, `typedb-driver`, `typedb-console`)

If no argument provided, ask which repository needs local dependency overrides.

1. **Identify the target repository's dependencies** by reading its `dependencies/typedb/repositories.bzl` file

2. **For each dependency that exists locally** in `repositories/`, generate the override

3. **Generate two outputs:**

### Option A: Modify repositories.bzl (persistent)

Show the user the changes needed to convert `git_repository` to `native.local_repository`:

**Before:**
```python
def typeql():
    git_repository(
        name = "typeql",
        remote = "https://github.com/typedb/typeql",
        tag = "X.Y.Z",
    )
```

**After:**
```python
def typeql():
    native.local_repository(
        name = "typeql",
        path = "../typeql"
    )
```

### Option B: Command-line flags (temporary)

Generate a bazel command with `--override_repository` flags:

```bash
bazel build //target \
  --override_repository=typeql=/absolute/path/to/repositories/typeql \
  --override_repository=typedb_protocol=/absolute/path/to/repositories/typedb-protocol
```

## Repository Name Mapping

Use this mapping between directory names and Bazel repository names:

| Directory | Bazel Name | Relative Path (from consumer) |
|-----------|------------|-------------------------------|
| `typeql` | `typeql` | `../typeql` |
| `typedb-protocol` | `typedb_protocol` | `../typedb-protocol` |
| `typedb-behaviour` | `typedb_behaviour` | `../typedb-behaviour` |
| `typedb-driver` | `typedb_driver` | `../typedb-driver` |
| `typedb` | `typedb` | `../typedb` |
| `dependencies` | `vaticle_dependencies` or `typedb_dependencies` | `../dependencies` |
| `bazel-distribution` | `vaticle_bazel_distribution` or `typedb_bazel_distribution` | `../bazel-distribution` |

Note: Private repos (`typedb-cluster`, `typedb-cloud`) use `typedb_dependencies` naming; public repos use `vaticle_dependencies`.

## Dependency Graph Reference

```
typedb          depends on: typeql, typedb_protocol, typedb_behaviour, vaticle_dependencies
typedb-driver   depends on: typedb_protocol, typedb_behaviour, vaticle_dependencies
typedb-console  depends on: typedb_driver, typeql, vaticle_dependencies
typeql          depends on: typedb_behaviour, vaticle_dependencies
typedb-cluster  depends on: typedb, typedb_dependencies, typedb_bazel_distribution
typedb-cloud    depends on: typedb-driver (Maven), typeql (Maven), vaticle_dependencies
```

## Output Format

```
## Local Dependency Overrides for [repo]

### Dependencies detected:
- typeql (currently: tag X.Y.Z)
- typedb_protocol (currently: tag X.Y.Z)

### Option A: Modify repositories.bzl

Edit `repositories/[repo]/dependencies/typedb/repositories.bzl`:

[Show exact edits needed]

### Option B: Command-line overrides

```bash
cd repositories/[repo]
bazel build //your:target \
  --override_repository=typeql=$(pwd)/../typeql \
  --override_repository=typedb_protocol=$(pwd)/../typedb-protocol
```

### Reverting

To revert Option A changes:
```bash
cd repositories/[repo]
git checkout dependencies/typedb/repositories.bzl
```
```
