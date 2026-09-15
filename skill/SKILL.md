---
name: make-readme
description: Use to generate or improve a project's README.md — analyzes the codebase and produces a tailored, well-structured README with optional companion files
---

# Generate or Improve README

Analyze the current project and either generate a complete README.md from scratch or improve an existing one. Then offer to create companion documentation files.

## Arguments

If arguments are provided after `/make-readme`, parse them as key=value pairs:

- `audience`: `developers` | `end-users` | `data-scientists` | `mixed` (default: auto-detect from project type)
- `type`: `library` | `cli` | `webapp` | `api` | `monorepo` (default: auto-detect)
- `tone`: `formal` | `casual` | `minimal` | `playful` (default: `professional` for create mode, `match-existing` for improve mode)
- `dry-run`: if present (no value needed), show the analysis, section plan, and gap report without generating any content. Useful for previewing what the skill would do.
- `--help`: if present, do not run any of the steps below. Instead, read and display the contents of `help.md` (in this skill's folder) verbatim, then stop.
- `--version`: if present, do not run any of the steps below. Instead: (1) read the installed version from `.claude-plugin/plugin.json` if present, else `.codex-plugin/plugin.json`, else `gemini-extension.json`, else (bare Claude Code install) the topmost version heading in `CHANGELOG.md`; (2) print `make-readme v<installed-version>`; (3) best-effort update check — find this skill's GitHub source repo from `git remote get-url origin` if this is a git checkout on `github.com`, else the first `https://github.com/<owner>/<repo>` URL in `README.md`; if no repo is found or `gh` isn't installed/authenticated, stop here with no further output; (4) otherwise run `gh api repos/<owner>/<repo>/releases/latest -q .tag_name` (strip leading `v`) and append a status line: `Status: up to date` if equal, `Status: newer version available (v<latest>). To update: if you installed this via a Claude Code marketplace, run /plugin marketplace update <marketplace-name> then reinstall; otherwise, git pull in your install directory if it's a git checkout, or re-copy from https://github.com/<owner>/<repo> per this README's Installation section.` if installed is older, or `Status: ahead of latest release (development checkout)` if installed is newer; on any API failure, print nothing further. Then stop, without running any of the steps below.

If not provided, auto-detect all three from the codebase analysis.

## Step 1: Analyze the Codebase

Scan the project root and gather information from these sources. Read files where they exist — don't guess.

**Package manifests** (pick the one that exists):
- `package.json` — name, description, scripts, dependencies, license, workspaces
- `pyproject.toml` / `setup.py` / `setup.cfg` — name, description, dependencies, scripts/entry points
- `Cargo.toml` — name, description, lib/bin targets, dependencies, categories, keywords
- `go.mod` — module name, Go version, dependencies
- `pom.xml` / `build.gradle` — artifact, dependencies, plugins
- `Gemfile` / `.gemspec` — name, dependencies
- `composer.json` — name, description, dependencies

**Directory structure**: Run `ls -la` at root and note key directories (src/, lib/, tests/, docs/, examples/, .github/, packages/, apps/).

**CI/CD config**: `.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/config.yml`, `.travis.yml`. For each command a workflow actually runs (lint, type-check, test, build), confirm the tool it invokes is declared in the dependency manifest or explicitly installed by the workflow (e.g. a separate `pip install`/`npm install -g` step, or an `actions/setup-*` action known to bundle it). A step that shells out to a tool nothing installs is very likely broken — note this as a finding rather than describing the pipeline as working in Step 4d or 5b.

**Phased plan or roadmap docs**: a spec/plan file with numbered phases or milestones — `spec.md`, `ROADMAP.md`, `docs/plans/`, a PRD, or similar. If found, this feeds the optional Status section (Step 4b) — do not just copy its checkboxes; verify each phase against the code, tests, and git history (see Status generation rules in Step 4d).

**Existing documentation**: `README.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md`, `SECURITY.md`, `LICENSE`, `docs/`

**Configuration files**: `.env.example`, `.env.sample`, `config/`, `*.config.js`, `*.config.ts`, `docker-compose.yml`, `Dockerfile`

**Entry points**: `src/index.*`, `src/main.*`, `src/lib.*`, `src/cli.*`, `bin/`, `main.go`, `main.py`, `app.*`

Record your findings internally. Do not present this raw analysis to the user.

## Step 2: Detect Project Type

Based on Step 1, classify the project:

| Signal | Type |
|--------|------|
| `workspaces` in package.json, or `packages/` + `apps/` dirs, or `turbo.json`/`lerna.json`/`pnpm-workspace.yaml` | **monorepo** |
| `bin` field in manifest, or `[project.scripts]` in pyproject.toml, or CLI framework deps (commander, click, clap, cobra) | **cli** |
| Express/Fastify/Flask/Django/Gin/Actix/Spring + route files | **api** |
| React/Vue/Angular/Svelte/Next.js + component dirs | **webapp** |
| `[lib]` in Cargo.toml, or `main` field with no bin, or library-pattern exports | **library** |
| Clearly a framework or toolkit (plugin system, middleware, extensibility) | **framework** |
| None of the above | **other** |

Also note:
- **Language**: JavaScript/TypeScript, Python, Rust, Go, Java, Ruby, PHP, etc.
- **Framework**: Express, React, Click, Actix, etc.
- **Test framework**: Jest, Vitest, pytest, cargo test, etc.
- **Build tool**: npm, pip, cargo, turbo, etc.

## Step 3: Determine Mode

- If `README.md` exists at project root and has more than just a title → **Improve Mode** (go to Step 5)
- If `README.md` does not exist, or exists with only a title / blank content → **Create Mode** (go to Step 4)

## Step 4: Create Mode

Generate a complete README.md from scratch. Follow these sub-steps in order.

### 4a: Ask About Badges

Ask the user one question: "Would you like badges (build status, version, license, etc.) at the top of the README?"

If yes, select 3-6 relevant badges from this list using shields.io format:
- Build status (only if CI config exists)
- Test coverage (only if coverage tooling is configured)
- Package version (only if published to a registry)
- License (only if license is specified)
- Downloads (only if published to a registry)
- Last commit

Use this format: `![Badge Name](https://img.shields.io/badge/...)` or the appropriate shields.io endpoint for the service.

### 4b: Select Sections

Choose which sections to include based on the project:

**Always include:**
1. Title (H1, project name)
2. Description (what/why/how, 2-4 sentences)
3. Highlights (3-6 bullet points of key features or design goals)
4. Getting Started (prerequisites + installation, copy-paste-ready)
5. Usage (code examples or CLI invocations)
6. Development (how to build, test, lint — for contributors)
7. Contributing (brief guidance or link to CONTRIBUTING.md)
8. License (name + link to LICENSE file)

**Include conditionally:**
- **Badges**: Only if user opted in (Step 4a)
- **Table of Contents**: Only if the README has more than 5 sections
- **API Reference**: Only for libraries — document main exports, function signatures, options
- **Configuration**: Only if `.env.example`, config files, or environment variables are detected. Use table format.
- **Architecture**: Only for monorepos or complex projects — describe package/module structure
- **Roadmap**: Only if there's evidence (GitHub milestones, TODO comments, roadmap file)
- **Status**: Only if the project has an explicit phased plan (see Step 1). One row per phase, verified against code/tests/git history — not the plan doc's own checkboxes
- **Limitations**: Only for concrete, already-true gaps or deliberate scope cuts you've verified are real — not prospective work (that's Roadmap)
- **Acknowledgments**: Only if the project clearly builds on other notable work

