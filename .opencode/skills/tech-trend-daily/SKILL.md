---
name: tech-trend-daily
description: "Daily tech trend collection customized for mkontani"
---

# Daily Trend Collection

Collect trending articles from Hatena Bookmark IT, Hacker News, Reddit, and security
blogs, then save results to `content/posts/daily/YYYYMMDD-trend.md`.

## CRITICAL: Token Budget Strategy

This skill fetches many sources. To stay within the model's context window, follow
these rules strictly:

1. **Always use the Task tool (subagent) for data collection.** Never fetch URLs
   directly from the main conversation. Launch subagents that fetch, extract, and
   return **only structured summary data** (title, URL, metrics). Raw HTML must
   never reach the main context.
2. **Use Bash + curl + jq/sed** instead of WebFetch wherever possible. Pipe fetched
   content through `jq` or `sed` to extract only the needed fields before the
   output is returned.
3. **Limit list sizes.** Keep only the top items per source as specified below.

---

## Steps

### 0. Load user profile

Read `mkontani-interests.md` to understand mkontani's technical interest areas.

### 1. Collect trend data (use Task tool subagents)

Launch **three Task-tool subagents in parallel** for the three source groups below.
Each subagent must return **only a compact structured text list** — never raw HTML.

---

#### Subagent A: Hatena Bookmark IT (Japanese market)

Prompt the subagent with:

> Fetch the Hatena Bookmark IT RSS feed and extract trending articles.
>
> Use the Bash tool to run:
>
> ```bash
> curl -s 'https://b.hatena.ne.jp/hotentry/it.rss' | \
>   python3 -c "
> import sys, xml.etree.ElementTree as ET
> tree = ET.parse(sys.stdin)
> ns = {'rdf':'http://www.w3.org/1999/02/22-rdf-syntax-ns#',
>        'rss':'http://purl.org/rss/1.0/',
>        'dc':'http://purl.org/dc/elements/1.1/',
>        'hatena':'http://www.hatena.ne.jp/info/xmlns#'}
> items = tree.findall('.//rss:item', ns)[:30]
> for i, item in enumerate(items, 1):
>     title = item.find('rss:title', ns)
>     link = item.find('rss:link', ns)
>     count = item.find('hatena:bookmarkcount', ns)
>     t = title.text if title is not None else 'N/A'
>     l = link.text if link is not None else 'N/A'
>     c = count.text if count is not None else '0'
>     print(f'{i}. [{t}]({l}) ({c} users)')
> "
> ```
>
> If the python3 approach fails, fall back to:
>
> ```bash
> curl -s 'https://b.hatena.ne.jp/hotentry/it.rss' | \
>   grep -oP '(?<=<item rdf:about=").*?(?=")' | head -30
> ```
>
> and then extract titles and bookmark counts similarly.
>
> Return ONLY the numbered list in this exact format — nothing else:
>
> ```
> 1. [Title](URL) (XXX users)
> 2. [Title](URL) (XXX users)
> ...
> ```
>
> Maximum 30 entries. No commentary, no raw HTML.

---

#### Subagent B: Hacker News (global)

Prompt the subagent with:

> Fetch the top 30 Hacker News stories using their API.
>
> Use the Bash tool to run:
>
> ```bash
> ids=$(curl -s 'https://hacker-news.firebaseio.com/v0/topstories.json' | jq '.[0:30] | .[]')
> for id in $ids; do
>   curl -s "https://hacker-news.firebaseio.com/v0/item/${id}.json" | \
>     jq -r '"[\(.title)](https://news.ycombinator.com/item?id=\(.id)) (\(.score)pt)"'
> done
> ```
>
> Return ONLY the numbered list in this exact format — nothing else:
>
> ```
> 1. [Title](https://news.ycombinator.com/item?id=XXXXX) (XXXpt)
> 2. [Title](https://news.ycombinator.com/item?id=XXXXX) (XXXpt)
> ...
> ```
>
> Maximum 30 entries. No commentary, no raw HTML.

---

#### Subagent C: Reddit (13 subreddits)

Prompt the subagent with:

> Fetch top posts from 13 Reddit subreddits using curl + jq.
>
> **Important**: Use old.reddit.com with the JSON API. Fetch 5 posts per subreddit.
> Add a 1-second delay between requests to respect rate limits.
>
> Subreddits grouped by category:
>
> - Security: netsec, cybersecurity
> - AI: OpenAI, LocalLLaMA, ClaudeCode
> - Core tech: programming, technology
> - OSS / indie dev: opensource, indiehackers, webdev
> - Crypto: CryptoCurrency, CryptoTechnology, defi
>
> For each subreddit, run:
>
> ```bash
> curl -s -H "User-Agent: tech-trend-collector/1.0 (trend analysis tool)" \
>   "https://old.reddit.com/r/SUBREDDIT/hot.json?t=day&limit=5" | \
>   jq -r '.data.children[] | "[\(.data.title)]( https://www.reddit.com\(.data.permalink)) (\(.data.ups) ups, \(.data.num_comments) comments)"'
> ```
>
> Return ONLY a categorized list in this exact format — nothing else:
>
> ```
> ### Security
> r/netsec:
> 1. [Title](URL) (XXX ups, XXX comments)
> ...
> r/cybersecurity:
> 1. [Title](URL) (XXX ups, XXX comments)
> ...
>
> ### AI
> r/OpenAI:
> ...
>
> ### Core tech
> ...
>
> ### OSS / Indie Dev
> ...
>
> ### Crypto
> ...
> ```
>
> No commentary, no raw JSON, no HTML.

---

#### Security blogs (inline — small payload)

After the three subagents return, optionally check these security blogs:

- https://www.aikido.dev/blog
- https://www.wiz.io/blog

Use WebFetch with `format: text` and extract only the latest 1-3 article titles
and URLs. Include any with interest rating three-star in notable topics. If this would
add too much context, skip these sources.

---

### 2. Analyze

Analyze the collected structured data from the following angles:

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

- **Use Task tool subagents for all data collection** to keep raw content out of main context
- Use Bash + curl + jq instead of WebFetch for structured APIs (HN, Reddit)
- **Every article must include a URL link — no link-less entries**
- **Hatena Bookmark: always extract the original article URL**, not the Hatena entry page URL
- **Hacker News: use the HN comment page URL (`item?id=` format)**, not the original article URL
- **Reddit: use the full Reddit comment page URL (`https://www.reddit.com/r/subreddit/comments/...`)**
- **All titles must be in English** (translate Japanese titles)
- Be mindful of Reddit API rate limits (approximately 60 requests per minute)
- Prioritize articles with high upvotes / comment counts / bookmark counts
- Use the actual execution date for YYYYMMDD in the output filename
