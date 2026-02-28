---
name: tech-trend-daily
description: "Daily tech trend collection customized for mkontani"
---

# Daily Trend Collection

Collect trending articles from Hatena Bookmark IT, Hacker News, Reddit, and security
blogs, then save results to `content/posts/daily/YYYYMMDD-trend.md`.

## Steps

### 0. Load user profile

Read `mkontani-interests.md` to understand mkontani's technical interest areas.

### 1. Collect trend data

Fetch the latest trending content from the following sources:

**Japanese market (Hatena Bookmark IT)**

- https://b.hatena.ne.jp/hotentry/it
- https://b.hatena.ne.jp/hotentry/it/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0
- https://b.hatena.ne.jp/hotentry/it/AI%E3%83%BB%E6%A9%9F%E6%A2%B0%E5%AD%A6%E7%BF%92
- https://b.hatena.ne.jp/hotentry/it/%E3%81%AF%E3%81%A6%E3%81%AA%E3%83%96%E3%83%AD%E3%82%B0%EF%BC%88%E3%83%86%E3%82%AF%E3%83%8E%E3%83%AD%E3%82%B8%E3%83%BC%EF%BC%89
- https://b.hatena.ne.jp/hotentry/it/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E6%8A%80%E8%A1%93
- https://b.hatena.ne.jp/hotentry/it/%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2
- Extract **title, original article URL, and bookmark count** for each entry
- Use the original article URL, not the Hatena Bookmark entry page URL

**Global (Hacker News)**

- https://news.ycombinator.com/
- Extract **title, HN comment page URL (`https://news.ycombinator.com/item?id=XXXXX`), and points** for each story
- **Use the HN comment page URL, not the original article URL** (so comments are accessible)
- **Translate titles to English** (they are usually already in English; normalize if not)

**Security (additional sources)**

- https://www.aikido.dev/blog — security for developer-focused audiences
- https://www.wiz.io/blog — cloud security
- Check the latest 1–3 articles; include any with interest rating ★★★ in notable topics

**Reddit (13 subreddits)**

- **Important**: WebFetch is blocked for reddit.com — use the **Bash tool with curl**
- Fetch top 10 posts per subreddit via `/hot.json?t=day&limit=10`
- Use **old.reddit.com** (not www.reddit.com)
- Set User-Agent header: `"User-Agent: tech-trend-collector/1.0 (trend analysis tool)"`
- Extract **title, full Reddit comment page URL, upvotes (ups), and comment count**
- **Translate titles to English**

Example curl command (run via Bash tool):

```bash
curl -s -H "User-Agent: tech-trend-collector/1.0 (trend analysis tool)" \
  "https://old.reddit.com/r/programming/hot.json?t=day&limit=10" | \
  jq -r '.data.children[] | "\(.data.title)|\(.data.ups)|\(.data.num_comments)|https://www.reddit.com\(.data.permalink)"'
```

Data fields:

- `data.children[].data.title`: title
- `data.children[].data.ups`: upvotes
- `data.children[].data.num_comments`: comment count
- `data.children[].data.permalink`: path (prepend `https://www.reddit.com` for full URL)

Security (2 subreddits):

- r/netsec
- r/cybersecurity

AI (3 subreddits):

- r/OpenAI
- r/LocalLLaMA
- r/ClaudeCode

Core tech (2 subreddits):

- r/programming
- r/technology

OSS / indie dev (3 subreddits):

- r/opensource
- r/indiehackers
- r/webdev

Crypto (3 subreddits):

- r/CryptoCurrency
- r/CryptoTechnology
- r/defi

### 2. Analyze

Analyze collected content from the following angles:

**Interest-area matching (highest priority)**

- Cross-reference each article against interest areas in `mkontani-interests.md`
- Place high-relevance articles at the top of "Notable Topics"
- Topics to prioritize:
  - AI (dev tools, security, ethics)
  - Security (vulnerabilities, attack techniques, defenses)
  - OSS / indie dev (success stories, marketing, monetization)
  - Crypto / Blockchain / DeFi (EVM, Solana, Sui, stablecoins, protocols)
  - JavaScript / TypeScript (new tech, tools, frameworks)