### 4c: Dry-Run Check

If `dry-run` was specified, present the analysis summary to the user and stop:

1. Show detected project type, language, framework, build tool, test framework
2. Show the section list from 4b — which sections will be included and why
3. Note which conditional sections were included or excluded, with the reason
4. Do NOT generate any content. Do NOT ask about badges. Do NOT offer companion files.
5. Stop here.

### 4d: Generate Each Section

Follow these rules for each section:

**Title**
- Use the project name from the manifest
- If badges were requested, place them on the line immediately after the H1

**Description**
- 2-4 sentences covering: what it does, why it exists, how it works (at a high level)
- Do NOT start with "This project is a..." or repeat the title
- Do NOT use buzzwords ("blazing fast", "cutting-edge", "seamless", "robust")
- Lead with what a user gets, not what the technology is

**Highlights**
- 3-6 bullet points
- Each bullet is a concrete feature or design goal, not marketing fluff
- Start each with a bold keyword: **Thread-safe** — ..., **Zero dependencies** — ..., etc.

**Getting Started**
- Split into Prerequisites and Installation sub-sections
- Prerequisites: list runtime version, database, external services — only what's actually needed
- Installation: copy-paste-ready commands from the actual manifest
  - npm/yarn/pnpm: show `npm install <package>` or `git clone` + `npm install`
  - pip: show `pip install <package>` or `git clone` + `pip install -e ".[dev]"`
  - cargo: show `cargo add <package>` or the Cargo.toml dependency line
  - Go: show `go get` or `go install`
