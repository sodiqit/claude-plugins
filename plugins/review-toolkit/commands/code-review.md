---
description: "Comprehensive PR review using specialized agents"
argument-hint: "[review-aspects] [--range <commit-range>]"
allowed-tools: ["Bash", "Glob", "Grep", "Read", "Task"]
---

# Comprehensive PR Review

Run a comprehensive pull request review using multiple specialized agents, each focusing on a different aspect of code quality.

**Review Aspects (optional):** "$ARGUMENTS"

## Review Workflow:

1. **Determine Review Scope**
   - Parse arguments for:
     - Review aspects (comments, tests, errors, types, code, simplify, all)
     - Commit range via `--range` flag (e.g., `--range HEAD~3..HEAD`, `--range main..feature`)
   - If no `--range` specified: use working tree changes (staged, unstaged, untracked)
   - Default review scope: Run all applicable reviews

2. **Available Review Aspects:**

   - **comments** - Analyze code comment accuracy and maintainability
   - **tests** - Review test coverage quality and completeness
   - **errors** - Check error handling for silent failures
   - **types** - Analyze type design and invariants (if new types added)
   - **code** - General code review for project guidelines
   - **simplify** - Simplify code for clarity and maintainability
   - **all** - Run all applicable reviews (default)

3. **Identify Changed Files**

   **If `--range` is specified** (e.g., `--range main..HEAD`):
   - Run: `git diff --name-only <commit-range>`
   - Example: `git diff --name-only main..HEAD`

   **If no `--range`** (default - working tree changes):
   - **Staged files**: `git diff --cached --name-only`
   - **Unstaged modified files**: `git diff --name-only`
   - **Untracked files**: `git ls-files --others --exclude-standard`

   Combine results and deduplicate. Then identify file types and determine which reviews apply.

4. **Determine Applicable Reviews**

   Based on changes:
   - **Always applicable**: code-reviewer (general quality)
   - **If test files changed**: pr-test-analyzer
   - **If comments/docs added**: comment-analyzer
   - **If error handling changed**: silent-failure-hunter
   - **If types/dtos added/modified**: type-design-analyzer
   - **After passing review**: code-simplifier (polish and refine)

5. **Launch Review Agents**

   **Sequential approach** (one at a time):
   - Easier to understand and act on
   - Each report is complete before next
   - Good for interactive review

   **Parallel approach** (user can request):
   - Launch all agents simultaneously
   - Faster for comprehensive review
   - Results come back together

6. **Aggregate Results**

   After agents complete, summarize:
   - **Critical Issues** (must fix before merge)
   - **Important Issues** (should fix)
   - **Suggestions** (nice to have)
   - **Positive Observations** (what's good)

7. **Provide Action Plan**

   Organize findings:
   ```markdown
   # PR Review Summary

   ## Critical Issues (X found)
   - [agent-name]: Issue description [file:line]

   ## Important Issues (X found)
   - [agent-name]: Issue description [file:line]

   ## Suggestions (X found)
   - [agent-name]: Suggestion [file:line]

   ## Strengths
   - What's well-done in this PR

   ## Recommended Action
   1. Fix critical issues first
   2. Address important issues
   3. Consider suggestions
   4. Re-run review after fixes
   ```

## Usage Examples:

**Full review (default):**
```
/review-toolkit:code-review
```

**Specific aspects:**
```
/review-toolkit:code-review tests errors
# Reviews only test coverage and error handling

/review-toolkit:code-review comments
# Reviews only code comments

/review-toolkit:code-review simplify
# Simplifies code after passing review
```

**Parallel review:**
```
/review-toolkit:code-review all parallel
# Launches all agents in parallel
```

**Review specific commit range:**
```
/review-toolkit:code-review --range HEAD~3..HEAD
# Reviews last 3 commits

/review-toolkit:code-review --range main..HEAD
# Reviews all commits since branching from main

/review-toolkit:code-review tests errors --range origin/main..HEAD
# Reviews only tests and errors for commits not yet pushed
```

## Agent Descriptions:

**comment-analyzer**:
- Verifies comment accuracy vs code
- Identifies comment rot
- Checks documentation completeness

**pr-test-analyzer**:
- Reviews behavioral test coverage
- Identifies critical gaps
- Evaluates test quality

**silent-failure-hunter**:
- Finds silent failures
- Reviews catch blocks
- Checks error logging

**type-design-analyzer**:
- Analyzes type encapsulation
- Reviews invariant expression
- Rates type design quality

**code-reviewer**:
- Checks CLAUDE.md compliance
- Detects bugs and issues
- Reviews general code quality

**code-simplifier**:
- Simplifies complex code
- Improves clarity and readability
- Applies project standards
- Preserves functionality

## Tips:

- **Run early**: Before creating PR, not after
- **Focus on changes**: Agents analyze git diff by default
- **Address critical first**: Fix high-priority issues before lower priority
- **Re-run after fixes**: Verify issues are resolved
- **Use specific reviews**: Target specific aspects when you know the concern

## Notes:

- Agents run autonomously and return detailed reports
- Each agent focuses on its specialty for deep analysis
- Results are actionable with specific file:line references
- Agents use appropriate models for their complexity
- All agents available in `/agents` list
