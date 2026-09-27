# make-readme

Works with: Claude Code · Codex · Antigravity · Gemini CLI

A Claude Code skill that generates or improves README.md files for any GitHub project. Analyzes the codebase — package manifests, directory structure, CI config, existing docs — and produces a well-structured, GitHub-flavored markdown README tailored to the detected project type.

Works in two modes: **create mode** (no README exists) generates a complete file from scratch; **improve mode** (README exists) performs a gap analysis and enhances the existing file in-place, preserving the author's voice. After the README is done, offers to create companion files like CONTRIBUTING.md, CODE_OF_CONDUCT.md, LICENSE, issue/PR templates, and — for projects that are themselves an agent skill or plugin — a `--help`/`:help` mechanism backed by `help.md`.

## Table of Contents

- [Highlights](#highlights)
- [Installation](#installation)
- [Compatibility](#compatibility)
- [References](#references)
- [Usage](#usage)
- [Supported Project Types](#supported-project-types)
- [Who It's For](#who-its-for)
- [Project Structure](#project-structure)
- [Development](#development)
- [Contributing](#contributing)
- [Changelog](#changelog)
- [Related Resources](#related-resources)
- [License](#license)

## Highlights

- **Two modes** — Create from scratch or improve an existing README with gap analysis
- **Project-aware** — Detects project type (library, CLI, webapp, API, monorepo) and tailors sections accordingly
- **Voice preservation** — Improve mode matches the tone and style of existing content
- **Companion files** — Offers to generate CONTRIBUTING.md, CODE_OF_CONDUCT.md, LICENSE, CHANGELOG.md, issue/PR templates
- **Help mechanism for skills/plugins** — For projects with a `SKILL.md` or plugin manifest, offers to add a `--help`/`:help` convention (`help.md` + a short pointer section in the manifest), extracting from any existing inline help text rather than duplicating it
- **No dependencies** — Pure prompt; install by copying a single file

## Installation

### From the mackayi marketplace (recommended)

```
/plugin marketplace add keithmackay/mackayi
/plugin install make-readme@mackayi
```

### Claude Code

```bash
cp -r /path/to/make-readme/skill/ ~/.claude/skills/make-readme/
```

Or symlink:
```bash
ln -s /path/to/make-readme/skill/ ~/.claude/skills/make-readme
```

Then invoke with: `/make-readme`

### Codex

Place the plugin directory where Codex can find it, then add an entry to your marketplace:

**`~/.agents/plugins/marketplace.json`** (create if absent):
```json
{
  "name": "personal",
  "interface": { "displayName": "Personal Plugins" },
  "plugins": [
    {
      "name": "make-readme",
      "source": { "source": "local", "path": "/path/to/make-readme/skill/" },
      "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
      "category": "Productivity"
    }
  ]
}
```

### Antigravity

**Global install** (all workspaces):
```bash
cp -r /path/to/make-readme/skill/ ~/.gemini/antigravity/skills/make-readme/
```

**Workspace install** (current project only):
```bash
cp -r /path/to/make-readme/skill/ .agents/skills/make-readme/
```

The root `SKILL.md` has no Claude Code-specific metadata, so it is used as-is — no separate Antigravity variant is needed.

Skills are auto-discovered. You can also mention the skill by name to force activation.

### Gemini CLI

Gemini CLI installs extensions directly from GitHub:

```bash
gemini extensions install https://github.com/keithmackay/make-readme
```

To update:
```bash
gemini extensions update make-readme
```

The skill is auto-discovered from `GEMINI.md` after installation.

## Compatibility

| Feature | Claude Code | Codex | Antigravity | Gemini CLI |
|---------|:-----------:|:-----:|:-----------:|:----------:|
| Core skill | ✅ | ✅ | ✅ | ✅ |

No Claude Code-specific frontmatter (`metadata`, `retrieval`, `tags`), sub-documents, or subagent dispatch is used by this skill, so there are no platform gaps to document — it ports cleanly to all four platforms.

Legend: ✅ Supported · ❌ Not supported

## References

- **Claude Code Skills:** https://code.claude.com/docs/en/skills
- **Claude Code Complete Guide (PDF):** https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf
- **Codex Plugins:** https://developers.openai.com/codex/plugins/build
- **Antigravity Skills:** https://antigravity.google/docs/skills
- **Gemini CLI Extensions:** https://github.com/google-gemini/gemini-cli/blob/main/docs/extension.md
- **Agent Skills open standard:** https://agentskills.io/home

## Usage

In any project directory, open a Claude Code session and run:

```
/make-readme
```

The skill will:

1. Analyze the codebase (package manifests, directory structure, CI config, existing docs)
2. Detect the project type (library, CLI, webapp, API, monorepo)
3. Ask whether you want badges
4. Generate a complete README (create mode) or improve the existing one (improve mode)
5. Offer to create companion files (CONTRIBUTING.md, CODE_OF_CONDUCT.md, LICENSE, a `--help`/`:help` mechanism for skills/plugins, etc.)

### Optional Arguments

Pass arguments after `/make-readme` to override auto-detection:

```
/make-readme audience=end-users tone=casual
/make-readme type=library tone=formal
/make-readme dry-run
```

| Argument | Values | Default |
|----------|--------|---------|
| `audience` | `developers`, `end-users`, `data-scientists`, `mixed` | Auto-detect |
| `type` | `library`, `cli`, `webapp`, `api`, `monorepo` | Auto-detect |
| `tone` | `formal`, `casual`, `minimal`, `playful` | `professional` (create) / `match-existing` (improve) |
| `dry-run` | (flag, no value) | Off |

`dry-run` shows the detected project type, planned section list, and gap analysis (for improve mode) without generating any content.

## Supported Project Types

- **Node.js** — package.json (npm, yarn, pnpm)
- **Python** — pyproject.toml, setup.py, setup.cfg
- **Rust** — Cargo.toml
- **Go** — go.mod
- **Java** — pom.xml, build.gradle
- **Ruby** — Gemfile, .gemspec
- **PHP** — composer.json
- **Monorepos** — Turborepo, Lerna, pnpm workspaces, npm workspaces

## Who It's For

**Good fit if you:**
- Want a README generated or improved from an actual codebase analysis (manifests, CI config, directory structure) rather than filled-in boilerplate
- Use Claude Code, Codex, Antigravity, or Gemini CLI and want a skill you install by copying one file
- Want an existing README improved in place, with your original voice and any custom sections preserved

**Not a fit if you:**
- Want a hosted/SaaS README generator with a web UI — this only runs inside a supported agent CLI, there's no separate app
- Need automated enforcement (a CI check that fails a build on a stale README) — this is an interactive skill invoked on demand, not a linter

## Project Structure

```
skill/SKILL.md               The skill — all prompt logic lives here
skill/related-resources.md   Optional user-authored blurb inserted into generated READMEs
skill/skills/make-readme/    Codex/Gemini CLI copy of the skill content
skill/.codex-plugin/         Codex plugin manifest
skill/gemini-extension.json  Gemini CLI extension manifest
skill/GEMINI.md              Gemini CLI context file (includes skill content)
tests/fixtures/              5 minimal fake projects for testing
tests/checklists/            Acceptance criteria (create, improve, companion files)
tests/snapshots/             Reference outputs from fixture runs
docs/TESTING_GUIDELINES.md   Prompt-testing strategy
```

## Development

There's no build step or automated test suite — the deliverable is a single prompt file. Testing is manual and fixture-based.

**Run the skill against a fixture:**

```bash
cd tests/fixtures/node-express-api
# In a Claude Code session:
/make-readme
```

**Validate output against checklists:**

Review the generated README against the appropriate checklist in `tests/checklists/`:

| Checklist | When to use |
|-----------|-------------|
| `create-mode-checklist.md` | After running on a fixture with no README |
| `improve-mode-checklist.md` | After running on `existing-readme-with-gaps` |
| `companion-files-checklist.md` | After generating companion files |

**Compare against snapshots:**

Reference outputs for each fixture are saved in `tests/snapshots/`. After changing `skill/SKILL.md`, re-run against at least 2 fixtures and compare with the snapshots.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and guidelines.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

## Related Resources

If you find practical AI tooling like this useful, I talk about and release them regularly in my free newsletter [Leading in the AI Era](https://tlcmentor.substack.com). 2026 subscribers will get newsletter free forever.

## License

[MIT](LICENSE) © 2026 Keith MacKay
