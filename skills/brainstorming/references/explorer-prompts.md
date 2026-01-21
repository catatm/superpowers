# Explorer Prompts for Parallel Codebase Exploration

These prompts are designed for dispatching concurrent Explore agents during the brainstorming phase. Each agent focuses on a specific aspect of codebase intelligence gathering.

## 1. Pattern Finder

**Purpose:** Discover existing patterns in the codebase that are similar to what we're building.

**Prompt:**
```
Search for existing implementations similar to: [FEATURE_DESCRIPTION]

Focus on:
- Similar features or components already in the codebase
- Naming conventions used for similar concepts
- How existing similar features are structured (files, folders, modules)
- Common abstractions or base classes that might apply
- Utility functions or helpers that could be reused

Search strategy:
1. Grep for keywords related to the feature concept
2. Look in directories where similar functionality lives
3. Check for existing interfaces or types that define contracts
4. Find factory patterns, builders, or configuration approaches used

Report: List 3-5 most relevant existing patterns with file locations and brief descriptions of how they work.
```

## 2. Architecture Mapper

**Purpose:** Map integration points, boundaries, and dependencies for the new feature.

**Prompt:**
```
Map the architecture context for adding: [FEATURE_DESCRIPTION]

Focus on:
- Entry points where this feature would integrate
- Existing module boundaries and layer separations
- Dependency injection patterns or service registries
- Configuration systems and how features are enabled/disabled
- Event systems, message buses, or hooks that might be relevant

Search strategy:
1. Find the module/package structure and identify where this belongs
2. Look for index files, barrel exports, or public APIs
3. Check for DI containers, context providers, or service locators
4. Identify cross-cutting concerns (logging, auth, caching) that apply

Report: Describe the integration landscape - where this fits, what it connects to, and what boundaries must be respected.
```

## 3. Test Pattern Scout

**Purpose:** Discover testing conventions and requirements for the codebase.

**Prompt:**
```
Scout testing patterns and conventions for: [FEATURE_DESCRIPTION]

Focus on:
- Test file organization (co-located vs separate test directories)
- Testing frameworks and assertion libraries in use
- Mocking strategies for dependencies and external services
- Test data factories, fixtures, or builders
- Integration test patterns vs unit test patterns
- Coverage requirements or testing standards

Search strategy:
1. Find test files for similar features
2. Look for test utilities, helpers, or shared test infrastructure
3. Check for testing documentation or conventions
4. Identify any test configuration files (jest.config, pytest.ini, etc.)

Report: Summarize testing conventions with examples of how similar features are tested, including any patterns that MUST be followed.
```

## 4. Prior Art Researcher

**Purpose:** Find prior attempts, TODOs, discussions, or related work in the codebase.

**Prompt:**
```
Research prior art and related work for: [FEATURE_DESCRIPTION]

Focus on:
- TODO comments mentioning this feature or related concepts
- FIXME or HACK comments that might be addressed by this feature
- Git history showing prior attempts or reverted implementations
- Comments referencing issues, RFCs, or design documents
- Deprecated code that attempted similar functionality
- README notes or documentation mentioning future plans

Search strategy:
1. Grep for TODO, FIXME, HACK with related keywords
2. Search for commented-out code related to this feature
3. Look for design docs or ADRs in docs/ folders
4. Check for related constants, feature flags, or config that hints at prior work

Report: List any prior art found - TODOs to address, prior attempts to learn from, or existing discussions to reference.
```

## Usage

When dispatching these agents, replace `[FEATURE_DESCRIPTION]` with a concise description of what's being designed (e.g., "user notification system", "caching layer for API responses", "form validation framework").

All four agents should be dispatched in parallel using a single message with multiple Task tool calls:

```
Use the Task tool with subagent_type=Explore for each of the 4 prompts above,
customizing [FEATURE_DESCRIPTION] for the current design session.
```
