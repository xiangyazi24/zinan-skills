Write and publish a post to infsup.com (Zinan's public blog). Invoke this whenever writing or editing a blog post — do NOT work from memory of the conventions; they drift and the host/renderer facts have been gotten wrong repeatedly. This file is the single source of truth.

## The site — VERIFIED FACTS (do not trust older notes that say KaTeX / netlify.app)

- **Repo:** `~/.openclaw/workspace/zinan/infsup-site` (Hugo, theme `hugo-ivy`, HUGO 0.145.0 extended, `pagefind` search). Git remote `github.com/xiangyazi24/zinan-blog`, branch `main`.
- **LIVE SITE = `https://infsup.com`** (custom domain). **The `zinan-blog.netlify.app` subdomain is STALE — NEVER verify against it.** This is the recurring mistake: polling netlify.app gives 404 / empty sitemap while the post is live on infsup.com. Always verify on `infsup.com`. Netlify auto-deploys on push to `main`, ~1–2 min.
- **Math renderer = MathJax 3 (tex-svg)**, configured in `layouts/partials/head_custom.html` — NOT KaTeX (old notes are wrong). Predefined macros available: `\R \N \Z \Q \E \Pr \norm \abs` (and `\lvert \rvert`).
- **Content:** `content/{en,cn,math}/NNN-slug.md` (three categories). Static assets in `static/papers/`, `static/images/`. Posts are numbered; check the latest: `ls content/en | sort | tail`.

## Frontmatter (copy, then edit)

```
---
title: "..."
date: 2026-06-20T11:30:00-05:00     # MUST be ≤ now. A future date → Hugo SILENTLY drops the post.
slug: explicit-slug-here            # set explicitly; else Hugo slugifies the TITLE (URL ≠ filename → 404 guesses)
categories: ["en"]                  # en / cn / math
tags: ["...", "..."]
math: true
draft: false
---
```

## Markdown × MathJax collisions (goldmark parses BEFORE MathJax — these are goldmark rules, renderer-independent)

- **Each `$$…$$` display block needs a blank line ABOVE and BELOW it**, but **NO blank line between `$$` and the math** (a blank line splits `$$` and content into different `<p>` and MathJax can't pair them). Prose between two display blocks must be its own paragraph with blanks around it.
- **Inside a display block, no line may start with** `- ` / `* ` / `+ ` / `# ` / `> ` / `digit. ` — goldmark eats it as a list/heading/quote and the whole block renders as plain text. For a leading minus: write `-12M` (no space), `\bigl(-12M\bigr)`, or move the sign to the RHS (`12M = \frac{-…}{…}`).
- **In tables, inline `|…|` breaks columns** → use `\lvert … \rvert`.
- **Escape prose `_` and `*`** (markdown emphasis); inside `$…$`/`$$…$$` they are fine (subscript/etc.).
- **Avoid bare `{{` `}}`** in prose (Hugo shortcode delimiters).
- Inside `\text{…}`, a literal `\ ` (backslash-space) renders as a literal backslash — use a plain space.
- **Equation systems / ODE groups: ONE `\begin{cases}…\end{cases}`**, not several separate `$$` blocks — one coupled system reads as one. End each line with `\\`, align with `\;=\;`, use `\dfrac`. List initial values as a separate `cases`. For an explicit GPAC/PIVP post, show: state-variable definitions, the full ODE system, the full initial values, total state count + readout.
- **Figure captions** via `{{< figure >}}` do NOT pass through MathJax. For math in a caption use a markdown image `![alt](path)` + italic line `*caption with $math$*` underneath.

## Voice (this is a PUBLIC blog)

- **Third person / impersonal. NEVER "my dad" / "爸爸" / family relationships.** Cite the work academically — "Huang", "Huang and Huls", "a recent paper by …". A stranger reading it should not sense who the author is related to. (A private version may live in `zinan/blog-drafts/private/`, unpublished.)
- **Undergrad-math audience, do not skip steps.** Explain every new concept in a sentence or two; write intermediate steps (no "obviously"); concrete/intuition before the abstract statement; never assume the reader shares our context.
- **No internal codenames** (DNA32 / BAC / UCNC25 / ch13 / RTCRN2 …). Replace with a description of the problem + authors. Conference/internal numbers out; author + content stays.
- 朴、真、有出处. Technical content + literary quotation welcome; private/family content not.

## Signature (every post ends with one)

- One curated line — Chinese classical preferred (occasionally Western), **中英对照**, that **resonates with THIS post's theme**, chosen for this post, not pulled from a fixed pool. Format: original + source + English gloss.
- **Must not repeat a prior signature.** BANNED: 「工欲善其事，必先利其器」(《论语》, overused). Before choosing, scan what's been used: `grep -rhA3 '^---$' ~/.openclaw/workspace/zinan/infsup-site/content/{en,cn,math}/*.md | grep -E '《|—' | tail -40` (or read the tails of recent posts). Known-used incl. 「致广大而尽精微」(010), 「依乎天理，批大郤，导大窾」(022, Zhuangzi).
- 不掉书袋、不小女生气。If nothing truly fits, leave it out. 「文章千古事，字字不易」— run the chosen line by Xiang before publishing.

## Workflow: draft → self-check → review → push → VERIFY LIVE

1. **Write** `content/<lang>/NNN-slug.md` (next number).
2. **Self-check (mandatory before any push):**
   - goldmark/display-math lint:
     ```bash
     cd ~/.openclaw/workspace/zinan/infsup-site
     python3 -c "
     import re,sys
     p='content/en/NNN-slug.md'
     t=open(p).read(); parts=t.split('\$\$'); bad=0
     for i,blk in enumerate(parts):
         if i%2==1:
             for ln,line in enumerate(blk.split(chr(10)),1):
                 if re.match(r'^[-*+#>]\s|^[0-9]+\.\s',line): print('display-block leading-md:',ln,line[:70]); bad+=1
     print('display blocks',(len(parts)-1)//2,'issues',bad)"
     ```
   - local build: `hugo` → confirm `public/<lang>/<slug>/index.html` exists. If it doesn't, it did NOT publish (future date? draft:true? slug mismatch) — fix, do not push.
3. **DRAFT-FIRST (default, since post 014):** do NOT push yet. Show Xiang the key block — title, abstract, core formula/conclusion, the chosen signature — and wait for approval. Exception: he says "直接发" / "你看着发". Same for edits to a published post (show the diff first).
4. **Push** (only on approval): `git add content/<lang>/<file>.md && git commit -m "blog NNN: <title> — <one line>" && git push origin main`. Commit ONLY the content file (`public/` is not tracked; Netlify builds from source). End the commit message with the Co-Authored-By trailer.
5. **VERIFY LIVE on infsup.com (mandatory — the recurring failure):** wait ~1–2 min for Netlify, then:
   ```bash
   url=$(curl -s https://infsup.com/sitemap.xml | grep -oE 'https://infsup.com/<lang>/[^<]*<slug-fragment>[^<]*' | head -1)
   curl -s -o /dev/null -w '%{http_code}\n' "$url"            # must be 200
   curl -s "$url" | grep -oiE '<the exact title>'             # must show real content, not a 200-returning 404 page
   ```
   URL-encode specials in the slug (π→%CF%80, é→%C3%A9). If the sitemap hasn't refreshed, confirm via the page itself (200 + title + signature present). **Only give Xiang the link after 200 + content confirmed — never a guessed/unchecked URL.**

## Do NOT
- Verify against `zinan-blog.netlify.app` (stale) — only `infsup.com`.
- Assume KaTeX — it's MathJax 3.
- Push before the local `hugo` build shows the `public/<lang>/<slug>/` dir.
- Push a research/methodology post without Xiang's go-ahead on the draft block (post-014 rule).
- Send a link before a real `curl` 200 + content check.
- Reuse a signature; never 「工欲善其事」.

$ARGUMENTS may name a post (number/slug/title) or be empty (start a new one).
