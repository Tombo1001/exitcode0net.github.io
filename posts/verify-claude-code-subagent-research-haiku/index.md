# How to Check a Cheap Claude Code Subagent's Research Before You Believe It

> Two Haiku subagents gave me confident research. One invented a Google policy quote and misdescribed a Home Assistant release. Here's the check I run now.

Source: https://exitcode0.net/posts/verify-claude-code-subagent-research-haiku/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-10-23
Updated: 2026-10-23
Tags: claude-code, subagents, haiku, ai-agents, verification


I handed two Claude Code subagents a research job. Both ran on Haiku to keep the cost down, and both came back with tidy tables, exact quotes and source URLs. Then I opened the sources, and a fair amount of what they'd told me was wrong. This post is what I found, and the small script I now use before acting on anything a cheap subagent tells me.

## What I asked them

I had a Claude Code session working through SEO and AdSense compliance on this blog, and I spun two background subagents off it with the model set to Haiku:

1. Read Google's current AdSense and publisher policies, then scan the site for anything that could get the account suspended.
2. Find what's genuinely new in the last 60 days across Home Assistant, Proxmox, local AI and Claude Code, and rank 15 post ideas.

This was Claude Code 2.1.292 on the same dev VM I wrote about in [turning a laptop into a dumb terminal](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/). The subagents had web search and fetch tools and nothing that could edit files.

## The tell was in the usage line

When a subagent finishes, Claude Code reports how many tool calls it made. The policy agent made 25. The topic-research agent made 2.

Two tool calls can't produce fifteen topics, each with dated evidence and three competing URLs. Most of that detail can't have come from what the agent read. I'd have spotted the problem sooner if I'd looked at that number first.

## What the checks found

I ran each claim that mattered against the primary source.

| Claim | What I checked | Result |
|---|---|---|
| Google's publisher policy says publishers "may not hide or remove ads", so a script on my site that removes auto ads is High risk | Fetched the policy page, 25,494 characters of text, and searched for the phrase | Zero matches. The AdSense program policy actually says publishers "are permitted to make modifications to the AdSense ad code so long as those modifications do not artificially inflate ad performance or harm advertisers" |
| Home Assistant 2026.9 headline feature is a native Ollama integration and a 9.5x streaming TTS speedup | Fetched the release post and the changelog | The release post mentions Ollama zero times and "9.5" zero times. The headline is Modbus, with 22 mentions, plus the Security dashboard and a Matter network map. The changelog has one Ollama line, a test fix |
| Google bans content that promotes hacking or cracking, and my Wi-Fi adapter driver post is exposed | Read the same policy page, then the post | The policy text is real, and the post did frame itself around recovering WPA passwords. I reworded it and added an authorised-use note |
| Pi shipped 1.0 and a 1.1 in early October | Fetched pi.dev/changelog | 1.1.0 is dated 7 October 2026 and the banner says Pi 1.0 shipped. I didn't confirm the 2 October date the agent gave for 1.0 |

I also fetched eleven of the URLs the topic agent cited. Nine returned 200, one returned 403 because of bot protection and one returned 429. Every one of them existing proved nothing about whether it said what the agent claimed. The Home Assistant page returned 200 and contradicted the agent's description of it.

So the agent was right about where to look and wrong about what was there. The first is useful. The second needs checking before you use it.

## The check

The rule I use now is that a quote or a number doesn't count until I've seen it in the source text myself. I wrote a script so that takes seconds. Save it as `check_quote.py`:

```python
#!/usr/bin/env python3
"""Check whether a quote or number really appears on a page.

  python3 -I check_quote.py URL "exact phrase" ["another phrase" ...]

Prints the HTTP status, the page title, and how many times each phrase
appears in the visible text, with a little context for the first hit.
Standard library only.
"""
import html, re, sys, urllib.request

url, phrases = sys.argv[1], sys.argv[2:]
request = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
try:
    with urllib.request.urlopen(request, timeout=30) as response:
        status, raw = response.status, response.read().decode("utf-8", "replace")
except Exception as error:
    sys.exit(f"{url}: could not fetch ({error})")

title = re.search(r"<title>(.*?)</title>", raw, re.S)
text = re.sub(r"<(script|style).*?</\1>", " ", raw, flags=re.S | re.I)
text = html.unescape(re.sub(r"\s+", " ", re.sub(r"<[^>]+>", " ", text)))

print(f"{status}  {html.unescape(title.group(1).strip())[:80] if title else '(no title)'}  [{len(text):,} chars of text]")
for phrase in phrases:
    hits = [m.start() for m in re.finditer(re.escape(phrase), text, re.I)]
    print(f"  {len(hits):3d}x  {phrase!r}")
    if hits:
        start = max(0, hits[0] - 70)
        print(f"        ...{text[start:hits[0] + len(phrase) + 70]}...")
```

Run it with the URL and the exact phrases you want to confirm:

```bash
python3 -I check_quote.py https://support.google.com/publisherpolicies/answer/10502938 "hide or remove ads" "hacking or cracking"
```

```text
200  Google Publisher Policies - Publisher Policies Help  [25,494 chars of text]
    0x  'hide or remove ads'
    1x  'hacking or cracking'
        ...; information or products for passing drug tests promotes any form of hacking or cracking and/or provides users with instructions, equipment, or software that...
```

Zero matches on the first phrase is the whole story. The `-I` flag stops Python importing anything from the current directory, which is a good habit whenever you run a script next to files you've downloaded.

## What I do differently now

- Look at the tool call count before the report. A long, well-sourced answer from a handful of calls is a warning.
- Ask for exact quotes, then check them with a script.
- Treat every model-written sentence about a page as a pointer. The page is the evidence.
- Keep the verifier separate from the researcher. Asking the same agent whether it was right gets you the same confidence back.
- Act on the checked findings only. The Wi-Fi post change went in because I'd read the policy myself. The ad-removal script stayed for now because the quote that condemned it didn't exist.

It's the same instinct behind [the dependency audit extension I built for Pi Agent](https://exitcode0.net/posts/pi-agent-dep-audit-supply-chain/): check what an agent hands you against a source before you trust it.

## It still saved time

The policy agent found a real risk I'd have missed, and it pointed me at exactly the right Google page. The topics agent was a bad researcher but a decent list of places to look: the Pi changelog and the Home Assistant release were both worth reading. Checking took a few minutes, and reading all of those sources myself would have taken far longer.

## Or just hand this page to your agent

If your own subagents hand you research, paste this into the Claude Code session that received it:

```text
Read https://exitcode0.net/posts/verify-claude-code-subagent-research-haiku/ and verify the research report above before I act on it. First list every quote, number and URL in the report, and how many tool calls the subagent made. Then fetch each cited page and use the check_quote.py script from that post, or an equivalent, to count exact matches for each quote or number. Give me a table with the claim, the page, the match count and a verdict of confirmed, not found or contradicted. Don't edit any files, don't act on anything marked not found or contradicted, and ask me before you run anything other than fetching pages.
```

The post gives the agent the script and the method. Only you know which claims in the report you're about to act on.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/verify-claude-code-subagent-research-haiku/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
