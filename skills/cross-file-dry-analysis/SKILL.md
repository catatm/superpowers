---
name: cross-file-dry-analysis
description: Use before merging to detect DRY violations, duplicated patterns, and dead code across the entire codebase
---

# Cross-File DRY Analysis

## Overview

Analyze the entire codebase to detect DRY violations, duplicated patterns, and dead code that per-task reviews miss. Unlike task-scoped review, this examines cross-file relationships.

**Core principle:** Duplication is the root of maintenance nightmares.

**Announce at start:** "I'm using the cross-file-dry-analysis skill to check for duplication and dead code across the codebase."

## When to Use

**Mandatory:**
- Before merging any feature branch
- After implementing similar functionality in multiple places
- When feature touches multiple modules

**Valuable:**
- Periodic codebase health checks
- Before major refactoring
- When onboarding to identify tech debt

## The Process

### Step 1: Identify New/Modified Code

```bash
# Get files changed in this branch
git diff --name-only origin/main...HEAD

# Get new patterns introduced
git diff origin/main...HEAD | grep "^+" | grep -v "^+++"
```

### Step 2: Search for Duplicated Patterns

**Check for duplicated constants:**
```bash
# Find repeated string literals
grep -rh '"[^"]\{10,\}"' src/ | sort | uniq -c | sort -rn | head -20

# Find repeated numbers (magic numbers)
grep -rhn '\b[0-9]\{4,\}\b' src/ | sort -t: -k3 | uniq -d
```

**Check for duplicated regex patterns:**
```bash
# Find regex patterns
grep -rh 'new RegExp\|/.*/' src/ | sort | uniq -c | sort -rn
```

**Check for duplicated functions:**
```bash
# Find similar function signatures
grep -rhn 'function\|const.*=.*=>' src/ | sort -t: -k3
```

### Step 3: Check for Dead Code

**Unused exports:**
```bash
# List all exports
grep -rh '^export' src/ | sed 's/export //' | cut -d' ' -f1-2

# For each export, verify it's imported somewhere
# grep -r "import.*{.*ExportName.*}" src/
```

**Unused functions:**
```bash
# Find function definitions
grep -rhn 'function [a-zA-Z_]' src/

# Check if each is called (excluding definition)
```

**Shadowed variables:**
```bash
# Look for variable redeclarations
grep -rhn 'const\|let\|var' src/ | sort -t: -k3 | uniq -d
```

### Step 4: Check Shared Code Directories

**Verify shared code is used:**
```bash
# List shared utilities
ls src/shared/ src/utils/ src/primitives/ 2>/dev/null

# For each file, check it's imported
# grep -r "from.*shared" src/
```

**Check for patterns that should be shared:**
```bash
# Find similar implementations in different modules
diff -rq src/content/ src/background/ 2>/dev/null | grep "differ"
```

### Step 5: Report Findings

**Output Format:**

```markdown
## Cross-File DRY Analysis

### Scan Summary
- Files scanned: N
- Patterns checked: N
- Time: Ns

### DRY Violations Found

#### Critical (Must Fix)

1. **Duplicated Pattern: [Pattern Name]**
   - Location 1: `path/to/file1.ts:line`
   - Location 2: `path/to/file2.ts:line`
   - Pattern: `[the duplicated code/pattern]`
   - Fix: Extract to `shared/[suggested-location].ts`

#### Important (Should Fix)

...

### Dead Code Found

#### Critical

1. **Unused Export: [name]**
   - File: `path/to/file.ts:line`
   - Defined: `export function unusedFn()`
   - References: None found
   - Fix: Remove or document why it exists

#### Important

1. **Shadowed Variable: [name]**
   - Outer: `path/to/file.ts:10`
   - Inner: `path/to/file.ts:25`
   - Fix: Rename inner variable

### Existing Patterns to Reuse
[List of shared utilities that could have been used]

### Assessment

**DRY Compliant:** [Yes/No/With fixes]

**Estimated Cleanup:** [X violations, ~Y minutes to fix]
```

## Subagent Dispatch

When dispatching as a subagent, use template at `dry-analyzer-prompt.md`:

```
Task tool (superpowers:cross-file-dry-analysis):
  Use template at cross-file-dry-analysis/dry-analyzer-prompt.md

  BASE_BRANCH: [origin/main or other base]
  FEATURE_DESCRIPTION: [what was implemented]
  KEY_FILES: [main files to focus on]
```

## Integration

**Called by:**
- **finishing-a-development-branch** (Step 1b) - Before standards check
- **subagent-driven-development** - After all tasks complete

**Pairs with:**
- **standards-aware-review** - Run DRY first, standards second
- **static-analysis** - Complementary dead code detection

## Pattern Library

**Common duplications to check:**

| Pattern | Command | Fix |
|---------|---------|-----|
| URL regex | `grep -r 'https?://' src/` | Extract to `shared/patterns.ts` |
| Date formatting | `grep -r 'toISOString\|toLocale' src/` | Use shared formatter |
| Error messages | `grep -rh '"Error:' src/` | Centralize in `shared/errors.ts` |
| API endpoints | `grep -r 'fetch\|axios' src/` | Create API client |
| Config values | `grep -r 'const.*=' src/ \| grep -i config` | Move to config file |

## Red Flags

**Never:**
- Skip analysis because "it's a small change"
- Ignore duplication "for now"
- Leave dead code "in case we need it"
- Copy-paste without checking for existing utils

**Always:**
- Search before implementing
- Extract after finding duplication
- Remove dead code immediately
- Document intentional duplication (rare)

## Example Output

```markdown
## Cross-File DRY Analysis

### Scan Summary
- Files scanned: 45
- Patterns checked: 12
- Time: 3.2s

### DRY Violations Found

#### Critical

1. **Duplicated Pattern: URL Regex**
   - Location 1: `src/content/scanner.ts:45`
   - Location 2: `src/background/urlHandler.ts:23`
   - Pattern: `/https?:\/\/[^\s]+/g`
   - Fix: Extract to `shared/patterns.ts` as `URL_PATTERN`

2. **Duplicated Function: formatTimestamp**
   - Location 1: `src/content/ui.ts:89-95`
   - Location 2: `src/popup/display.ts:34-40`
   - Code: Identical timestamp formatting (6 lines each)
   - Fix: Move to `shared/formatters.ts`

#### Important

1. **Duplicated Constant: API_TIMEOUT**
   - `src/api/client.ts:5` → `const TIMEOUT = 5000`
   - `src/api/retry.ts:8` → `const TIMEOUT = 5000`
   - Fix: Export from single location

### Dead Code Found

#### Critical

1. **Unused Export: `legacyParser`**
   - File: `src/shared/parsers.ts:145`
   - Defined: `export function legacyParser()`
   - References: None found in codebase
   - Fix: Remove function

#### Important

1. **Shadowed Variable: `result`**
   - Outer: `src/handlers/process.ts:23`
   - Inner: `src/handlers/process.ts:45`
   - Fix: Rename inner to `processedResult`

### Existing Patterns Not Used
- `shared/patterns.ts` has `LINK_REGEX` but `content/scanner.ts` defines its own
- `shared/formatters.ts` has `formatDate` but `popup/display.ts` reimplements

### Assessment

**DRY Compliant:** No

**Estimated Cleanup:** 5 violations, ~15 minutes to fix
```
