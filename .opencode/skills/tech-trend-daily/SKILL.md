---
name: tech-trend-daily
description: "Daily tech trend collection customized for mkontani"
---

# Daily Trend Collection

Collect trending articles from Hatena Bookmark IT, Hacker News, Reddit, and Feedly
OPML subscriptions, then save results to `content/posts/daily/YYYYMMDD-trend.md`.

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

Launch **four Task-tool subagents in parallel** for the four source groups below.
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

#### Subagent D: Feedly OPML subscriptions

Prompt the subagent with:

> Fetch recent articles from RSS feeds listed in the Feedly OPML file.
>
> **CRITICAL — Token budget rules:**
>
> - There are ~44 feeds in the OPML. Many will return errors or be slow.
>   Set a **5-second curl timeout** per feed (`--max-time 5`).
> - Extract at most **3 items per feed** (the latest 3).
> - Return at most **5 items per category** (keep only the most recent).
> - Total output must not exceed **120 lines**. If it does, truncate the
>   lowest-priority categories first (Hacker > News > Devops).
> - Never return raw XML/HTML. Return only the structured list below.
>
> **Step 1 — Parse the OPML file**
>
> Read the file `feedly-opml-a7e028cf-ff56-443f-b8f7-1dbf2b6d8390.opml`
> in the repository root. Extract the `xmlUrl` and `title` for every
> `<outline type="rss" ...>` element, grouped by parent category
> (Devops, Hacker, News).
>
> **Step 2 — Fetch feeds with a single bash script**
>
> Write and execute a bash script that:
>
> 1. Loops through each feed URL.
> 2. Fetches with `curl -s --max-time 5`.
> 3. Pipes through `python3` (or `grep`/`sed` fallback) to extract the
>    latest 3 `<item>` or `<entry>` elements, outputting title, link, and
>    publication date.
> 4. Sleeps 0.5 s between requests to be polite.
> 5. Skips feeds that return errors or empty results silently.
>
> Recommended python3 one-liner for each feed:
>
> ```bash
> curl -s --max-time 5 "$url" | python3 -c "
> import sys, xml.etree.ElementTree as ET
> try:
>     tree = ET.parse(sys.stdin)
>     root = tree.getroot()
>     ns = {'atom':'http://www.w3.org/2005/Atom'}
>     items = root.findall('.//item')[:3] or root.findall('.//atom:entry', ns)[:3]
>     for it in items:
>         title = it.findtext('title') or it.findtext('atom:title', namespaces=ns) or 'N/A'
>         link = it.findtext('link') or ''
>         if not link:
>             le = it.find('atom:link', ns)
>             link = le.get('href','') if le is not None else ''
>         date = it.findtext('pubDate') or it.findtext('dc:date') or it.findtext('atom:updated', namespaces=ns) or ''
>         print(f'- [{title}]({link}) ({date})')
> except Exception:
>     pass
> "
> ```
>
> **Step 3 — Format output**
>
> Return ONLY a categorized list in this exact format — nothing else:
>
> ```
> ### Devops
> Kubernetes Blog:
> - [Title](URL) (date)
> Google Online Security Blog:
> - [Title](URL) (date)
> ...
>
> ### Hacker
> DevelopersIO:
> - [Title](URL) (date)
> ...
>
> ### News
> Publickey:
> - [Title](URL) (date)
> Krebs on Security:
> - [Title](URL) (date)
> ...
> ```
>
> Maximum 5 items per category (15 total across 3 categories).
> No commentary, no raw XML, no HTML.

---

### 1.5. Deduplicate against recent reports

Before analysis, check for articles that already appeared in previous daily
reports. This prevents the same trending topic from surfacing day after day.

1. **Read recent reports.** Use the Glob tool to find files matching
   `content/posts/daily/*-trend.md`. From the results, select the most recent
   **3 files** (excluding today's date). Read each file.
2. **Extract all URLs.** From each file, collect every URL that appears inside
   a Markdown link `[...](URL)`. Build a set of **known URLs**.
3. **Build an exclusion list.** Any article URL (after stripping trailing
   slashes, query parameters, and fragments) that matches a known URL is
   marked as a **duplicate**.
4. **Apply during analysis.** In Step 2 (Analyze):
   - **Exclude duplicates from Notable Topics tables entirely.** They must not
     appear in any Notable Topics section.
   - **Mark duplicates in All Entries / Entries by Category lists** by
     appending `[dup]` after the summary. Keep them in the list for
     completeness but move them to the bottom of their section.

If no previous reports exist (first run), skip this step.

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

**Feedly OPML subscriptions**

- Devops: Kubernetes, CNCF, Docker, Istio, HashiCorp, security (Google, Flatt,
  LAC, Tokumaru, smallstep, Datadog) — infrastructure and security updates
- Hacker (JP engineering blogs): DevelopersIO, Mercari, LINE/LY Corp, DeNA,
  CyberAgent, Cookpad, Hatena, Cybozu, Darknet — JP engineering culture and
  security research
- News: Publickey, GIGAZINE, TechCrunch Japan, Krebs on Security, piyolog,
  FIDO Alliance, DigiCert — breaking tech/security news
- Cross-reference with interest areas: security disclosures, auth/FIDO updates,
  and JP engineering blog posts are highest priority

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

## Feedly OPML Subscriptions

### Notable Topics

| Title                | Interest | Category             | Source    | Notes                              |
| -------------------- | -------- | -------------------- | --------- | ---------------------------------- |
| [Title](article URL) | ★★★/★★/★ | Security/DevOps/etc. | Feed name | Points useful for content creation |

### Entries by Category

#### Devops

1. [Title](URL) — Source feed name — summary
2. ...

#### Hacker (JP Engineering Blogs)

1. [Title](URL) — Source feed name — summary
2. ...

#### News

1. [Title](URL) — Source feed name — summary
2. ...
```

## Notes

- **Deduplication: articles that appeared in the last 3 daily reports are excluded from Notable Topics and marked `[dup]` in All Entries.** URL matching is normalized (strip trailing `/`, query params, fragments) before comparison.
- **Use Task tool subagents for all data collection** to keep raw content out of main context
- Use Bash + curl + jq instead of WebFetch for structured APIs (HN, Reddit, RSS feeds)
- **Feedly OPML feeds: 5-second timeout per feed, max 3 items per feed, max 5 items per category (15 total)**
- **Every article must include a URL link — no link-less entries**
- **Hatena Bookmark: always extract the original article URL**, not the Hatena entry page URL
- **Hacker News: use the HN comment page URL (`item?id=` format)**, not the original article URL
- **Reddit: use the full Reddit comment page URL (`https://www.reddit.com/r/subreddit/comments/...`)**
- **All hyperlinks must open in a new tab (`_blank`)**
- For Markdown output, render links using HTML anchors to enforce this: `<a href="URL" target="_blank" rel="noopener noreferrer">Title</a>`
- **All titles can remain in their original language** (no translation required)
- Be mindful of Reddit API rate limits (approximately 60 requests per minute)
- Prioritize articles with high upvotes / comment counts / bookmark counts
- Use the actual execution date for YYYYMMDD in the output filename
