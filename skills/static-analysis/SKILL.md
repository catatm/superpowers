---
name: static-analysis
description: Use to run static analysis tools (linters, type checkers) to catch dead code, shadowed variables, and type issues before code review
---

# Static Analysis

## Overview

Run the project's static analysis tools (linters, type checkers, bundlers) to catch issues that manual review misses. Static analysis catches dead code, type errors, shadowed variables, and style violations automatically.

**Core principle:** Let tools catch what tools catch best.

**Announce at start:** "I'm using the static-analysis skill to run linters and type checkers."

## When to Use

**Mandatory:**
- Before any code review
- Before merging
- After significant refactoring

**Valuable:**
- When debugging type issues
- After adding new dependencies
- When CI fails locally

## The Process

### Step 1: Identify Project Tools

**Check for configuration files:**

```bash
# JavaScript/TypeScript
ls package.json tsconfig.json .eslintrc* .prettierrc* biome.json 2>/dev/null

# Python
ls pyproject.toml setup.cfg .flake8 .pylintrc mypy.ini 2>/dev/null

# Rust
ls Cargo.toml rustfmt.toml clippy.toml 2>/dev/null

# Go
ls go.mod .golangci.yml 2>/dev/null
```

**Check package.json scripts:**
```bash
cat package.json | grep -A 20 '"scripts"'
```

### Step 2: Run Type Checker

**TypeScript:**
```bash
# Full type check
npx tsc --noEmit

# Or project script
npm run typecheck
npm run type-check
npm run check-types
```

**Python (mypy):**
```bash
mypy src/
```

**Common type issues to fix:**
- Implicit `any` types
- Missing return types
- Nullable access without guards
- Type assertion abuse

### Step 3: Run Linter

**ESLint:**
```bash
# Run with auto-fix for style issues
npx eslint src/ --fix

# Run without fix to see all issues
npx eslint src/
```

**ESLint rules that catch dead code:**
- `no-unused-vars` - Unused variables
- `no-unreachable` - Unreachable code
- `no-shadow` - Shadowed variables
- `@typescript-eslint/no-unused-vars` - TS-aware unused vars

**Python (flake8/pylint):**
```bash
flake8 src/
pylint src/
```

**Rust (clippy):**
```bash
cargo clippy -- -W clippy::all
```

### Step 4: Run Bundle Analysis (if applicable)

**Check bundle size:**
```bash
# Webpack
npx webpack --analyze

# Vite
npx vite build --analyze

# Or custom script
npm run build:analyze
npm run analyze
```

**Check for:**
- Unexpectedly large bundles
- Duplicate dependencies
- Unused imports increasing size

### Step 5: Check Size Limits

**If project has size limits documented:**
```bash
# Count lines per file
find src -name "*.ts" -exec wc -l {} \; | sort -rn | head -20

# Check function sizes (rough)
grep -n "function\|=>" src/**/*.ts | head -50
```

### Step 6: Report Findings

**Output Format:**

```markdown
## Static Analysis Results

### Tools Run
- [x] TypeScript (`tsc --noEmit`): [PASS/FAIL]
- [x] ESLint (`eslint src/`): [PASS/FAIL - N issues]
- [x] Bundle Analysis: [N/A or results]

### Type Errors

1. **[Error Code]: [Message]**
   - File: `path/to/file.ts:line:col`
   - Code: `problematic code snippet`
   - Fix: [How to fix]

### Linter Issues

#### Errors (Must Fix)

1. **[Rule]: [Message]**
   - File: `path/to/file.ts:line`
   - Fix: [Auto-fixable? How to fix]

#### Warnings (Should Fix)

[Same format]

### Dead Code Detected

1. **Unused variable: `varName`**
   - File: `path/to/file.ts:line`
   - Rule: `no-unused-vars`
   - Fix: Remove or use

2. **Shadowed variable: `varName`**
   - Outer: `file.ts:10`
   - Inner: `file.ts:25`
   - Rule: `no-shadow`
   - Fix: Rename inner variable

### Size Analysis

| Metric | Value | Limit | Status |
|--------|-------|-------|--------|
| Largest file | N LOC | 300 | OK/OVER |
| Largest function | N LOC | 50 | OK/OVER |
| Bundle size | N KB | N/A | - |

### Assessment

**Static Analysis Passed:** [Yes/No/With fixes]

**Issues to fix:** [N errors, M warnings]
```

## Tool-Specific Commands

### TypeScript Projects

```bash
# Type check
npx tsc --noEmit

# ESLint with TypeScript
npx eslint . --ext .ts,.tsx

# Find unused exports (requires plugin)
npx ts-prune

# Check for circular dependencies
npx madge --circular src/
```

### JavaScript Projects

```bash
# ESLint
npx eslint src/

# Find unused dependencies
npx depcheck
```

### Python Projects

```bash
# Type check
mypy src/ --strict

# Linting
flake8 src/
pylint src/

# Dead code
vulture src/
```

### Rust Projects

```bash
# Clippy (comprehensive linter)
cargo clippy -- -D warnings

# Format check
cargo fmt -- --check

# Unused dependencies
cargo +nightly udeps
```

## Integration

**Called by:**
- **subagent-driven-development** - Before code review
- **finishing-a-development-branch** - Before merge options

**Pairs with:**
- **cross-file-dry-analysis** - Static catches different issues
- **standards-aware-review** - After static passes

## Common Issues Caught

| Issue | Tool | Rule |
|-------|------|------|
| Unused variables | ESLint | `no-unused-vars` |
| Shadowed variables | ESLint | `no-shadow` |
| Unreachable code | ESLint | `no-unreachable` |
| Type mismatches | TypeScript | N/A |
| Missing null checks | TypeScript | strict mode |
| Implicit any | TypeScript | `noImplicitAny` |
| Unused imports | ESLint | `no-unused-imports` |
| Console statements | ESLint | `no-console` |

## Red Flags

**Never:**
- Skip static analysis because "it takes too long"
- Disable rules to make issues go away
- Ignore warnings "for now"
- Commit with type errors

**Always:**
- Fix all errors before review
- Address warnings or document exceptions
- Run full analysis, not just changed files
- Check CI matches local results

## Troubleshooting

**"Tool not found":**
```bash
# Install dev dependencies
npm install
# or
npm ci
```

**"Too many errors":**
```bash
# Run on changed files only
npx eslint $(git diff --name-only origin/main | grep -E '\.(ts|js)$')
```

**"CI fails but local passes":**
```bash
# Match CI environment
rm -rf node_modules
npm ci
npm run lint
```
