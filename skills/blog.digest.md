# blog skill — DIGEST (auto-generated, do not edit)
# Full version: ~/repos/zinan-skills/skills/blog.md


## The site — VERIFIED FACTS (do not trust older notes that say KaTeX / netlify.app)
- **Repo:** `~/.openclaw/workspace/zinan/infsup-site` (Hugo, theme `hugo-ivy`, HUGO 0.145.0 extended, `pagefind` search). Git remote `github.com/xiangyazi24/zinan-blog`, branch `main`.
  Key: LIVE SITE = `https://infsup.com` | Math renderer = MathJax 3 (tex-svg) | The `zinan-blog.netlify.app` subdomain is STALE — NEVER verify against it.

## Frontmatter (copy, then edit)
---

## Markdown × MathJax collisions (goldmark parses BEFORE MathJax — these are goldmark rules, renderer-independent)
- **Each `$$…$$` display block needs a blank line ABOVE and BELOW it**, but **NO blank line between `$$` and the math** (a blank line splits `$$` and content into different `<p>` and MathJax can't pair them). Prose between two display blocks must be its own paragraph with blanks around it.
  Key: Avoid bare `{{` `}}` | Equation systems / ODE groups: ONE `\begin{cases}…\end{cases}` | Figure captions | In tables, inline `|…|` breaks columns | Inside a display block, no line may start with

## Voice (this is a PUBLIC blog)
- **Third person / impersonal. NEVER "my dad" / "爸爸" / family relationships.** Cite the work academically — "Huang", "Huang and Huls", "a recent paper by …". A stranger reading it should not sense who the author is related to. (A private version may live in `zinan/blog-drafts/private/`, unpublished.)
  Key: No internal codenames | Undergrad-math audience, do not skip steps.

## Signature (every post ends with one)
- One curated line — Chinese classical preferred (occasionally Western), **中英对照**, that **resonates with THIS post's theme**, chosen for this post, not pulled from a fixed pool. Format: original + source + English gloss.
  Key: After publishing each post, APPEND its signature's source+line to the USED list below | Must not repeat a prior signature — pick a source/line NOT in the USED list below. | USED (do NOT reuse — by source + opening words; any line from these counts as used): | 明令禁用(爸爸点名)

## Workflow: draft → self-check → review → push → VERIFY LIVE
1. **Write** `content/<lang>/NNN-slug.md` (next number).
  1. **Write**
  2. **Self-check (mandatory before any push):**
  3. **DRAFT-FIRST (default, since post 014):**
  4. **Push**
  5. **VERIFY LIVE on infsup.com (mandatory — the recurring failure):**

## Do NOT
- Verify against `zinan-blog.netlify.app` (stale) — only `infsup.com`.
