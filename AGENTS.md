# AGENTS.md

Guidelines for agentic coding agents working in this repository.

---

## Agent Response Style

- **Conclusion first.** Lead with the answer or result, then explain if needed.
- No greetings, no preamble, no phrases like "Certainly!", "Great question!", or "Sure!".
- No progress narration ("Now I will...", "Next, I'll..."). Act, then report the outcome.
- No emojis.
- Be direct. If something is wrong or a better approach exists, say so plainly without hedging.
- Keep responses concise. Omit filler sentences that add no information.

---

## Tool Priority

Use the most specific tool available for each task. Fall back to general-purpose
tools only when the preferred tool fails or is unavailable.

| Priority | Tool type | When to use |
|---|---|---|
| 1 | Custom skills (`skill` tool) | Always attempt first for any supported task |
| 2 | Named specialist tools | File read/write/edit, glob, grep, bash, etc. |
| 3 | General-purpose tools | `WebFetch`, `WebSearch`, open-ended agents — only when higher-priority tools cannot complete the task |

Rules:

- Prefer `Read`, `Edit`, `Write`, `Glob`, `Grep` over shelling out to `cat`,
  `find`, `grep`, `sed`, etc.
- Prefer `Bash` for system commands; do not simulate shell behavior in prose.
- Invoke a custom skill before reaching for a generic web search or broad agent.
- If a custom skill returns an error or no result, fall back to the next tier
  and note why the preferred tool was skipped.
- Never use a lower-priority tool in parallel with a higher-priority one for the
  same subtask.

---

## Project Overview

This is a tech-trends curation repository. Content is written in Markdown and
published as a static site via GitHub Pages. Automation helpers are written as
shell scripts. The static site generator is **Hugo**.

---

## Repository Layout

```
.
├── content/          # Markdown articles and trend entries
│   ├── posts/        # Individual curated articles (YYYY-MM-DD-slug.md)
│   └── tags/         # Auto-generated tag pages (do not edit manually)
├── static/           # Static assets (images, favicons, etc.)
├── themes/           # Hugo theme (git submodule or copied)
├── layouts/          # Custom Hugo layout overrides
├── scripts/          # Shell automation scripts
├── .github/
│   └── workflows/    # CI/CD — GitHub Actions
├── hugo.toml         # (or config.yaml / config.toml) Site configuration
└── AGENTS.md         # This file
```

---

## Build Commands

| Task | Command |
|---|---|
| Install Hugo (macOS) | `brew install hugo` |
| Start local dev server | `hugo server -D` |
| Build for production | `hugo --minify` |
| Build to custom output dir | `hugo --minify -d public` |
| Check Hugo version | `hugo version` |

---

## Lint Commands

### Markdown

```bash
# Install markdownlint-cli2
npm install -g markdownlint-cli2

# Lint all Markdown files
markdownlint-cli2 "**/*.md"

# Lint a single file
markdownlint-cli2 content/posts/2026-02-28-my-article.md

# Auto-fix fixable issues
markdownlint-cli2 --fix "**/*.md"
```

Config lives in `.markdownlint.yaml` (or `.markdownlint.jsonc`).

### Link checking

```bash
# Install lychee (macOS)
brew install lychee

# Check all links in Markdown files
lychee --verbose "content/**/*.md"

# Check a single file
lychee content/posts/2026-02-28-my-article.md
```

### Shell scripts

```bash
# Install shellcheck (macOS)
brew install shellcheck

# Check all shell scripts
shellcheck scripts/*.sh

# Check a single script
shellcheck scripts/fetch-feeds.sh
```

---

## Test / Validation Commands

There is no test suite in the traditional sense. "Tests" are lint and build
checks:

```bash
# Full validation pass (run before every commit)
markdownlint-cli2 "**/*.md" && \
  shellcheck scripts/*.sh && \
  hugo --minify

# Quick single-file check (fastest feedback loop)
markdownlint-cli2 content/posts/<filename>.md
shellcheck scripts/<scriptname>.sh
```

---

## Content Style Guidelines

### Front Matter (Hugo)

Every post **must** include the following YAML front matter:

