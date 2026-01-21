# Superpowers

Superpowers is a complete software development workflow for your coding agents, built on top of a set of composable "skills" and some initial instructions that make sure your agent uses them.
 
## How it works

It starts from the moment you fire up your coding agent. As soon as it sees that you're building something, it *doesn't* just jump into trying to write code. Instead, it steps back and asks you what you're really trying to do. 

Once it's teased a spec out of the conversation, it shows it to you in chunks short enough to actually read and digest. 

After you've signed off on the design, your agent puts together an implementation plan that's clear enough for an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing to follow. It emphasizes true red/green TDD, YAGNI (You Aren't Gonna Need It), and DRY. 

Next up, once you say "go", it launches a *subagent-driven-development* process, having agents work through each engineering task, inspecting and reviewing their work, and continuing forward. It's not uncommon for Claude to be able to work autonomously for a couple hours at a time without deviating from the plan you put together.

There's a bunch more to it, but that's the core of the system. And because the skills trigger automatically, you don't need to do anything special. Your coding agent just has Superpowers.


## Sponsorship

If Superpowers has helped you do stuff that makes money and you are so inclined, I'd greatly appreciate it if you'd consider [sponsoring my opensource work](https://github.com/sponsors/obra).

Thanks! 

- Jesse


## Installation

**Note:** Installation differs by platform. Claude Code has a built-in plugin system. Codex and OpenCode require manual setup.

### Claude Code (via Plugin Marketplace)

In Claude Code, register the marketplace first:

```bash
/plugin marketplace add obra/superpowers-marketplace
```

Then install the plugin from this marketplace:

```bash
/plugin install superpowers@superpowers-marketplace
```

### Verify Installation

Check that commands appear:

```bash
/help
```

```
# Should see:
# /superpowers:brainstorm - Interactive design refinement
# /superpowers:write-plan - Create implementation plan
# /superpowers:execute-plan - Execute plan in batches
```

### Codex

Tell Codex:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

**Detailed docs:** [docs/README.codex.md](docs/README.codex.md)

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**Detailed docs:** [docs/README.opencode.md](docs/README.opencode.md)

## The Basic Workflow

1. **brainstorming** - Activates before writing code. **Reads project standards first** (architecture, coding, security docs). Refines rough ideas through questions, explores alternatives, presents design in sections for validation. Rejects approaches that violate standards. Saves design document.

2. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

3. **writing-plans** - Activates with approved design. **Searches for existing patterns first** (shared/, utils/, primitives/). Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps, and reuse references.

4. **subagent-driven-development** or **executing-plans** - Activates with plan. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.

5. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.

6. **requesting-code-review** - Activates between tasks. Reviews against plan, **checks project standards**, **detects dead code and DRY violations**, reports issues by severity. Critical issues block progress.

7. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, **runs cross-file DRY analysis**, **runs standards compliance check**, then presents options (merge/PR/keep/discard), cleans up worktree.

**The agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

### Enhanced Quality Gates

The workflow includes three new quality gates that catch issues task-scoped reviews miss:

```
Tests Pass → Cross-File DRY Analysis → Standards Check → Merge Options
                    ↓                        ↓
            Catches:                  Catches:
            - Duplicated patterns     - Architecture violations
            - Dead code               - Security issues
            - Shadowed variables      - Coding standard violations
```

**If any gate fails:** Fix issues, commit, re-run analysis until clean.

## What's Inside

### Skills Library

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**
- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure it's actually fixed

**Quality Assurance**
- **standards-aware-review** - Reviews code against project-specific standards (architecture, coding, security). Reads docs first, cites violations with section references.
- **cross-file-dry-analysis** - Scans entire codebase for DRY violations, duplicated patterns, dead code, and shadowed variables. Catches issues task-scoped reviews miss.
- **static-analysis** - Runs project's linters and type checkers (ESLint, TypeScript, etc.) to catch dead code and type issues automatically.

**Collaboration**
- **brainstorming** - Socratic design refinement. Now reads project standards before exploring approaches.
- **writing-plans** - Detailed implementation plans. Now searches for existing patterns before planning new code.
- **executing-plans** - Batch execution with checkpoints
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **requesting-code-review** - Pre-review checklist. Now includes dead code detection and DRY violation checks.
- **receiving-code-review** - Responding to feedback
- **using-git-worktrees** - Parallel development branches
- **finishing-a-development-branch** - Merge/PR decision workflow. Now runs cross-file analysis and standards check before presenting options.
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)

**Meta**
- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-superpowers** - Introduction to the skills system

## Workflow Verification Checklist

Use this checklist to verify the enhanced workflow is functioning correctly when reviewing implementation logs:

### Brainstorming Phase
- [ ] Agent announced "I'm using the brainstorming skill"
- [ ] Agent searched for `*standard*.md` and `CLAUDE.md` files
- [ ] Agent read project standards before proposing approaches
- [ ] Agent mentioned standards constraints when presenting design options

### Planning Phase
- [ ] Agent announced "I'm using the writing-plans skill"
- [ ] Agent searched `shared/`, `utils/`, `primitives/` for existing patterns
- [ ] Plan includes "Reuse Analysis" or "Existing Patterns" section
- [ ] Individual tasks include "Reuse from" references where applicable

### Code Review Phase
- [ ] Reviewer checked for dead code (unused variables, imports)
- [ ] Reviewer searched codebase for duplicated patterns
- [ ] Reviewer checked against project standards (if present)
- [ ] Reviewer reported DRY violations with file:line references

### Finishing Phase
- [ ] Agent ran tests first (Step 1)
- [ ] Agent dispatched `cross-file-dry-analysis` subagent (Step 1b)
- [ ] Agent dispatched `standards-aware-review` subagent (Step 1c)
- [ ] Agent fixed any violations before presenting merge options
- [ ] Only after all checks passed: presented 4 merge/PR options

### Issues That Should Be Caught
| Issue Type | Caught By | Expected Output |
|------------|-----------|-----------------|
| DRY violation (duplicated pattern) | cross-file-dry-analysis | "Duplicated pattern: X at file1:line, file2:line" |
| Dead code (unused export) | cross-file-dry-analysis, static-analysis | "Unused export: X at file:line" |
| Shadowed variable | code-quality-reviewer, static-analysis | "Shadowed: X at outer:line, inner:line" |
| Architecture violation | standards-aware-review | "[Standard Name] Section X.Y: violation at file:line" |
| Security violation | standards-aware-review | "Security Standard Section X: violation" |

## Philosophy

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success

Read more: [Superpowers for Claude Code](https://blog.fsck.com/2025/10/09/superpowers/)

## Contributing

Skills live directly in this repository. To contribute:

1. Fork the repository
2. Create a branch for your skill
3. Follow the `writing-skills` skill for creating and testing new skills
4. Submit a PR

See `skills/writing-skills/SKILL.md` for the complete guide.

## Updating

Skills update automatically when you update the plugin:

```bash
/plugin update superpowers
```

## License

MIT License - see LICENSE file for details

## Support

- **Issues**: https://github.com/obra/superpowers/issues
- **Marketplace**: https://github.com/obra/superpowers-marketplace
