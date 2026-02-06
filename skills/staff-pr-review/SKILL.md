---
name: staff-pr-review
description: Use when reviewing pull requests with the rigor of a senior staff engineer - catches race conditions, API contract changes, security issues, and architectural problems others miss
---

# Staff Engineer PR Review

Perform rigorous code reviews with the critical eye of a senior staff engineer.

## Core Principles

- **Technical accuracy over politeness** - Be direct, not mean
- **Catch what others miss** - Race conditions, contract changes, security holes
- **Explain WHY it matters** - Connect issues to real-world failures
- **Acknowledge good work** - Give credit where due

## Review Process

### 1. Fetch PR Data

```bash
# Get PR metadata
gh pr view {PR_NUMBER} --json title,body,additions,deletions,files,commits,author

# Get the diff
gh pr diff {PR_NUMBER}
```

For GitHub Enterprise, prefix with `GH_HOST`:
```bash
GH_HOST=git.example.com gh pr view {PR_NUMBER} ...
```

### 2. Analyze Systematically

#### Critical Issues (Block Merge)
- Race conditions, deadlocks, resource leaks
- Security vulnerabilities (credential exposure, injection, auth bypass)
- Data corruption or loss scenarios
- Breaking API contract changes without documentation
- Silent behavior changes affecting downstream consumers

#### Medium Issues (Should Fix)
- Exception handling too broad (`except Exception`) or too narrow
- Missing error logging that hurts debugging
- Performance concerns under load
- Thread safety issues in concurrent code
- Fragile patterns (monkey-patching, global mutable state)
- Incorrect HTTP status codes or error semantics

#### Minor Issues (Nice to Fix)
- Inefficient but functional code
- Missing type hints or documentation
- Code style inconsistencies

#### Positive Callouts
- Good refactors, DRY improvements
- Proper async patterns
- Security hardening

### 3. Post Inline Comments

Use the GitHub API to post comments on specific lines:

```bash
gh api /repos/{OWNER}/{REPO}/pulls/{PR_NUMBER}/comments \
  -f body="**Issue title.** Explanation of the problem.

Suggested fix or alternative." \
  -f commit_id="{HEAD_SHA}" \
  -f path="{FILE_PATH}" \
  -F line={LINE_NUMBER} \
  -f side="RIGHT"
```

Get the HEAD SHA first:
```bash
gh pr view {PR_NUMBER} --json headRefOid --jq '.headRefOid'
```

### 4. Submit Overall Review

```bash
gh pr review {PR_NUMBER} --request-changes --body "## Staff Engineer Review

**Verdict: Request Changes**

Overall direction is good—[brief positive]. But there are issues to address.

---

### Critical Issues

#### 1. [Title] (\`path/to/file.py:123\`)
[Explanation of problem and suggested fix]

---

### Medium Issues

#### 2. [Title] (\`path/to/file.py:456\`)
[Explanation]

---

### Minor Issues

#### 3. [Title]
[Brief note]

---

### Good Stuff

- **[Feature]** (\`file.py\`) — [Why it's good]

---

Fix critical issues, address mediums, and this is good to go."
```

## Comment Style Guide

| Do | Don't |
|----|-------|
| **Lead with bold issue** | Bury the lede |
| Explain WHY it matters | Just say "this is wrong" |
| Give concrete fix | Be vague |
| Use code blocks | Describe code in prose |
| Prefix praise with emoji | Ignore good work |
| Be direct | Use softening language |

### Examples

**Good:**
```
**Race condition.** You check the cache outside the lock, then acquire it.
If another task evicts the entry between `.get()` and `.move_to_end()`,
you get a `KeyError`. This only manifests under load.

Either use the lock for all access, or don't call `move_to_end` in the fast-path.
```

**Bad:**
```
Maybe you should consider using a lock here? I think there might be an issue.
```

## Red Flags Checklist

Run through this for every PR:

```
□ Exception handling: Catching Exception/BaseException too broadly?
□ Concurrency: Shared mutable state without locks? Race conditions?
□ Resource management: Connections/files/tasks properly closed/cancelled?
□ Error visibility: Will failures be logged? Debuggable at 3am?
□ API contracts: Status codes changed? Response shape changed? Breaking clients?
□ Security: User input sanitized? Secrets exposed? Auth checked?
□ Performance: O(n²) loops? Blocking calls in async code? Memory leaks?
□ Retry logic: Exponential backoff? Jitter? Circuit breakers actually shared?
□ Thread safety: random module? Global state? Non-atomic operations?
□ Silent changes: Defaults changed? Parameters dropped? Behavior altered?
```

## Review Verdicts

| Verdict | When to Use |
|---------|-------------|
| `--approve` | No critical/medium issues, code is solid |
| `--request-changes` | Critical issues OR multiple medium issues |
| `--comment` | Questions only, no blocking issues |
