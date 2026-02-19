# CI/CD Pipelines

By the end of this section, you'll understand how CI/CD pipelines automate testing and deployment when you push code, and how to set up basic GitHub Actions workflows.

## What Is CI/CD?

**Continuous Integration (CI)** means automatically running checks every time someone pushes code or opens a pull request. Instead of waiting until merge day to discover broken tests or style violations, you catch them immediately.

**Continuous Delivery/Deployment (CD)** takes it further: once code passes all checks, it's automatically deployed (or staged for deployment) without manual intervention.

Together, CI/CD turns your Git workflow into a pipeline: push code, run checks, deploy if everything passes.

## Why CI/CD Matters

Without CI/CD, quality depends on everyone remembering to run tests, lint their code, and follow the process before pushing. That works when it's just you. It falls apart with a team.

CI/CD gives you:

- **Immediate feedback.** You know within minutes if your change broke something.
- **Consistent standards.** Every PR gets the same checks, regardless of who submitted it. No more "it works on my machine."
- **Confidence to merge.** If the pipeline passes, the change meets baseline quality. Reviewers can focus on design and logic, not whether the tests pass.
- **Faster iteration.** Automated deployment means changes reach production sooner and in smaller increments.

## GitHub Actions

GitHub Actions is GitHub's built-in CI/CD platform. It runs workflows in response to events (pushes, pull requests, schedules, etc.) using YAML configuration files stored in your repository.

### Where workflows live

Workflows are YAML files in the `.github/workflows/` directory of your repository:

```
my-repo/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── src/
└── README.md
```

GitHub automatically discovers and runs any workflow files in this directory.

### A basic workflow

Here's a minimal workflow that runs whenever you push to `main` or open a pull request:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run linter
        run: echo "Run your lint command here"

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: echo "Run your test command here"
```

Let's break this down:

| Section | What it does |
|---------|--------------|
| `name` | A label for the workflow (appears in the GitHub UI) |
| `on` | The events that trigger the workflow (push, pull_request, schedule, etc.) |
| `jobs` | The work to perform, each job runs in a fresh environment |
| `runs-on` | The operating system for the job (usually `ubuntu-latest`) |
| `steps` | The sequence of commands or actions to run |
| `uses` | References a reusable action (e.g., `actions/checkout@v4` clones your repo) |
| `run` | Executes a shell command |

### Triggers

Workflows run in response to events. Common triggers:

| Trigger | When it runs |
|---------|--------------|
| `push` | When commits are pushed to the specified branches |
| `pull_request` | When a PR is opened, updated, or reopened against the specified branches |
| `schedule` | On a cron schedule (e.g., nightly builds) |
| `workflow_dispatch` | Manually triggered from the GitHub UI |

### Real-world example: puppet-lint

If your team writes Puppet code and stores it in Git, you can run `puppet-lint` on every push and PR:

```yaml
# .github/workflows/puppet-lint.yml
name: Puppet Lint

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.3'

      - name: Install puppet-lint
        run: gem install puppet-lint

      - name: Run puppet-lint
        run: puppet-lint .
```

This catches style issues and errors before code is merged, instead of relying on everyone to run `puppet-lint` locally.

### Real-world example: mkdocs build

For a documentation site built with mkdocs (like this one), you can verify the build passes on every PR:

```yaml
# .github/workflows/docs.yml
name: Docs Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Build docs
        run: mkdocs build --strict
```

The `--strict` flag fails the build on warnings like broken links, so you catch documentation issues before they're merged.

## How CI/CD Fits into Your Workflow

CI/CD extends the Git workflow you already know:

1. Create a branch
2. Make changes and commit
3. Push and open a pull request
4. **CI runs automatically** (lint, test, build)
5. Fix any failures, push again
6. Reviewers approve (knowing CI passed)
7. Merge to `main`
8. **CD deploys automatically** (if configured)

The pipeline becomes a gatekeeper. Most teams configure their repository so PRs can't be merged unless the pipeline passes. This is called a **branch protection rule** on GitHub.

## Branch Protection Rules

Branch protection rules prevent direct pushes to important branches and require checks to pass before merging.

Common settings for `main`:

- **Require pull request reviews** -- at least one approval before merging
- **Require status checks to pass** -- CI must pass before the merge button is enabled
- **Require branches to be up to date** -- the PR branch must be current with `main`
- **Do not allow bypassing the above settings** -- applies rules to everyone, including admins

These rules enforce the workflow through tooling rather than relying on people to follow the process.

## Exercises

1. **Read a workflow file.** Find an open-source project on GitHub that uses GitHub Actions. Navigate to `.github/workflows/` and read through a workflow file. Identify the trigger, jobs, and steps.

2. **Create a basic workflow.** In a test repository, create `.github/workflows/ci.yml` with a simple workflow that runs `echo "Hello from CI"` on push. Push it and watch it run in the Actions tab.

3. **Add a real check.** Extend your workflow to do something useful: run a linter, validate YAML files, or build documentation. Make it fail on purpose to see what a failed pipeline looks like.

4. **Explore branch protection.** In your test repository's settings, set up a branch protection rule for `main` that requires the CI check to pass before merging.
