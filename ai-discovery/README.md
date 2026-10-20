# AI search and unbranded discovery kit

Drop-in files and content drafts so AI answer engines (ChatGPT, Perplexity, Claude, Gemini / Google AI Overviews) can find, read and recommend Epsylo for unbranded questions such as *"best app to learn grammar and sentence structure"* or *"Grammario alternative"*.

The app's source is private, so nothing here is deployed yet. Each file says where it goes.

> This repository is public. Anything committed here, drafts included, can be read by anyone.

| File | Deploy to | Status |
|---|---|---|
| [`site-root/llms.txt`](site-root/llms.txt) | `https://epsylo.app/llms.txt` | Ready. Update the Links section if routes change. |
| [`site-root/robots.txt`](site-root/robots.txt) | `https://epsylo.app/robots.txt` | Ready. Merge with any existing `Disallow` rules. |
| [`comparisons/epsylo-vs-grammario.md`](comparisons/epsylo-vs-grammario.md) | `/epsylo-vs-grammario` | **Draft.** Grammario column must be verified first. |
| [`guides/german-word-order.md`](guides/german-word-order.md) | `/guides/german-word-order` | Ready for review. |
| [`guides/japanese-particles.md`](guides/japanese-particles.md) | `/guides/japanese-particles` | Ready for review. |

---

## 1. Let AI crawlers in

### robots.txt
Deploy `site-root/robots.txt`. It allows the search/answer bots (OAI-SearchBot, ChatGPT-User, PerplexityBot, Perplexity-User, Claude-SearchBot, Claude-User) and the training bots (GPTBot, ClaudeBot, Google-Extended, Applebot-Extended).

Decide on the training bots on purpose. Allowing them is how a model learns to recommend Epsylo without searching; blocking them is a legitimate choice too, and doesn't affect live AI search.

`Google-Extended` is a robots.txt token, not a crawler: Googlebot still does the fetching. Blocking Googlebot anywhere blocks Google AI features too.

### Cloudflare / CDN checklist
robots.txt does nothing if the CDN blocks the request first. Check each of these in the Cloudflare dashboard (names as of 2025–2026; they move):

- [ ] **Security → Bots → "Block AI bots"** (or *AI Crawl Control*): off, or set to allow the bots above. Cloudflare turned AI-bot blocking on by default for many new zones in 2025.
- [ ] **Managed robots.txt**: if enabled, Cloudflare prepends its own `Disallow` rules for AI bots to your file. Turn it off or check the served file.
- [ ] **Bot Fight Mode / Super Bot Fight Mode**: can challenge verified bots. Make sure "Verified bots" are allowed.
- [ ] **WAF custom rules / rate limits**: no rule matching `cf.client.bot` or AI user agents with Block / Challenge.
- [ ] **Under Attack mode / JS challenge**: off for public marketing pages. A JS challenge blocks every crawler.
- [ ] **Pay per crawl** (if your plan has it): not set to charge or block these bots.

### Headers and rendering
- [ ] No `X-Robots-Tag: noindex` or `<meta name="robots" content="noindex">` on public pages.
- [ ] Core content (hero, features, pricing, comparison tables, guide text) is in the **server-rendered HTML**. Most AI crawlers do not run JavaScript. If the site is a client-rendered SPA, pre-render or SSR these routes.
- [ ] `llms.txt` and `robots.txt` served as `text/plain`, status 200, not behind a challenge.
- [ ] Add all new routes to `sitemap.xml`.

### Verify after deploying
Run from a machine outside your network:

```sh
for ua in "OAI-SearchBot/1.0; +https://openai.com/searchbot" \
          "PerplexityBot/1.0; +https://perplexity.ai/perplexitybot" \
          "ClaudeBot/1.0; +claudebot@anthropic.com" \
          "Claude-SearchBot/1.0" \
          "GPTBot/1.1; +https://openai.com/gptbot"; do
  printf '%-40s ' "$ua"
  curl -s -o /dev/null -w '%{http_code}\n' -A "Mozilla/5.0 (compatible; $ua)" https://epsylo.app/
done
curl -s https://epsylo.app/robots.txt
curl -s https://epsylo.app/llms.txt | head
# Content must be in raw HTML, not only after JS:
curl -s https://epsylo.app/epsylo-vs-grammario | grep -c '<table'
```

