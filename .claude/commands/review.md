Review the current changes for adherence to TypeDB development principles.

## Review Scope

Review the changes in: $ARGUMENTS (if empty, review all uncommitted changes in the current repository)

## Review Criteria

### 1. Meta-Principles (from CLAUDE.md)

**Architectural Correctness:**
- [ ] Does the change follow existing architectural patterns in the repository?
- [ ] Is it cohesive with surrounding code?
- [ ] Does it maintain the established module boundaries and dependencies?

**Simplicity:**
- [ ] Is this the simplest solution that solves the problem?
- [ ] Could any part be simplified further without losing functionality?
- [ ] Are there unnecessary abstractions, indirections, or over-engineering?
- [ ] Is there premature generalization for hypothetical future requirements?

**Readability:**
- [ ] Would a reviewer unfamiliar with the change understand it easily?
- [ ] Does naming match conventions used elsewhere in the codebase?
- [ ] Is the code self-documenting, or does it need clarifying comments?

**Correctness:**
- [ ] Are error cases handled consistently with other code in the repository?
- [ ] Are there missing edge cases? Consider:
  - Empty/null inputs
  - Boundary conditions
  - Concurrent access (if applicable)
  - Resource cleanup / error recovery
  - Invalid state transitions
- [ ] Does exhaustive pattern matching cover all cases?

### 2. Code Quality

**Error Handling:**
- [ ] Are errors propagated correctly using the codebase's error patterns?
- [ ] Do error messages provide sufficient context for debugging?
- [ ] Is error chaining preserved to maintain root cause?

**Type Safety:**
- [ ] Are types used to prevent invalid states?
- [ ] Could stronger types eliminate runtime checks?

**Resource Management:**
- [ ] Are resources (files, connections, locks) properly acquired and released?
- [ ] Is cleanup guaranteed even in error paths?

### 3. Cross-Repository Impact

- [ ] Could this change affect other repositories? (See dependency graph)
- [ ] Does `typedb-behaviour` need new or updated test scenarios?
- [ ] Does `typedb-docs` need documentation updates?
- [ ] Are there breaking changes to APIs or protocols?

### 4. Style Conformance

**Rust repositories:**
- [ ] Follows rustfmt configuration (120 char width, crate-level imports)
- [ ] Uses `typedb_error!` macro for error types
- [ ] Proper use of `#![deny(...)]` attributes

**All repositories:**
- [ ] Correct license header present
- [ ] No wildcard imports
- [ ] Imports grouped: std → external → internal

## Output Format

Provide your review as:

```
## Summary
[One paragraph overall assessment]

## Issues Found

### [Critical/Major/Minor]: [Issue Title]
**Location:** [file:line or general area]
**Problem:** [Description of the issue]
**Suggestion:** [How to fix it]

## Edge Cases to Consider
- [List any edge cases that may not be handled]

## Questions for the Author
- [Any clarifications needed]

## Positive Observations
- [What's done well - architectural decisions, clean patterns, etc.]
```

## Instructions

1. First, identify what files have changed using `git diff` or `git status`
2. Read the changed files and understand the context
3. For each repository touched, read its README.md and any architecture.md to understand conventions
4. Apply the review criteria above systematically
5. Be specific - reference exact file paths and line numbers
6. Prioritize: focus on correctness and architectural issues over style nits
7. If changes span multiple repos, check for consistency across them
