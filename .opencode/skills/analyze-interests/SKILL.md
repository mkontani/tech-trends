---
name: analyze-interests
description: Fetch https://blog.nicopun.com/index.xml and summarize mkontani's tech interests into mkontani-interests.md
compatibility: opencode
---

## What I do

Fetch the RSS feed at `https://blog.nicopun.com/index.xml`, analyze all entries,
and write (or overwrite) `mkontani-interests.md` in the repository root with a
structured summary of mkontani's technical interests and curiosity areas.

## Steps

1. Fetch `https://blog.nicopun.com/index.xml` with `WebFetch`.
2. Parse every `<item>` — extract title, description, and any URLs referenced.
3. Identify tech-relevant signals: tools mentioned, protocols discussed, opinions
   on specific technologies, recurring themes.
4. Cluster signals into interest areas (see seed clusters below).
5. Write `mkontani-interests.md` following the output format below.

## Seed interest clusters (update as new signals emerge)

- **LLM / AI coding** — Claude (especially Sonnet/Opus), OpenAI API, AI-assisted
  development, context windows, prompt engineering, agent-to-agent protocols
- **Auth / Identity** — FIDO, ID-JAG, x402, authorization delegation to IdP,
  autonomous agent authentication
- **Blockchain / Web3** — Ethereum/EVM, Solana, Sui, stablecoins, x402 HTTP
  payment protocol, tokenization, DAO/Web3 work styles
- **OSS & tooling** — OSS discovery, GitHub trending, evaluating new libraries,
  contribution quality in the AI era
- **Frontend / BaaS / indie dev** — static frontends, Netlify/Vercel/GitHub Pages,
  BaaS (Supabase-style), MVP-first approach, UI/UX focus
- **Security** — cyber-security awareness, protocol-layer security considerations

## Output format

Write `mkontani-interests.md` with the following structure:

```markdown
# mkontani's Tech Interests

> Auto-generated from https://blog.nicopun.com/index.xml
> Last updated: YYYY-MM-DD

## Interest Areas

### <Area name>

**Summary:** One or two sentences describing the interest.

**Signals from blog:**
- "<post title>" (YYYY-MM-DD) — one-line note on what was discussed
- ...

**Relevance for curation:** What kinds of articles/trends to surface for this area.

---
```

Repeat the block for each identified interest area. End the file with:

```markdown
## Tags to prioritize in this repository

List the AGENTS.md taxonomy tags most aligned with mkontani's interests, ordered
by relevance.
```

## Constraints

- Only include areas with at least one concrete signal from the feed.
- Do not invent interests not evidenced by the feed content.
- Keep each "Signals from blog" list to the 3 most recent or most specific entries.
- Run `markdownlint-cli2 mkontani-interests.md` after writing and fix all warnings.
