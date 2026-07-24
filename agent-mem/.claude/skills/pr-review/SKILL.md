---
name: pr-review
description: Review a GitHub pull request for design compliance, coding quality, correctness, thread safety, and test coverage
allowed-tools: Bash, Read, Glob, Grep, Agent, WebFetch
argument-hint: "<owner/repo#number> [extra instruction]"
---

# PR Review

Review a GitHub pull request thoroughly and produce a structured report.

## Arguments

`$ARGUMENTS` should contain a PR identifier followed by an optional extra instruction:

```
<PR_ID> [extra instruction text]
```

**PR identifier** (required) — one of:
- `owner/repo#123` — fully qualified
- `#123` or `123` — uses the current git remote's repo

**Extra instruction** (optional) — any text after the PR identifier is treated as an
additional review instruction. Apply it throughout the review as a supplementary focus area.
For example: `/pr-review #42 pay special attention to backward compatibility` or
`/pr-review org/repo#100 check that all new endpoints have rate limiting`.

Parse the identifier, resolve the repo and PR number, and note the extra instruction if provided.

## Steps

### 1. Fetch PR metadata and diff

```bash
gh pr view <NUMBER> --repo <OWNER/REPO> --json title,body,state,author,files,additions,deletions,baseRefName,headRefName
gh pr diff <NUMBER> --repo <OWNER/REPO>
```

Read the full diff carefully. If it's large, read it in chunks.

### 2. Identify relevant design docs

For each directory touched by the PR, search for design documents:

```
Glob pattern: <changed_directory>/**/DESIGN.md
Glob pattern: <changed_directory>/**/design*.md
Glob pattern: <changed_directory>/**/ARCHITECTURE.md
```

Also check the project root for `DESIGN.md`, `ARCHITECTURE.md`, `AGENTS.md`, and `CLAUDE.md`.
Read any found design docs. These define the contracts the PR must comply with.

Also check the project's `CLAUDE.md` for any project-specific review criteria — if it has a
"PR Review Instructions" or "Code Review Checklist" section, follow those criteria in addition
to the ones below.

### 3. Review the PR

Evaluate the PR against these criteria:

#### 3a. Design Doc Compliance (if design docs exist)
- Check that the implementation conforms to documented contracts, invariants, and assumptions.
- For each requirement in the design doc that is relevant to this PR, note whether the PR
  complies or violates it.
- Present as a table: requirement | pass/fail/partial | notes.
- If no design docs are found, skip this section and note that.

#### 3b. Coding Quality
- **Docstrings**: Every new or modified public method/function must have a clear docstring.
  Flag any that are missing.
- **Function clarity**: Check that function signatures, parameter names, and return types
  are self-explanatory and have type hints.
- **Consistency**: Config classes, naming conventions, file placement, and patterns should
  match the existing codebase. Flag inconsistencies.

#### 3c. Correctness
- Identify logic bugs, especially in error/failure handling paths.
- Check resource management: locks released, pool entries freed, file descriptors closed,
  memory freed — especially in exception paths.
- Verify that error/failure paths don't leave the system in an inconsistent state
  (e.g., registering objects on store failure, leaking allocations on exception).
- Check edge cases: empty inputs, None values, boundary conditions.

#### 3d. Thread Safety
- Check shared state access patterns and lock granularity.
- Verify that concurrent use from multiple threads is safe where documented.
- Flag busy-loops without sleep/yield, and potential deadlocks.

#### 3e. Test Coverage
- Note what the tests cover and what's missing.
- In particular, check for: failure/error path tests, concurrent access tests,
  edge case tests (empty inputs, boundary values).

#### 3f. Extra Instruction (if provided)
If the user supplied an extra instruction after the PR identifier, apply it as an additional
review pass. Dedicate a section in the output to findings specific to that instruction.
Title the section with the gist of the instruction (e.g., "### Backward Compatibility" or
"### Rate Limiting Check"). If there are no findings, note "No issues found."

### 4. Format the output

Structure the review as follows:

```
## PR #<NUMBER> Review: <TITLE>

**Author:** <author> | **+<additions>/-<deletions>** | <file_count> files | Target: <base_branch>

### Summary
<2-3 sentence description of what the PR does>

### Design Doc Compliance
<Table if design docs found, or "No design docs found in changed directories.">

### Coding Quality Issues
<Numbered list of issues>

### Correctness Issues
<Numbered list of issues, with file:line references>

### Thread Safety
<Issues or "No concerns.">

### Test Coverage
<What's covered, what's missing>

### <Extra Instruction Topic> (only if extra instruction was provided)
<Findings specific to the user's extra instruction>

### Summary of Findings
| Severity | Count | Key Items |
|----------|-------|-----------|
| Bug      | N     | ...       |
| Missing docstring | N | ... |
| Design concern | N | ...   |
| Performance | N  | ...      |
| Minor    | N     | ...       |
```

## Important Notes

- Always read the full diff, not just file names.
- Reference specific files and line numbers (from the diff) when citing issues.
- Be concrete: show the problematic code snippet when flagging a bug.
- Distinguish between bugs (incorrect behavior) and style/quality issues.
- If the PR description lists TODOs or known limitations, don't flag those as bugs —
  note them as acknowledged.
- Do NOT leave PR comments or approve/reject — only produce the review report for the user.
