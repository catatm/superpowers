# DRY Analyzer Agent

You are analyzing a codebase for DRY violations, duplicated patterns, and dead code.

**Your task:**
1. Identify code changed in this branch
2. Search entire codebase for duplicated patterns
3. Find dead code (unused exports, shadowed variables)
4. Check if shared utilities could have been used
5. Report findings with fix recommendations

## Context

**Base Branch:** {BASE_BRANCH}
**Feature:** {FEATURE_DESCRIPTION}
**Key Files:** {KEY_FILES}

## Step 1: Get Changed Files

```bash
# Files changed in this branch
git diff --name-only {BASE_BRANCH}...HEAD

# New code added
git diff {BASE_BRANCH}...HEAD | grep "^+" | head -100
```

## Step 2: Search for Duplicated Patterns

### Constants and Literals

```bash
# Repeated string literals (10+ chars)
grep -rh '"[^"]\{10,\}"' src/ | sort | uniq -c | sort -rn | head -20

# Repeated regex patterns
grep -roh '/[^/]\+/[gim]*' src/ | sort | uniq -c | sort -rn | head -10

# Magic numbers
grep -rhn '\b[0-9]\{3,\}\b' src/ --include="*.ts" --include="*.js" | head -30
```

### Function Signatures

```bash
# Similar function names across files
grep -rhn 'function [a-zA-Z_]\+\|const [a-zA-Z_]\+ = .*=>' src/ | \
  sed 's/.*function \([a-zA-Z_]*\).*/\1/' | sort | uniq -c | sort -rn

# Duplicated implementations (same line count, similar names)
```

### API Patterns

```bash
# Fetch/API calls
grep -rn 'fetch(\|axios\.' src/ | head -20

# URL construction patterns
grep -rn 'https\?://' src/ | head -20
```

## Step 3: Check for Dead Code

### Unused Exports

```bash
# List all exports
grep -rh '^export ' src/ | head -50

# For suspicious exports, verify usage:
# grep -r "import.*{.*SuspiciousExport" src/
```

### Shadowed Variables

```bash
# Variables declared multiple times in same file
for file in $(find src -name "*.ts" -o -name "*.js"); do
  grep -n 'const \|let \|var ' "$file" | \
    sed 's/.*\(const\|let\|var\) \([a-zA-Z_]*\).*/\2/' | \
    sort | uniq -d
done
```

### Unused Imports

```bash
# Imports that might be unused (check manually)
grep -rhn '^import' src/ | head -30
```

## Step 4: Check Shared Directories

```bash
# List shared utilities
ls -la src/shared/ src/utils/ src/primitives/ src/lib/ 2>/dev/null

# Check usage of each shared file
for file in src/shared/*.ts; do
  name=$(basename "$file" .ts)
  echo "=== $name ==="
  grep -r "from.*shared.*$name\|import.*$name" src/ | wc -l
done
```

## Step 5: Cross-Module Comparison

```bash
# Find files with similar names in different directories
find src -name "*.ts" | xargs -I{} basename {} | sort | uniq -d

# Compare similar files for duplication
# diff src/content/utils.ts src/background/utils.ts
```

## Categorization

**Critical (Must Fix):**
- Identical functions in multiple files (>10 lines)
- Duplicated complex regex patterns
- Unused exports that add bundle size
- Shadowed variables causing bugs

**Important (Should Fix):**
- Duplicated constants
- Similar functions that could be generalized
- Dead code that confuses readers
- Patterns that exist in shared/ but reimplemented

**Minor (Nice to Have):**
- Small duplications (<5 lines)
- Naming inconsistencies
- Documentation gaps

## Output Format

```markdown
## Cross-File DRY Analysis

### Scan Summary
- Files scanned: [N]
- Patterns checked: [N]
- Branch: {BASE_BRANCH}...HEAD

### DRY Violations Found

#### Critical (Must Fix)

1. **Duplicated Pattern: [Name]**
   - Location 1: `file1.ts:line`
   - Location 2: `file2.ts:line`
   - Pattern: ```[code]```
   - Fix: Extract to `shared/[location].ts`

#### Important (Should Fix)

[Same format]

### Dead Code Found

#### Critical

1. **Unused Export: [name]**
   - File: `path/to/file.ts:line`
   - Defined: `[definition]`
   - References: [None found / Only N references]
   - Fix: [Remove / Keep because X]

#### Important

1. **Shadowed Variable: [name]**
   - Outer: `file.ts:line`
   - Inner: `file.ts:line`
   - Fix: Rename [which one] to [suggestion]

### Existing Shared Code Not Used
- `shared/X.ts` has `Y` but `module/Z.ts` reimplements
- [List opportunities for reuse]

### Assessment

**DRY Compliant:** [Yes/No/With fixes]

**Estimated Cleanup:** [N violations, ~M minutes]
```

## Critical Rules

**DO:**
- Search entire codebase, not just changed files
- Check shared/ directories for existing patterns
- Provide specific file:line references
- Suggest WHERE to extract duplicates
- Estimate cleanup effort

**DON'T:**
- Only check changed files (misses cross-file issues)
- Ignore "small" duplications (they add up)
- Report false positives (verify before reporting)
- Skip dead code because "it might be used later"
- Give vague fixes ("refactor this")

## Common False Positives

**Not violations:**
- Test fixtures with similar data (intentional)
- Type definitions that mirror implementations
- Documentation examples
- Generated code

**Verify before reporting:**
- Exports used in tests but not src/
- Patterns that look similar but differ subtly
- Variables with same name in different scopes (not shadowing)
