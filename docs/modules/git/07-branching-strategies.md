# Branching Strategies

By the end of this section, you'll understand three common approaches to organizing branches on a team, their tradeoffs, and how to recognize which one you're working with.

!!! tip "TL;DR: Start with GitHub Flow"
    If you're not sure which strategy to use, **GitHub Flow** is the safe default. Create a branch for each change, open a pull request, merge to `main` after review. It's simple, works for most teams, and you can always adopt something more complex later if needed.

## Why Strategies Matter

When you're working alone, you can branch however you want. On a team, everyone needs to follow the same conventions or chaos ensues. A branching strategy answers questions like:

- Where does new feature work happen?
- How do changes get to production?
- How do we handle urgent fixes?
- What branches exist and what are they for?

There's no single "right" strategy. The best choice depends on your team size, release cadence, and how you deploy.

## Git Flow

Git Flow is a structured model designed for projects with scheduled releases (like versioned software with v1.0, v2.0, etc.).

### The branches

| Branch | Purpose | Lifespan |
|--------|---------|----------|
| `main` | Production-ready code, tagged with version numbers | Permanent |
| `develop` | Integration branch for features, always deployable to staging | Permanent |
| `feature/*` | New features, branched from `develop` | Temporary |
| `release/*` | Preparing a release, branched from `develop` | Temporary |
| `hotfix/*` | Emergency fixes for production, branched from `main` | Temporary |

### How it works

```
main     ─────●─────────────────●─────────────●───────
              │                 ↑             ↑
              │           merge │       merge │
              │                 │             │
hotfix   ─────┼─────────────────┼─────●───────┘
              │                 │     │
develop  ─────●─────●─────●─────●─────●─────●─────────
              │     ↑     ↑     ↑           ↑
              │     │     │     │           │
feature  ─────┴─────┴─────┴─────┘           │
                                            │
release  ───────────────────────────────────┘
```

1. **Feature development**: Create `feature/add-login` from `develop`. Work on it. Merge back to `develop` when done.

2. **Release preparation**: When `develop` has enough features for a release, create `release/1.2.0` from `develop`. Only bug fixes go into the release branch. When ready, merge to both `main` (and tag it) and back to `develop`.

3. **Hotfixes**: Critical bug in production? Create `hotfix/fix-crash` from `main`, fix it, merge to both `main` (tag it) and `develop`.

### When to use Git Flow

Git Flow works well when:

- You ship versioned releases on a schedule (monthly, quarterly)
- You need to maintain multiple versions simultaneously
- You have a QA phase before releases
- You're building installable software (desktop apps, libraries, firmware)

### When to avoid Git Flow

Git Flow adds overhead that isn't worth it when:

- You deploy continuously (multiple times per day)
- You only maintain one production version
- Your team is small and moves fast
- You're building a web service with rolling deployments

!!! note "Git Flow's history"
    Git Flow was introduced in 2010 and became very popular. It solved real problems for its time. However, as continuous deployment became common, many teams found it too heavyweight. The creator himself has noted it's not ideal for web applications with continuous delivery.

## GitHub Flow

GitHub Flow is a simpler model designed for continuous deployment. If you deploy frequently and only maintain one production version, this is probably what you want.

### The branches

| Branch | Purpose | Lifespan |
|--------|---------|----------|
| `main` | Always deployable, represents production | Permanent |
| `feature-name` | Any new work, branched from `main` | Temporary |

That's it. Two types of branches.

### How it works

```
main     ─────●─────●─────●─────●─────●─────●─────────
              │     ↑     │     ↑     │     ↑
              │     │     │     │     │     │
feature  ─────┴─────┘     └─────┘     └─────┘
           (add-login)  (fix-bug)  (update-api)
```

1. **Create a branch** from `main` with a descriptive name
2. **Make commits** on your branch
3. **Open a pull request** when ready for review
4. **Discuss and review** the changes
5. **Deploy from the branch** to verify in production (optional but recommended)
6. **Merge to main** after approval

### The key principles

- **`main` is always deployable.** Never merge broken code.
- **Branch names describe the work.** No prefixes required, but use them if your team likes them.
- **Pull requests are mandatory.** All changes go through review.
- **Deploy frequently.** Ideally, merge and deploy multiple times per day.

### When to use GitHub Flow

