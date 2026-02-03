# Platform Engineering Docs

Platform Engineering docs for IT staff, built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## What's Here

Learning modules covering platform engineering fundamentals:

- **Puppet**: Configuration management
- **Git**: Version control essentials

Each module includes prerequisites, hands-on guides, and practical exercises.

## Local Development

### Prerequisites

- Python 3.10+
- [uv](https://docs.astral.sh/uv/) (Python package manager)

### Setup

```bash
# Clone the repository
git clone <repo-url>
cd platform-docs

# Install dependencies with uv
uv sync
```

### Run Locally

```bash
uv run mkdocs serve
```

Open http://127.0.0.1:8000 in your browser. The site auto-reloads when you edit files.

### Build for Production

```bash
uv run mkdocs build --strict
```

This generates static files in `site/` and fails on broken links or warnings. Always run this before opening a PR.

## Deployment

The site is deployed to S3 and served via CloudFront.

### GitHub Actions Workflow

<!-- TODO: Document the GitHub Actions workflow -->

The deployment pipeline:

1. Triggered on push to `main`
2. Builds the site with `mkdocs build --strict`
3. Syncs to S3 bucket
4. Invalidates CloudFront cache

### S3 Configuration

<!-- TODO: Document S3 bucket setup -->

- Bucket name: `[TBD]`
- Static website hosting enabled
- Public access configured via bucket policy

### CloudFront Configuration

<!-- TODO: Document CloudFront distribution setup -->

- Distribution ID: `[TBD]`
- Custom domain: `[TBD]`
- SSL certificate via ACM

### Required Secrets

<!-- TODO: Document required GitHub secrets -->

The following secrets must be configured in GitHub:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `S3_BUCKET`
- `CLOUDFRONT_DISTRIBUTION_ID`

## Contributing

1. Create a branch for your changes
2. Run `uv run mkdocs serve` to preview locally
3. Run `uv run mkdocs build --strict` before committing
4. Open a pull request

See `CLAUDE.md` for detailed writing style guidelines.
