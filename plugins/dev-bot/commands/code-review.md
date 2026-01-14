---
allowed-tools: Bash(gh pr view:*), Bash(gh pr diff:*), Bash(gh api:*)
description: Code review a pull request
disable-model-invocation: false
---

Provide a code review for the given pull request using inline comments.

Arguments:
- `--force` or `-f`: Skip eligibility checks and always provide a review

## Steps

1. **Eligibility check** (skip if `--force`): Use a Haiku agent to check if the PR is closed or already has your review. Draft PRs are allowed.

2. **Fetch PR details**: Get the PR diff and metadata using `gh pr view` and `gh pr diff`.

3. **Review the changes**: Launch 2 parallel Sonnet agents:
   - **Agent 1 (Logic & Bugs)**: Scan for logic errors, bugs, edge cases, and incorrect assumptions
   - **Agent 2 (Security & Best Practices)**: Check for security issues, race conditions, and anti-patterns

   Each agent returns issues with: file path, line number, category, description, and suggested fix if applicable.

4. **Filter issues**: Keep only high-confidence issues (score 75+). Discard:
   - Pre-existing issues (not introduced by this PR)
   - Linter/type errors (CI will catch these)
   - Nitpicks a senior engineer wouldn't mention
   - Intentional changes related to the PR's purpose

5. **Validate issues**: For each remaining issue, launch a parallel Haiku agent to validate it's not a false positive. The agent should:
   - Re-read the relevant code context
   - Verify the issue is real and introduced by this PR
   - Check if it's actually a bug vs intentional behavior
   - Score confidence 0-100

   Discard any issue scoring below 80. Common false positives:
   - Code that looks wrong but is correct in context
   - Issues already handled elsewhere in the codebase
   - Stylistic concerns disguised as bugs
   - Changes that are intentional based on PR description

6. **Post review with inline comments**: Use the GitHub API to submit a review. You MUST use the exact templates below - do not deviate from the format.

   ```bash
   gh api repos/{owner}/{repo}/pulls/{number}/reviews \
     -f event="COMMENT" \
     -f body="<MAIN_REVIEW_TEMPLATE>" \
     -f 'comments[][path]=file.ts' \
     -f 'comments[][line]=42' \
     -f 'comments[][body]=<INLINE_COMMENT_TEMPLATE>'
   ```

---

## Templates (USE EXACTLY)

### MAIN_REVIEW_TEMPLATE (when issues found)

You MUST use this exact format for the main review body:

```
## Code Review

{one_sentence_summary}

<details>
<summary>Found {count} issue(s)</summary>

| Category | File | Issue |
|----------|------|-------|
| {category} | `{file}:{line}` | {brief_description} |

</details>

See inline comments for details.

---
<sub>[Lleverage Dev Bot](https://github.com/lleverage-ai/dev-bot) · :+1: if helpful</sub>
```

### MAIN_REVIEW_TEMPLATE (no issues)

```
## Code Review

{one_sentence_summary}

No issues found.

---
<sub>[Lleverage Dev Bot](https://github.com/lleverage-ai/dev-bot)</sub>
```

### INLINE_COMMENT_TEMPLATE (with suggestion)

Use this when you can provide a code fix:

```
**{category}**: {description}

```suggestion
{corrected_code}
```

<details>
<summary>Explanation</summary>

{detailed_explanation}

</details>
```

### INLINE_COMMENT_TEMPLATE (without suggestion)

Use this when no direct code fix applies:

```
**{category}**: {description}

<details>
<summary>Explanation</summary>

{detailed_explanation}

</details>
```

---

## Categories

Use these exact category names:

| Category | Use for |
|----------|---------|
| `logic` | Logic errors, incorrect conditions, wrong assumptions |
| `security` | Vulnerabilities, injection risks, auth issues |
| `error-handling` | Missing or incorrect error handling |
| `race-condition` | Concurrency issues, race conditions |
| `perf` | Significant performance issues |

## Notes

- Use templates EXACTLY as shown - consistent formatting is required
- Always include the collapsible Explanation section
- Only include `suggestion` block if you have a concrete fix
- Focus on significant issues only, not style or nitpicks
- Do not run builds, tests, or type checks (CI handles this)
