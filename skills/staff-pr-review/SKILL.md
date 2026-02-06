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

# Get HEAD SHA for inline comments
gh pr view {PR_NUMBER} --json headRefOid --jq '.headRefOid'
```

For GitHub Enterprise, prefix with `GH_HOST`:
```bash
GH_HOST=git.example.com gh pr view {PR_NUMBER} ...
```

### 2. Review Checklist

#### Code Quality
- Clean separation of concerns?
- Proper error handling?
- Type safety (if applicable)?
- DRY principle followed?
- Edge cases handled?

#### Architecture
- Sound design decisions?
- Scalability considerations?
- Performance implications?
- Security concerns?

#### Testing
- Tests actually test logic (not just mocks)?
- Edge cases covered?
- Integration tests where needed?
- All tests passing?

#### Production Readiness
- Migration strategy (if schema changes)?
- Backward compatibility considered?
- Documentation complete?
- No obvious bugs?

### 3. Categorize Issues by Severity

#### Critical (Must Fix - Block Merge)
- Race conditions, deadlocks, resource leaks
- Security vulnerabilities (credential exposure, injection, auth bypass)
- Data corruption or loss scenarios
- Breaking API contract changes without documentation
- Silent behavior changes affecting downstream consumers
- Bugs, broken functionality

#### Important (Should Fix)
- Exception handling too broad (`except Exception`) or too narrow
- Missing error logging that hurts debugging
- Performance concerns under load
- Thread safety issues in concurrent code
- Fragile patterns (monkey-patching, global mutable state)
- Incorrect HTTP status codes or error semantics
- Architecture problems, missing features, test gaps

#### Minor (Nice to Have)
- Inefficient but functional code
- Missing type hints or documentation
- Code style inconsistencies
- Optimization opportunities

### 4. Post Inline Comments

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

### 5. Submit Overall Review

```bash
gh pr review {PR_NUMBER} --request-changes --body "## Staff Engineer Review

**Verdict: Request Changes**

---

### Strengths
[What's well done? Be specific with file:line references]

---

### Issues

#### Critical
1. **[Title]** (\`path/file.py:123\`)
   - Issue: [What's wrong]
   - Why: [Why it matters]
   - Fix: [How to fix]

#### Important
...

#### Minor
...

---

### Recommendations
[Improvements for code quality, architecture, or process beyond just fixing issues]

---

### Assessment

**Ready to merge?** [Yes / No / With fixes]

**Reasoning:** [Technical assessment in 1-2 sentences]"
```

## Comment Style Guide

| Do | Don't |
|----|-------|
| **Lead with bold issue** | Bury the lede |
| Explain WHY it matters | Just say "this is wrong" |
| Give concrete fix | Be vague ("improve error handling") |
| Use code blocks | Describe code in prose |
| Be specific (file:line) | Give feedback on code you didn't review |
| Prefix praise with 👍 | Ignore good work |
| Be direct | Use softening language |
| Give clear verdict | Avoid committing to Yes/No |

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
□ Tests: Actually testing logic or just testing mocks?
□ Migrations: Schema changes with rollback strategy?
□ Backward compat: Will existing clients break?
```

## Review Verdicts

| Verdict | Command | When to Use |
|---------|---------|-------------|
| Approve | `--approve` | No critical/important issues, code is solid |
| Request Changes | `--request-changes` | Critical issues OR multiple important issues |
| Comment | `--comment` | Questions only, no blocking issues |

## Output Format

Always include:
1. **Strengths** - What's well done (be specific)
2. **Issues** - Categorized by severity with file:line refs
3. **Recommendations** - Future improvements
4. **Assessment** - Clear Yes/No/With fixes verdict with reasoning
