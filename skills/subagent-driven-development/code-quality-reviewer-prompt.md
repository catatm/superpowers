# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment

---

## Enhanced Review Checklist

The code reviewer MUST check these additional items beyond the base template:

### 1. Project Standards Check

**Before reviewing code, read project standards:**

```bash
# Find standards documentation
find . -name "*standard*.md" -o -name "CLAUDE.md" 2>/dev/null
ls docs/*.md 2>/dev/null
```

**Check compliance with:**
- `docs/architecture-standard.md` - Layer boundaries, patterns
- `docs/coding-standard.md` - Size limits, naming, conventions
- `docs/security-standard.md` - Data minimization, broadcast security

**If standards exist, add to review output:**
```markdown
### Standards Compliance
- [ ] Architecture standard followed
- [ ] Coding standard followed
- [ ] Security standard followed
```

### 2. Dead Code Detection

**Check for dead code in changed files:**

```bash
# Unused variables (ESLint will catch, but verify)
grep -n 'const\|let\|var' [changed-files] | while read line; do
  var=$(echo "$line" | sed 's/.*\(const\|let\|var\) \([a-zA-Z_]*\).*/\2/')
  # Check if var is used after definition
done
```

**Look for:**
- Unused imports at top of file
- Variables declared but never read
- Functions defined but never called
- Commented-out code blocks
- `console.log` statements left in

**Add to Issues section:**
```markdown
#### Dead Code

1. **Unused variable: `name`**
   - File: `path:line`
   - Fix: Remove or use
```

### 3. DRY Violations

**Search codebase for existing patterns before marking as compliant:**

```bash
# Check if similar patterns exist elsewhere
grep -rn "[key-pattern-from-new-code]" src/
```

**Look for:**
- Constants that should be shared
- Functions that duplicate existing utils
- Regex patterns defined in multiple places
- Similar error handling that could be abstracted

**Add to Issues section:**
```markdown
#### DRY Violations

1. **Duplicated pattern: [description]**
   - New code: `path:line`
   - Existing: `shared/path:line`
   - Fix: Reuse existing or extract to shared
```

### 4. Size Violations

**Check against documented limits (or sensible defaults):**

| Metric | Default Limit | Check Command |
|--------|---------------|---------------|
| Function LOC | 50 lines | Count lines in each function |
| File LOC | 300 lines | `wc -l [file]` |
| Cyclomatic complexity | 10 | Count branches |

**Add to Issues section if violated:**
```markdown
#### Size Violations

1. **Function too long: `functionName`**
   - File: `path:line`
   - Size: 75 LOC (limit: 50)
   - Fix: Extract [specific logic] to separate function
```

### 5. Shadowed Variables

**Check for variable shadowing:**

```bash
# Find variables with same name in nested scopes
grep -n 'const\|let' [file] | sort by variable name
```

**Common patterns:**
- Parameter shadows outer variable
- Loop variable shadows function variable
- Catch block variable shadows outer scope

**Add to Issues section:**
```markdown
#### Shadowed Variables

1. **Shadowed: `result`**
   - Outer scope: `path:23`
   - Inner scope: `path:45`
   - Fix: Rename inner to `processedResult`
```

---

## Updated Output Format

The code reviewer should return:

```markdown
### Strengths
[What's well done]

### Issues

#### Critical (Must Fix)
[Bugs, security, data loss]

#### Important (Should Fix)
- [Architecture problems]
- [Dead code]
- [DRY violations]
- [Size violations]
- [Shadowed variables]
- [Standards violations]

#### Minor (Nice to Have)
[Style, optimization]

### Standards Compliance (if standards exist)
- [x] Architecture: [compliant/violation at X]
- [x] Coding: [compliant/violation at X]
- [x] Security: [compliant/violation at X]

### Assessment

**Ready to merge?** [Yes/No/With fixes]

**Reasoning:** [Technical assessment]
```
