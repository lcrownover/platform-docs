# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a mkdocs-material documentation site providing technical training guides for IT staff at University of Oregon. The current focus is platform-engineering topics: Puppet 101 → Git 101 → Puppet 102 → Git 102.

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

- Learning module folders live in `docs/learn/` (e.g., `docs/learn/git/`, `docs/learn/puppet/`)
- Each module has:
  - `00-index.md` (landing page with overview and lesson list)
  - `00-prerequisites.md` (required/recommended tools in table format)
  - Numbered lessons: `01-local-foundations.md`, `02-understanding-history.md`, etc.
- Navigation is defined in `mkdocs.yml` (keep modules ordered logically)
- Shared assets go in `docs/assets/`

## Writing Style

**Audience**: IT professionals at a university. They're technical but may be new to the specific tool. Don't over-explain fundamentals, but do explain _why_ things work the way they do.

**Tone**: Second person ("you", "your"), encouraging but direct. Treat readers as capable colleagues. Use "Let's" for guided activities. Acknowledge real-world scenarios ("When something breaks at 2 AM...").

### Page Structure

**Learn index page (`00-index.md`)**:

- H1: Module title only (e.g., "Git 101")
- Opening paragraph: Relatable problem statement, then how this tool solves it
- "Why You Should Care" section with bullet points
- "Lessons" section: numbered list with bold lesson names and brief descriptions
- "Tips for Success" section with practical advice
- Optional notes about the tool's context or history

**Lesson pages**:

- H1: Lesson title only
- Opening line: "By the end of this section, you'll..." (sets expectations)
- H2 for major sections, H3 for subsections (avoid H4+)
- End with "## Exercises" section (numbered list, bold task names, imperative verbs)

### Content Patterns

**Code examples**:

```markdown
Run the command:

\`\`\`bash
git status
\`\`\`

You'll see:

\`\`\`
On branch main
nothing to commit
\`\`\`

This tells you three things: ...
```

Show command first, expected output in a separate block, then explain what happened.

**Comparisons and reference info**: Use tables, not bullet lists.

| Instead of... | Write... |
|--------------|----------|
| `Update config` | `Increase nginx worker connections to 4096` |

**Admonitions**: Use sparingly but effectively:

- `!!! tip` for helpful shortcuts or best practices
- `!!! note` for context or clarifications
- `!!! warning` for common mistakes or gotchas
- `!!! danger` for destructive commands or serious risks

**Troubleshooting sections**: Use quoted H3 headers for problem descriptions:

```markdown
### "I committed to the wrong branch"

**Solution:** Move the commit to the right branch.
```

**Cross-references**: Use relative paths with descriptive link text:

```markdown
See [Branching and Merging](03-branching-and-merging.md#handling-merge-conflicts) for details.
```

### Formatting

- **Bold** for key terms and emphasis
- `inline code` for commands, flags, filenames, and config values
- Parentheses for asides (not emdashes)
- Numbered lists for sequential steps or ordered lessons
- Bullet lists for conceptual points or unordered items

### Content Quality

- Long-form explanations are welcome when they add useful context, rationale, or real-world examples
- Include terminal examples that readers can copy and run
- Explain tradeoffs and edge cases that working IT staff actually encounter
- Avoid filler text and generic advice (be specific and practical)
- Use concrete analogies to explain abstract concepts

### Naming

- Kebab-case filenames: `01-local-foundations.md`
- Title-case headings: "Understanding History"
- Imperative commit messages: `Add Git recovery section`

## Lab Guides

Lab guides live in `docs/lab-guides/` and serve as cliff-notes companions to the learning modules. They are **not** full lessons; they're concise checklists for an in-person lab session.

**Format**:

- H1: Lab title (e.g., "Puppet 101 Lab")
- `## Prerequisites` section: table of required tools linking to the corresponding tools docs (same format as learning module prerequisites)
- `## Setup` section: connection info and environment confirmation
- Numbered `## Part N: Title` sections: each covers one focused activity
- Each part: brief numbered steps with bold action names, code blocks for commands/snippets, and "apply and verify" checkpoints
- `## Recap` section: short bullet list of key takeaways
- Keep prose minimal; let the code blocks do the heavy lifting

## Lab Participant Conventions

Participants are numbered 1-40. All derived values use zero-padded two-digit numbers (e.g., participant 5 → `05`).

| Property | Pattern | Example (participant 5) |
|----------|---------|------------------------|
| Username | `participantXX` | `participant05` |
| Password | `UOISLabNumber@XX` | `UOISLabNumber@05` |
| User repository | `participantXX/myrepo` | `participant05/myrepo` |
| Group repository | `groupN/ourrepo` | `group1/ourrepo` |

**Groups**: participants 1-5 = group1, 6-10 = group2, 11-15 = group3, ..., 36-40 = group8.

Use these conventions consistently across all lab guides. Not every lab uses every property (e.g., Puppet 101 doesn't use repositories).

## Puppet Content Guidelines

When writing Puppet documentation, follow these principles:

- **Prefer plain Puppet code over Forge modules.** Only recommend Forge modules when they provide significant value. Keep things simple and understandable.
- **Teach the "Package → File → Service" pattern.** This is the mental model: install packages as needed, write configuration files, and notify services when those files change. Most Puppet work follows this pattern.

## Before Committing

Always run `uv run mkdocs build --strict` before PRs (it fails on broken links and warnings).
