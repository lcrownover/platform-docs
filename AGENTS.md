# Repository Guidelines

Welcome to the platform-engineering training docs. These notes keep contributions consistent and friendly for learners moving through Puppet 101 → Git 101 → Puppet 201.

## Project Structure & Module Organization

- `mkdocs.yml` defines navigation; keep modules ordered: Puppet 101 (no Git yet) → Git 101 → Puppet 201.
- Place pages in `docs/` under module folders: `docs/puppet-101/`, `docs/git-101/`, `docs/puppet-201/`. Use `index.md` for section landing pages.
- Each module needs a “Prerequisites” page covering required tools for macOS and Windows plus install steps (e.g., Puppet agent, Git CLI, VS Code, terminal basics). Keep commands verified for both platforms.
- Design each lesson so an instructor-led group of 20–30 can finish within ~2 hours; self-paced should be faster.
- Store shared assets in `docs/assets/` (images, diagrams, downloadable code). Reference with relative paths.
- Keep pages focused and short; link to follow-up sections rather than duplicating content.

## Build, Preview, and Release Commands

- `uv venv .venv` — create a local virtualenv managed by `uv`.
- `source .venv/bin/activate` (macOS/Linux) or `.\.venv\Scripts\activate` (Windows PowerShell) — enter the env.
- `uv pip sync requirements.txt` — install mkdocs-material and plugins from the pinned list.
- `uv run mkdocs serve` — live-reload preview at `http://127.0.0.1:8000/`.
- `uv run mkdocs build --strict` — production build; fails on broken links or warnings (run before PRs/CI).
- `uv run mkdocs gh-deploy` — publish to GitHub Pages if enabled.

## Content Style & Naming Conventions

- Write in a concise, encouraging training tone; spell out prerequisites at the top of each page.
- Use kebab-case filenames (`getting-started.md`); title-case headings; keep one H1 per page.
- Prefer task-first sections: “Do X”, then “Why it matters”. Include terminal examples with fenced code blocks and language hints (` ```bash `, ` ```yaml `).
- Leverage mkdocs-material features (admonitions like `!!! note`, tabs, callouts) for clarity; keep screenshots lightweight and captioned.

## Testing & Review Guidelines

- Always run `mkdocs build --strict` after adding or reorganizing pages; fix warnings immediately.
- Manually click through `mkdocs serve` to verify navigation, cross-links, and code block rendering.
- When adding a module page, ensure the nav entry in `mkdocs.yml` matches its sequential spot.

## Commit & Pull Request Guidelines

- Use imperative commit messages: `Add Puppet 101 setup page`, `Clarify Git branching exercise`.
- Keep PRs scoped to one training slice (e.g., a lesson or exercise). Note which commands you tested.
- PR description should summarize intent, key changes, and screenshots of new/changed pages when visual.
- Link related issues/tickets and call out any reader prerequisites or environment files that changed.
