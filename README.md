# tech-trends

mkontani's daily tech trend curation site, published via GitHub Pages.

**Live site:** <https://mkontani.github.io/tech-trends/>

## Overview

Daily curated tech articles and trends collected from:

- Hatena Bookmark (IT / Programming / AI / Security / Engineer categories)
- Hacker News
- Reddit (13 subreddits: security, AI, core tech, OSS/indie dev, crypto)
- Feedly OPML subscriptions (44 feeds across Devops, Hacker, News categories)

Each entry is tagged and annotated with a personal interest rating based on
[mkontani's interest profile](./mkontani-interests.md).

## Repository Structure

```
.
├── content/
│   ├── posts/
│   │   └── daily/          # Daily trend reports (YYYYMMDD-trend.md)
│   └── _index.md
├── static/                 # Static assets
├── layouts/                # Hugo layout overrides
├── scripts/                # Shell automation scripts
├── .github/
│   └── workflows/
│       ├── deploy.yml      # Build and publish to GitHub Pages on push to main
│       └── lint.yml        # Markdown lint + shellcheck + Hugo build check on PR
├── hugo.toml               # Hugo site configuration
├── feedly-opml-*.opml      # Feedly OPML export (RSS feed subscriptions)
├── mkontani-interests.md   # Auto-generated interest profile (used for curation)
└── AGENTS.md               # Guidelines for agentic coding agents
```

## Local Development

**Prerequisites:** Hugo (extended), Go

```bash
# Install Hugo (macOS)
brew install hugo

# Start local dev server (includes draft posts)
hugo server -D

# Build for production
hugo --minify
```

## Lint and Validation

```bash
# Install tools
npm install -g markdownlint-cli2
brew install lychee shellcheck

# Full validation pass (run before every commit)
markdownlint-cli2 "**/*.md" && \
  shellcheck scripts/*.sh && \
  hugo --minify

# Lint a single post
markdownlint-cli2 content/posts/daily/20260228-trend.md
```

## Adding a Daily Report

1. Create `content/posts/daily/YYYYMMDD-trend.md` with Hugo front matter.
2. Collect entries from configured sources.
3. Cross-reference against `mkontani-interests.md` for interest ratings (★★★ / ★★ / ★).
4. Run lint and verify with `hugo server -D`.
5. Commit: `content(posts): add YYYY-MM-DD trend report`.

## Updating Feedly OPML

The Feedly OPML file (`feedly-opml-*.opml`) contains RSS feed subscriptions
used by the daily trend collection skill. When you add, remove, or change
subscriptions in Feedly, update the local file:

1. Open <https://feedly.com/i/opml> in a browser (login required).
2. Export (download) the OPML file.
3. Replace the existing `feedly-opml-*.opml` file in the repository root with
   the downloaded file.
4. Commit: `chore(feeds): update Feedly OPML export`.

## CI/CD

| Workflow | Trigger | Action |
|---|---|---|
| `deploy.yml` | Push to `main` | Build Hugo site and deploy to GitHub Pages |
| `lint.yml` | PR or push to `main` | Run markdownlint-cli2, shellcheck, Hugo build check |

## Theme

Uses [PaperMod](https://github.com/adityatelange/hugo-PaperMod) via Hugo module.
