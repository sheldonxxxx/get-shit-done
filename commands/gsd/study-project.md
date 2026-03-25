---
name: gsd:study-project
description: Analyze a cloned open-source project in depth to extract learnings for building your own project in the same domain
argument-hint: "<path-to-cloned-repo> [optional: focus area]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Task
---

<objective>
Analyze a cloned open-source project deeply and produce a structured learning folder with findings relevant to building your own project in the same domain.

**Important:** The project must be cloned locally — this command does not clone repos. Pass the absolute path to a cloned repository.
</objective>

<core-principle>
**Code Over Docs:** Never treat README or documentation as the source of truth. The documentation describes *intended* behavior; the code reveals *actual* behavior. Always verify claims by reading the real implementation. Discrepancies between docs and code are among the most valuable findings.
</core-principle>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-planner — Use for ALL subagents in this workflow
</available_agent_types>

<context>
**Required input:** `$ARGUMENTS` — must contain an absolute path to a cloned repository.

**Optional:** Focus area (e.g., "focus on the auth system")

**Example:**
- Input: `/Users/sheldon/projects/axios` or `./axios-study/`
- Study root: `/Users/sheldon/projects/axios-study/`
</context>

<output_structure>
```
{project-name}-study/
├── {project-name}-learning/     # Final synthesized learning documents
└── research/                    # Intermediate research (preserve — proves subagent use)
    ├── 01-topology.md
    ├── 02-tech-stack.md
    ├── 03-community.md
    ├── 04-architecture.md
    ├── 05-features.md
    ├── 06-code-quality.md
    └── 07-security-perf.md
```
</output_structure>

<process>

## 0. Validate Input

Extract the repo path from $ARGUMENTS. The path must point to an existing cloned repository.

**If no path provided:** Error — a path to a cloned repo is required.

**If path doesn't exist or isn't a git repo:** Error with helpful message.

## 1. Determine Output Paths

1. Extract project name from the repo path (e.g., `/path/to/axios` → `axios`)
2. Create the study root: `{project-name}-study/` as sibling to the cloned repo or in current working directory
3. Create subdirectories: `{project-name}-study/research/`

## 2. Resolve Planner Model

```bash
PLANNER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-planner --raw)
```

## 3. Spawn Subagents (Phase 1 — 3 parallel)

Spawn 3 subagents **in parallel**, each using `subagent_type="gsd-planner"` and model=`$PLANNER_MODEL`.

### Subagent A — Project Topology
- **Task:** Explore the project structure and write findings
- **Inputs:**
  - `repo_path`: extracted repo path
  - `output_file`: `/path/to/{project-name}-study/research/01-topology.md`
- **Actions:**
  - Run `ls -la` at root, explore 2-3 levels deep
  - Identify project type, languages, build system
  - Find key entry points
  - **Use Write tool** to create file at `output_file`

### Subagent B — Tech Stack & Choices
- **Task:** Analyze dependencies and config
- **Inputs:**
  - `repo_path`: extracted repo path
  - `output_file`: `/path/to/{project-name}-study/research/02-tech-stack.md`
- **Actions:**
  - Examine dependencies (`package.json`, `go.mod`, `Cargo.toml`, etc.)
  - Identify framework versions and rationale
  - Look for CI/CD configs, Docker files
  - **Use Write tool** to create file at `output_file`

### Subagent C — Community & Metadata
- **Task:** Extract project metadata
- **Inputs:**
  - `repo_path`: extracted repo path
  - `output_file`: `/path/to/{project-name}-study/research/03-community.md`
- **Actions:**
  - Check `package.json`, `Cargo.toml` for version, license, author
  - Look at `.github/` for contributing guidelines
  - Note repository activity from git history
  - **Use Write tool** to create file at `output_file`

## 4. Spawn Subagents (Phase 2 — 4 parallel)

After Phase 1 completes, spawn 4 more subagents **in parallel**.

### Subagent D — Architecture & Design Patterns
- **Task:** Analyze architecture
- **Inputs:**
  - `repo_path`: extracted repo path
  - `research_dir`: `/path/to/{project-name}-study/research/`
  - `output_file`: `/path/to/{project-name}-study/research/04-architecture.md`
- **Actions:**
  - Read `research/01-topology.md` for context
  - Identify architectural pattern (MVC, microservices, event-driven, hexagonal, layered, etc.)
  - Find key abstractions, interfaces, base classes
  - Trace how components communicate
  - Look for design patterns in actual use
  - Map out major modules and their responsibilities
  - **Use Write tool** to create file

### Subagent E — Feature Implementation Deep Dive
- **Task:** Analyze key features
- **Inputs:**
  - `repo_path`: extracted repo path
  - `research_dir`: `/path/to/{project-name}-study/research/`
  - `output_file`: `/path/to/{project-name}-study/research/05-features.md`