```yaml
---
title: "Descriptive Title of the Article or Trend"
date: 2026-02-28          # ISO 8601, publication date
lastmod: 2026-02-28       # Update this when editing existing posts
draft: false
description: "One or two sentence summary of why this is interesting."
tags:
  - ai
  - devtools
  - observability
source_url: "https://example.com/original-article"
---
```

Optional fields: `author`, `image`, `weight`.

### Markdown Formatting

- Use ATX headings (`#`, `##`, `###`) — never Setext (`===`, `---`).
- One blank line before and after every heading, code block, and list.
- Fenced code blocks must specify a language: ` ```bash `, ` ```yaml `, etc.
- Maximum line length: **120 characters** (prose lines may be longer; code
  blocks must not exceed 120).
- Use `**bold**` for emphasis, not `__bold__`.
- Use `-` for unordered lists, not `*` or `+`.
- External links: always use absolute URLs including scheme (`https://...`).
- Internal links: use Hugo ref shortcodes `{{< ref "posts/slug" >}}` rather
  than bare relative paths.

### File Naming

- Posts: `content/posts/YYYY-MM-DD-kebab-case-slug.md`
  - Example: `content/posts/2026-02-28-llm-context-windows.md`
- Static assets: `static/images/YYYY-MM-DD-description.png` (kebab-case)
- Shell scripts: `scripts/snake_case_name.sh`
- All filenames: lowercase, no spaces, no special characters except `-` and `_`.

### Tag Taxonomy

Use existing tags before creating new ones. Preferred tags:

`ai`, `devtools`, `security`, `cloud`, `observability`, `frontend`,
`backend`, `databases`, `opensource`, `career`, `research`, `tools`,
`typescript`, `python`, `go`, `rust`, `kubernetes`, `llm`

Tags must be lowercase, single-word or hyphenated. No CamelCase tags.

---

## Shell Script Conventions

Every shell script must begin with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

- Use `snake_case` for variable names and function names.
- Quote all variable expansions: `"${variable}"`, not `$variable`.
- Use `[[ ... ]]` for conditionals, not `[ ... ]`.
- Use `$(...)` for command substitution, not backticks.
- Print errors to stderr: `echo "ERROR: message" >&2`
- Exit with a non-zero code on failure: `exit 1`
- Add a brief comment at the top of each script explaining its purpose.

Example:

```bash
#!/usr/bin/env bash
set -euo pipefail
# fetch-feeds.sh — Download RSS feeds and append new entries to content/posts/

readonly OUTPUT_DIR="content/posts"

main() {
  echo "Fetching feeds..."
  # ...
}

main "$@"
```

---

## Git Conventions

### Branch Naming

```
content/<short-description>    # New article or trend entry
feat/<short-description>       # New site feature or script
fix/<short-description>        # Bug fix in scripts or config
chore/<short-description>      # Dependency bumps, config tweaks
```

### Commit Messages

Follow the Conventional Commits format:

```
<type>(<scope>): <short imperative summary>

[optional body]
```

Types: `content`, `feat`, `fix`, `chore`, `docs`, `ci`, `style`, `refactor`

Examples:
```
content(posts): add article on LLM context window advances
feat(scripts): add RSS feed fetcher script
fix(config): correct base URL for GitHub Pages deployment
chore(deps): bump markdownlint-cli2 to v0.13
```

- Subject line: max 72 characters, imperative mood, no trailing period.
- Reference issues with `Closes #N` or `Related #N` in the body.

---

## GitHub Actions / CI

Workflows live in `.github/workflows/`. Expected workflows:

- **`deploy.yml`** — Build Hugo site and publish to `gh-pages` branch on push
  to `main`.
- **`lint.yml`** — Run `markdownlint-cli2` and `shellcheck` on pull requests.

The site is published from the `gh-pages` branch (or `docs/` folder if
configured in repository settings under Pages).

---

## Adding a New Article (Checklist)

1. Create `content/posts/YYYY-MM-DD-kebab-slug.md` with complete front matter.
2. Write a brief commentary section explaining why the article is worth reading.
3. Include the original `source_url` in front matter.
4. Assign 2–5 tags from the approved taxonomy.
5. Run `markdownlint-cli2 content/posts/<file>.md` — fix any warnings.
6. Run `hugo server -D` and verify the post renders correctly.
7. Commit with `content(posts): add article on <topic>`.
