---
name: standards-aware-review
description: Use when reviewing code against project-specific architecture, coding, and security standards - reads project docs and enforces compliance
---

# Standards-Aware Code Review

## Overview

Review code against project-specific standards documented in the codebase. Unlike generic code review, this skill ensures compliance with documented architecture patterns, coding conventions, and security requirements.

**Core principle:** Standards exist for a reason - enforce them.

**Announce at start:** "I'm using the standards-aware-review skill to check compliance with project standards."

## When to Use

**Mandatory:**
- Before merging any feature branch
- After major refactoring
- When modifying security-sensitive code

**Valuable:**
- During code review to catch standards violations early
- When onboarding to ensure consistent patterns
- After receiving code review feedback

## The Process

### Step 1: Find Project Standards

Search for standards documentation:

```bash
# Common locations
find . -name "*standard*.md" -o -name "CLAUDE.md" -o -name "ARCHITECTURE.md" 2>/dev/null
ls docs/*.md 2>/dev/null
```

**Standard files to look for:**
- `docs/architecture-standard.md` - Layer boundaries, patterns
- `docs/coding-standard.md` - Implementation conventions
- `docs/security-standard.md` - Security requirements
- `CLAUDE.md` - Project-specific instructions

**If no standards found:** Skip to generic review, note that standards should be documented.

### Step 2: Read Standards Completely

**CRITICAL:** Read ALL standards files before reviewing any code.

For each standard file:
1. Read the entire document
2. Note the "Merge Checklist" or "Required" sections
3. Identify concrete, verifiable rules

### Step 3: Review Against Each Standard

For each standards document, create a compliance checklist:

**Architecture Standard Check:**
- [ ] Layer boundaries respected?
- [ ] No prohibited cross-layer dependencies?
- [ ] Documented patterns followed?
- [ ] Extension points used correctly?

**Coding Standard Check:**
- [ ] Naming conventions followed?
- [ ] File size limits respected?
- [ ] Function complexity within limits?
- [ ] Required patterns used (error handling, etc.)?

**Security Standard Check:**
- [ ] Data minimization followed?
- [ ] Broadcast security maintained?
- [ ] No sensitive data in logs?
- [ ] Input validation present?

### Step 4: Report Findings

**Output Format:**

```markdown
## Standards Compliance Review

### Standards Checked
- [x] architecture-standard.md
- [x] coding-standard.md
- [x] security-standard.md

### Violations Found

#### Critical (Must Fix Before Merge)

1. **[Standard Name] Section X.Y Violation**
   - File: `path/to/file.ts:123`
   - Standard says: "Quote the specific requirement"
   - Code does: Description of violation
   - Fix: How to fix it

#### Important (Should Fix)

...

#### Minor (Nice to Have)

...

### Compliant Areas
- [What was done correctly, with specific examples]

### Assessment

**Standards Compliant:** [Yes/No/With fixes]

**Reasoning:** [Brief technical assessment]
```

## Subagent Dispatch

When dispatching as a subagent, use template at `standards-reviewer-prompt.md`:

```
Task tool (superpowers:standards-aware-review):
  Use template at standards-aware-review/standards-reviewer-prompt.md

  BASE_SHA: [commit before changes]
  HEAD_SHA: [current commit]
  DESCRIPTION: [what was implemented]
```

## Integration

**Called by:**
- **finishing-a-development-branch** (Step 1c) - Before presenting merge options
- **subagent-driven-development** - After code quality review

**Pairs with:**
- **cross-file-dry-analysis** - Check DRY after standards
- **code-reviewer** - General code quality (run standards first)

## Red Flags

**Never:**
- Review without reading standards first
- Skip a standard because "it doesn't apply"
- Mark compliant without checking each rule
- Give vague feedback ("improve security")

**Always:**
- Quote the specific standard section violated
- Provide file:line references
- Explain how to fix violations
- Acknowledge what's done well

## Example Output

```markdown
## Standards Compliance Review

### Standards Checked
- [x] architecture-standard.md
- [x] coding-standard.md
- [x] security-standard.md

### Violations Found

#### Critical

1. **Coding Standard Section 4.2: Function Complexity**
   - File: `src/handlers/message.ts:45-120`
   - Standard says: "Functions must be < 50 LOC"
   - Code does: `handleMessage()` is 75 LOC
   - Fix: Extract URL detection and metadata parsing into separate functions

2. **Security Standard Section 2.1: Data Minimization**
   - File: `src/content/relay.ts:89`
   - Standard says: "Only include data required for feature"
   - Code does: Broadcasts full `post` object including unused fields
   - Fix: Create minimal interface with only `id`, `text`, `author`

#### Minor

1. **Coding Standard Section 3.1: Naming**
   - File: `src/utils/parse.ts:12`
   - Standard says: "Use descriptive names, no abbreviations"
   - Code does: Variable named `msg` instead of `message`
   - Fix: Rename to `message`

### Compliant Areas
- Architecture layers properly respected (content -> shared -> background)
- Error handling follows documented patterns
- Type safety maintained throughout

### Assessment

**Standards Compliant:** With fixes

**Reasoning:** Two critical violations (function size, data minimization) need fixing before merge. Architecture and security patterns otherwise well-followed.
```