GitHub Flow works well when:

- You deploy continuously (daily or more)
- You only have one production version
- You have good automated testing
- Your team does code review via pull requests
- You're building web services or SaaS products

### When to avoid GitHub Flow

GitHub Flow may not fit when:

- You need to maintain multiple release versions
- You have long QA cycles before releases
- You can't deploy frequently due to external constraints

## Trunk-Based Development

Trunk-Based Development (TBD) takes simplicity further: everyone commits to `main` (the "trunk") with very short-lived branches or no branches at all.

### The philosophy

The core idea: integration pain increases exponentially with branch lifetime. A branch that lives for a day is easy to merge. A branch that lives for a month is a nightmare. So keep branches as short as possible, or eliminate them entirely.

### How it works

**With short-lived branches:**

```
main     ─────●─────●─────●─────●─────●─────●─────────
              │     ↑     │     ↑     │     ↑
              └─────┘     └─────┘     └─────┘
            (1-2 days)  (few hours)  (1 day)
```

Branches live for hours or a day or two at most. Developers integrate to `main` at least daily.

**Without branches:**

```
main     ─────●─────●─────●─────●─────●─────●─────────
              │     │     │     │     │     │
            (dev1) (dev2) (dev1) (dev1) (dev2) (dev1)
```

Developers commit directly to `main` multiple times per day. This requires excellent testing and often feature flags.

### Feature flags

How do you work on a feature that takes a week if you're committing to `main` daily? Feature flags.

```python
if feature_enabled("new_checkout_flow"):
    show_new_checkout()
else:
    show_old_checkout()
```

You commit incomplete code behind a flag. The flag is off in production, so users don't see it. When the feature is complete and tested, you turn on the flag. If something breaks, turn it off instantly without a deploy.

### When to use Trunk-Based Development

TBD works well when:

- You have excellent automated testing
- You can deploy multiple times per day
- Your team is experienced with continuous integration
- You're willing to invest in feature flag infrastructure
- You want to minimize merge conflicts

### When to avoid Trunk-Based Development

TBD may not fit when:

- Your testing is unreliable or slow
- You can't deploy frequently
- Your team is less experienced with Git
- You need code review before changes hit `main` (though some TBD teams do post-commit review)

## Choosing a Strategy

Here's a decision framework:

### Start with these questions

1. **How often do you deploy?**
   - Weekly or less → Git Flow might fit
   - Daily → GitHub Flow
   - Multiple times daily → GitHub Flow or Trunk-Based

2. **How many versions do you maintain?**
   - Multiple concurrent versions → Git Flow
   - Just one (latest) → GitHub Flow or Trunk-Based

3. **What's your team's Git experience?**
   - Mixed or learning → GitHub Flow (simpler)
   - Experienced → Any, including Trunk-Based

4. **How good is your automated testing?**
   - Comprehensive and fast → Trunk-Based is viable
   - Gaps or slow → GitHub Flow with PR reviews as a safety net

### Summary comparison

| Factor | Git Flow | GitHub Flow | Trunk-Based |
|--------|----------|-------------|-------------|
| Complexity | High | Low | Low |
| Branch lifespan | Days to weeks | Hours to days | Hours |
| Release style | Scheduled versions | Continuous | Continuous |
| Best for | Versioned software | Web services | High-velocity teams |
| Merge conflicts | Common | Occasional | Rare |
| Requires | Release planning | PR reviews | Excellent CI/CD |

### What most teams should start with

If you're unsure, **start with GitHub Flow**. It's simple enough to learn quickly, structured enough to maintain quality through pull requests, and flexible enough to adapt as your needs change.

You can always add complexity later. It's much harder to simplify an overcomplicated workflow.

## Exercises

1. **Identify the strategy:** Look at a few open-source projects on GitHub. Examine their branches, pull requests, and tags. Can you tell which strategy they use?

2. **Evaluate your projects:** Think about a project you work on. Which strategy would fit best based on how often you deploy and how you release?

3. **Try GitHub Flow:** In your sandbox repository, practice the GitHub Flow cycle: create a branch, make commits, then imagine opening a PR (or actually open one if you push to a remote).

4. **Explore feature flags:** Research feature flag services (LaunchDarkly, Unleash, or simple environment variables). Consider how you'd use them to enable trunk-based development.