**Hatena Bookmark IT**

- Topics resonating with Japanese engineers
- Potentially controversial topics
- Tech trends (AI, dev practices, tooling)
- Crypto / Blockchain / DeFi (EVM, Solana, Sui, stablecoins, DeFi protocols)

**Hacker News**

- Globally trending tech topics
- Startup and product news
- Security (vulnerabilities, attacks, incidents)
- High-engagement discussions (high point count)

**Reddit (13 subreddits)**

- Security: latest threats, practical attack/defense techniques
- AI: OpenAI, local LLMs, Claude Code
- OSS / indie dev: OSS projects, personal projects, web dev
- Crypto: cryptocurrency, blockchain, DeFi
- Prioritize by upvote count and comment count (community signal)
- Favor actively debated topics (high comment count)

### 3. Output

**Return "Collection complete." first, then save results to `content/posts/daily/YYYYMMDD-trend.md`.**

Use the following format:

```markdown
# Trend Report: YYYY-MM-DD

## Hatena Bookmark IT (Japanese market)

### Notable Topics

| Title                         | Bookmarks | Interest | Category           | Notes                              |
| ----------------------------- | --------- | -------- | ------------------ | ---------------------------------- |
| [Title](original article URL) | XXX users | ★★★/★★/★ | AI/Dev/Crypto/etc. | Points useful for content creation |

**Interest rating definition**:

- ★★★: Directly related to interest areas (AI x security, OSS, indie dev, crypto/blockchain/DeFi, etc.)
- ★★: Indirectly related (general tech trends, engineering culture)
- ★: General IT/tech news

### All Entries

1. [Title](original article URL) (XXX users) — summary
2. ...

## Hacker News (global)

### Notable Topics

| Title                        | Points | Interest | Category             | Notes                              |
| ---------------------------- | ------ | -------- | -------------------- | ---------------------------------- |
| [Title](HN comment page URL) | XXXpt  | ★★★/★★/★ | AI/Security/Dev/etc. | Points useful for content creation |

### All Entries

1. [Title](HN comment page URL) (XXXpt) — summary
2. ...

## Reddit (13 subreddits)

### Notable Topics

| Title                                 | Upvotes | Comments | Interest | Category             | Subreddit   | Notes                              |
| ------------------------------------- | ------- | -------- | -------- | -------------------- | ----------- | ---------------------------------- |
| [Title](full Reddit comment page URL) | XXX ups | XXX      | ★★★/★★/★ | Security/AI/OSS/etc. | r/subreddit | Points useful for content creation |

### Entries by Category

#### Security

1. [Title](Reddit comment page URL) (XXX ups, XXX comments) — r/netsec — summary
2. ...

#### AI

1. [Title](Reddit comment page URL) (XXX ups, XXX comments) — r/OpenAI — summary
2. ...

#### OSS / Indie Dev

1. [Title](Reddit comment page URL) (XXX ups, XXX comments) — r/opensource — summary
2. ...

#### Crypto

1. [Title](Reddit comment page URL) (XXX ups, XXX comments) — r/CryptoCurrency — summary
2. ...
```

## Notes

- Use the WebFetch tool to retrieve content
- **Every article must include a URL link — no link-less entries**
- **Hatena Bookmark: always extract the original article URL**, not the Hatena entry page URL
- **Hacker News: use the HN comment page URL (`item?id=` format)**, not the original article URL
- **Reddit: use the full Reddit comment page URL (`https://www.reddit.com/r/subreddit/comments/...`)**
- **All titles must be in English**
- Be mindful of Reddit API rate limits (approximately 60 requests per minute)
- Prioritize articles with high upvotes / comment counts / bookmark counts
- Use the actual execution date for YYYYMMDD in the output filename
