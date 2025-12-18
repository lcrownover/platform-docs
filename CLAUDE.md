# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a mkdocs-material documentation site providing technical training modules for IT staff at University of Oregon. The current focus is platform-engineering topics: Puppet 101 → Git 101 → Puppet 201.

## Build & Preview Commands

```bash
# Setup (one time)
uv venv .venv
source .venv/bin/activate  # macOS/Linux
uv pip sync requirements.txt

# Development
uv run mkdocs serve                  # live preview at http://127.0.0.1:8000/
uv run mkdocs build --strict         # production build (run before PRs)
```

## Content Structure

- Pages live in `docs/` under module folders: `docs/puppet-101/`, `docs/git-101/`, `docs/puppet-201/`
- Each module has `index.md` (landing page) and a `prerequisites.md`
- Navigation is defined in `mkdocs.yml`—keep modules ordered logically (Puppet 101 → Git 101 → Puppet 201)
- Shared assets go in `docs/assets/`

## Writing Style

**Audience**: IT professionals at a university. They're technical but may be new to the specific tool. Don't over-explain fundamentals, but do explain *why* things work the way they do.

**Tone**: Encouraging but direct. Treat readers as capable colleagues learning a new skill.

**Structure**:
- One H1 per page (the title)
- Task-first sections: "Do X", then "Why it matters"
- Fenced code blocks with language hints (`bash`, `yaml`, `powershell`)
- Use mkdocs-material features: admonitions (`!!! note`, `!!! warning`), tabs for OS-specific commands, callouts

**Content quality**:
- Long-form explanations are welcome when they add useful context, rationale, or real-world examples
- Include terminal examples that readers can copy and run
- Explain tradeoffs and edge cases that working IT staff actually encounter
- Avoid filler text and generic advice—be specific and practical

**Naming**:
- Kebab-case filenames: `getting-started.md`
- Title-case headings
- Imperative commit messages: `Add Puppet 101 setup page`

## Hands Off

Do not read, edit, or modify `docs/ai-disclaimer.md`. That page is intentionally human-written.

## Before Committing

Always run `uv run mkdocs build --strict` before PRs—it fails on broken links and warnings.