Expect `200` for each. Spoofed user agents won't trip IP-verified rules, so also check **Cloudflare → Security → Events** filtered by those bots after a few days, and the bot reports in Google Search Console / Bing Webmaster Tools.

---

## 2. Comparison and alternative pages

Start with `comparisons/epsylo-vs-grammario.md`. Rules that make these pages work for AI answers:

- One-paragraph **short answer** first, then the **table**, both in the first screen of HTML.
- Real `<table>` markup, one fact per cell, no images of tables.
- Describe the competitor accurately and fairly, with a "last checked" date. AI systems cross-check claims, and an unfair page gets ignored.
- Include a "Which should you use?" section that admits when the competitor is the better fit. That honesty is what gets a page quoted.
- `FAQPage` + `SoftwareApplication` JSON-LD matching the visible text.

Next pages to build with the same template, once you've confirmed who users compare you with:
- `/grammario-alternatives` (list format: "best alternatives to Grammario")
- `/epsylo-vs-deepl` and `/epsylo-vs-google-translate` (translation users who want explanations)
- `/best-apps-to-learn-grammar` (honest roundup that includes competitors)

---

## 3. Long-tail grammar guides

`guides/` has two complete guides. The pattern: direct answer and a summary table in the first ~30% of the page, then detail, common mistakes, and a short, factual Epsylo mention at the end.

Candidate topics, picked from your four hand-written profiles (EN, IT, JA, ES) plus high-demand German:

- Japanese: keigo levels explained · て-form · ている vs てある
- Spanish: ser vs estar · por vs para · when to use the subjunctive
- Italian: passato prossimo vs imperfetto · essere or avere as auxiliary · articulated prepositions (del, nel, sul)
- German: the four cases · dative vs accusative prepositions · separable verbs
- English (for learners): present perfect vs simple past · phrasal verbs word order

Have a fluent speaker review each guide before publishing. Wrong grammar on a grammar site is the fastest way to lose both readers and AI citations.

---

## 4. Forum and community mentions

Mentions on Reddit, forums and review sites alongside competitors are a big input to what AI models recommend. They only help if they are genuine, and astroturfing gets accounts banned and brands called out on exactly the subreddits that matter.

Do:
- **Disclose every time.** "I built Epsylo, so I'm biased, but…" Reddit's rules and most subreddit rules require it, and disclosed founder replies are usually well received.
- Answer the question first, fully, without a link. Mention Epsylo only where it really fits (someone asking *why* a sentence is structured a certain way), and mention the alternatives too.
- Check each subreddit's self-promotion rules first. r/languagelearning, r/LearnJapanese, r/German, r/Spanish and r/italianlearning all have them; some allow promotion only in specific threads.
- Keep to Reddit's ~10% self-promotion guideline: most of your activity should be helpful answers that don't mention Epsylo.
- Post a "Show HN" / r/SideProject launch, and get listed on Product Hunt, AlternativeTo (as an alternative to Grammario and others) and relevant "awesome" lists.

Don't:
- Use multiple accounts, sock puppets, paid posts without disclosure, or ask friends to post as if they were neutral users.
- Write reviews of competitors or comment on their posts negatively.

---

## 5. Measure

- Every month, ask ChatGPT, Perplexity, Claude and Google (AI Overviews) a fixed set of prompts, logged in a sheet:
  - "best app to learn grammar and sentence structure"
  - "app that explains grammar of a translation"
  - "Grammario alternative"
  - "how does German word order work" / "difference between wa and ga"
  Record whether Epsylo is mentioned, the position, and which URL is cited.
- Watch referrals from `chatgpt.com`, `perplexity.ai`, `claude.ai`, `gemini.google.com` in analytics.
- Watch crawler hits per bot in Cloudflare or server logs.