- **Actions:**
  - Read `research/01-topology.md` and `research/04-architecture.md` for context
  - Pick the 3-5 most important/interesting features
  - For each: find core implementation file(s) and read them thoroughly
  - Trace how the feature flows through the codebase
  - Note clever solutions, shortcuts, or technical debt
  - Pay attention to error handling, validation, edge cases
  - **Use Write tool** to create file

### Subagent F — Code Quality & Practices
- **Task:** Assess code quality
- **Inputs:**
  - `repo_path`: extracted repo path
  - `research_dir`: `/path/to/{project-name}-study/research/`
  - `output_file`: `/path/to/{project-name}-study/research/06-code-quality.md`
- **Actions:**
  - Read `research/02-tech-stack.md` for context
  - Examine actual test files — not just that tests exist, but what they cover
  - Look for type systems (TypeScript, mypy, Rust types, etc.)
  - Check linting/formatting config (ESLint, Prettier, rustfmt, gofmt)
  - Identify code review patterns (PR descriptions, commit message style)
  - Note patterns around: error handling, logging, config management, secrets
  - **Use Write tool** to create file

### Subagent G — Security & Performance Patterns
- **Task:** Analyze security and performance
- **Inputs:**
  - `repo_path`: extracted repo path
  - `research_dir`: `/path/to/{project-name}-study/research/`
  - `output_file`: `/path/to/{project-name}-study/research/07-security-perf.md`
- **Actions:**
  - Read `research/02-tech-stack.md` and `research/06-code-quality.md` for context
  - Look for how secrets are managed (env vars, vault, config files)
  - Examine authentication/authorization implementation if present
  - Check for input validation and sanitization
  - Look for performance patterns: caching, pagination, lazy loading, query optimization
  - **Use Write tool** to create file

## 5. Synthesis

After all 7 subagents complete, read all `research/*.md` files and synthesize them into the final learning folder.

**Output location**: `{project-name}-learning/` within the project study root

**Default structure** (adapt based on project type):
```
{project-name}-study/
├── {project-name}-learning/
│   ├── README.md                  # High-level summary, key takeaways
│   ├── 01-project-overview.md
│   ├── 02-architecture.md
│   ├── 03-tech-stack.md
│   ├── 04-features-deep-dive.md
│   ├── 05-code-quality.md
│   ├── 06-ci-cd.md
│   ├── 07-documentation.md
│   ├── 08-security.md
│   ├── 09-dependencies.md
│   ├── 10-community.md
│   ├── 11-patterns.md
│   ├── 12-lessons-learned.md
│   ├── 13-my-action-items.md
│   ├── assets/
│   └── snippets/
└── research/                      # PRESERVE — evidence of parallel subagent work
```

**README.md template:**
```markdown
# [Project Name] — Learning Reference

## What This Is
Analysis of the [Project] codebase for the purpose of informing development of [your similar project].

## Key Takeaways
- [3-5 bullet points of the most important things learned]

## Should Trust / Should Not Trust
- **Trust**: Architecture decisions, code patterns, test coverage
- **Do Not Trust**: README claims about simplicity (verify with code)

## At a Glance
- Language: ...
- Architecture: ...
- Key Libraries: ...
- Notable Patterns: ...
- Stars / Activity: ...
- License: ...
```

## Subagent Guidelines

1. **Read actual files.** Every claim must be backed by code evidence, not just file names.
2. **Be specific.** Instead of "good error handling," say "uses custom error types with stack traces in `errors/` package."
3. **Be honest about bad findings.** Don't soften criticisms — they're the most valuable learnings.
4. **Quantify when possible.** "0.1% test coverage" is better than "low test coverage."
5. **Note missing things.** If a project lacks tests, documentation, or type safety, that is a finding.
6. **Preserve file paths.** Always reference which file a finding came from.
7. **Use code snippets.** Copy relevant code sections verbatim when they illustrate a point.
8. **Respect `.gitignore` and privacy.** Don't copy secrets, keys, or proprietary snippets.

## Handling Edge Cases

- **Monorepos**: Analyze the most relevant sub-package, note the monorepo structure benefits
- **Very large projects**: Focus on the core package, note that analysis was scoped
- **Microservices**: Pick 2-3 representative services for deep dive
- **Minimal projects**: Adapt — don't create empty sections; focus on what's there
- **Unfamiliar language**: Still analyze architecture and patterns; note language-specific conventions

## Task Call Template

```python
Task(
  prompt=f"""[subagent prompt from above]""",
  subagent_type="gsd-planner",
  model="$PLANNER_MODEL"
)
```