- Every command in a fenced code block with the correct language hint (`bash`, `sh`, `toml`, etc.)

**Usage**
- For CLIs: show 2-3 example invocations with realistic arguments and expected output
- For libraries: show import + basic usage in 5-15 lines of code
- For APIs: show a curl or fetch example hitting the main endpoints
- For webapps: show how to start the dev server and what to expect
- For monorepos: show the main entry point for each key package
- Code blocks MUST have language hints (```js, ```python, ```rust, etc.)

**Configuration**
- Table format with columns: Variable | Description | Default | Required
- Populate from `.env.example` or detected config files
- Use `> [!NOTE]` GitHub alert for important configuration notes

**API Reference** (libraries only)
- Document main exports with function/method signatures
- Include parameter types and return types
- Use code blocks for signatures
- Keep it concise — link to full docs if they exist

**Architecture** (monorepos and complex projects)
- Tree or table showing package/module structure
- One-line description of each package
- Use `<details>` collapsible sections for per-package detail if there are more than 3 packages

**Development**
- List available scripts/commands (build, test, lint, format, etc.)
- Pull commands directly from the manifest — don't invent them
- Include a code block developers can copy-paste to get started contributing:
  ```
  git clone <repo>
  cd <project>
  <install deps>
  <run tests>
  ```

**Contributing**
- 2-3 sentences inviting contributions
- If CONTRIBUTING.md exists or will be created, link to it
- Mention the relevant workflow (fork → branch → PR)

**License**
- State the license name
- Link to the LICENSE file: `[MIT](LICENSE)` or similar
- If no license is detected, use `> [!NOTE]` to suggest the user add one

**Table of Contents**
- Markdown links to each H2 section
- Place after Description, before Highlights

**Roadmap**
- Only include if there's actual evidence of planned work
- Bullet list of planned features or improvements
- Use checkbox format: `- [ ] Feature name`

**Status** (phased projects only)
- Table format: `Phase | What it covers | Status`
- Status values: `Done`, `In progress`, `Not started`, or a short caveat in place of a bare status (e.g. "Done, unverified" for code that exists but has no evidence of ever having run — no passing test, no log, no deploy record)
- Verify every phase against the code, tests, and git history — never report a phase's status from the plan doc's own checkboxes alone
- If the plan doc's checkboxes disagree with what the code/tests actually show, trust the code and say so in one line rather than silently picking a side
- If work happened out of the plan's declared order (a later phase done before an earlier one), say so plainly instead of reordering the table to look sequential

**Limitations**
- Bullet list, one line each: what's missing or narrower than it looks, and the concrete reason (a real constraint you found — a missing field, an unimplemented fallback, an external dependency that was never wired up)
- Only include gaps you've verified by reading the code — don't speculate or hedge with maybes

### 4e: Assemble and Present

1. Assemble all sections in the order listed above
2. Ensure a blank line between every section
3. Run the Quality Checklist (Step 7) internally before presenting
4. Present the complete README to the user
5. Ask: "Does this look good? I can adjust any section, or we can move on to companion files."
6. After the user approves (or after making requested changes), write the file to `README.md`

## Step 5: Improve Mode

When a README already exists with real content, improve it in-place.

### 5a: Map Existing Sections

Parse the existing README's headings and map them to the section menu from Step 4b. Use fuzzy matching:
- "Installation" / "Setup" / "Quick Start" → Getting Started
- "API" / "Reference" / "Docs" → API Reference
- "Tech Stack" / "Built With" → could be part of Description or Architecture
- "Features" / "Key Features" / "Why X?" → Highlights
- "Running Tests" / "Testing" → part of Development
- "How to Contribute" → Contributing
- Headings that don't map to any standard section: note them as custom sections to preserve

### 5b: Score Each Section

For each section in the menu (Step 4b), assign a score:

- **Strong**: Well-written, accurate, complete. Leave it alone.
- **Adequate**: Correct but could be richer. Leave it alone (minor polish at most).
- **Weak**: Present but thin, vague, or missing key information. Enhance it.
- **Missing**: Not present at all. Generate it.

A Development/CI section that describes a lint, type-check, test, or build step is not automatically Strong or Adequate just because it's present and well-written — run the config drift check from Step 1 first. If it describes a step that's actually broken (tool invoked but never installed), score it Weak and correct it rather than leaving it alone.

### 5c: Present Gap Report

Before making any changes, show the user a gap report:

```
## README Gap Analysis

| Section | Status | Notes |
|---------|--------|-------|
| Description | Weak | Only one sentence, doesn't explain why or how |
| Getting Started | Missing | No installation instructions |
| Usage | Missing | No code examples |
| ... | ... | ... |

**Plan**: I'll generate content for Missing sections and strengthen Weak ones.
Strong and Adequate sections will be preserved as-is.
```

Wait for user approval before proceeding.

If `dry-run` was specified, stop here after presenting the gap report. Do NOT generate content or offer companion files.

### 5d: Generate Improvements

- For **Missing** sections: generate fresh content following the same rules as Create Mode (Step 4c)
- For **Weak** sections: keep the existing text as a starting point, then expand and improve it. Do not rewrite from scratch.
- For **Strong/Adequate** sections: leave them unchanged
- For **custom sections** (not in the standard menu): preserve them in their original position

### 5e: Voice Preservation

When generating or enhancing content for improve mode:
- Read the existing README carefully and note: sentence length, vocabulary level, use of humor, formality, use of "we" vs. "you" vs. passive voice
- Match these patterns in new content
- If the existing README is casual, be casual. If it's terse, be terse. If it's detailed, be detailed.
- The reader should not be able to tell which sections are original and which are added

If the `tone` argument was provided, use that instead of matching the existing voice.

### 5f: Assemble and Present

1. Produce the complete improved README (not a diff — the full file)
2. Run the Quality Checklist (Step 7) internally
3. Present to the user, noting which sections were added or enhanced
4. After approval, write to `README.md`

## Step 6: Companion Files

After the README is finalized (create or improve), offer companion files. Read `references/companion-files.md` (in this skill's folder) for the full scan (6a), offer (6b), and generation (6c) procedure — including the help-mechanism (`--help`/`:help` + `help.md`) companion file.

## Step 7: Quality Checklist

Run this checklist internally before presenting any generated content. Do not show it to the user — just verify compliance.

- [ ] Title matches project name from manifest
- [ ] Description does not start with "This project is a..." or repeat the title
- [ ] No buzzwords: "blazing fast", "cutting-edge", "seamless", "robust", "powerful", "elegant"
- [ ] No placeholder text: "[describe X]", "TODO", "coming soon", "TBD"
- [ ] No emoji in section headers
- [ ] All code blocks have language hints
- [ ] All commands are pulled from the actual manifest (not invented)
- [ ] Relative links used for in-repo files
- [ ] Blank line between every section
- [ ] Getting Started commands are copy-paste-ready
- [ ] Configuration table has all 4 columns if present
- [ ] Only relevant sections included (no API Reference for CLIs, no Architecture for simple projects)
- [ ] Length is proportional to project complexity
- [ ] Professional but approachable tone (or matches existing tone in improve mode)
- [ ] No passive voice in key statements
- [ ] Content is factually correct based on the codebase analysis
- [ ] If a Status section is included, every phase's status was checked against code/tests/git history — not copied from the plan doc's own checkboxes
- [ ] If CI config is described anywhere, its invoked tools were checked against the dependency manifest (config drift check) before calling the pipeline working
