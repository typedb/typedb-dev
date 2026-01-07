Analyze the current changes and identify cross-repository impacts.

## Instructions

1. **Identify changed files** in the current repository (or the repository specified: $ARGUMENTS)
   - Use `git status` and `git diff` to find modified files
   - Categorize changes: schema/grammar, protocol, API, behaviour tests, documentation

2. **Analyze impact based on the dependency graph:**

   ```
   dependencies ─┬─► typeql ─────────────────┬─► typedb ──────► typedb-cluster
                 │                           │
                 ├─► typedb-protocol ────────┼─► typedb-driver ─┬─► typedb-console
                 │                           │                  │
                 └─► typedb-behaviour ───────┘                  └─► typedb-cloud
                                                                      │
                                                                      └─► typedb-cloud-infrastructure
   ```

3. **For each type of change, check downstream impacts:**

   | Change Type | Check These Repos |
   |-------------|-------------------|
   | `typeql` grammar/AST | `typedb` (compiler), `typedb-driver` (error messages), `typedb-cloud` (Maven dep) |
   | `typedb-protocol` .proto files | `typedb`, `typedb-driver`, `typedb-console`, `typedb-studio` |
   | `typedb` server behaviour | `typedb-behaviour` (new scenarios?), `typedb-docs`, `typedb-cluster` |
   | `typedb-driver` API changes | `typedb-console`, `typedb-studio`, `typedb-cloud`, `typedb-docs` |
   | `typedb-behaviour` scenarios | `typedb`, `typedb-driver`, `typeql` (test runners) |
   | `typedb-cluster` changes | Usually isolated; check if cluster-specific docs needed |
   | `typedb-cloud` changes | Check `typedb-cloud-infrastructure` for infra impacts |

4. **Specifically check:**
   - [ ] Does `typedb-behaviour` need new or updated test scenarios?
   - [ ] Does `typedb-docs` need documentation updates? (Check even if behavior hasn't changed - docs may be incomplete)
   - [ ] Are there breaking changes to user-facing APIs or commands?
   - [ ] Do error messages or error codes change?

5. **Report findings:**
   - List each impacted repository with specific files/areas that may need changes
   - Indicate priority: "Must change" vs "Should review" vs "Consider updating"
   - Flag any breaking changes that would require coordinated releases

## Output Format

```
## Cross-Repository Impact Analysis

### Repository: [current repo]
**Changes detected:**
- [file1]: [description of change]
- [file2]: [description of change]

### Impacted Repositories

#### [repo-name] - [Must change | Should review | Consider updating]
**Reason:** [why this repo is impacted]
**Specific areas:**
- [file or module that needs attention]
- [what needs to change or be verified]

### Recommended PR Order
1. [first repo to merge]
2. [second repo]
...

### Breaking Changes
- [ ] None detected
- [ ] [Description of breaking change and migration path]
```
