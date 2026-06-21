---
description: "Prediction-market AI-edge hunter, Kalshi-first. /poly = scan Kalshi (the legal, CFTC-regulated US venue with a real API) for liquid clearly-resolving markets, find edges two ways — (a) AI probability estimate (Claude, cross-checked via /chatgpt) vs market price, and (b) CROSS-VENUE ARBITRAGE between Kalshi and Polymarket on the same event — size with fractional Kelly, record the pick. Two legs, switchable: PAPER (simulated, to measure if the edge is real) and LIVE (small real money on Kalshi). Goal is money (mother's side business); honest EV — liquid markets are efficient, edges are mostly logic/arb not prophecy. Trigger: /poly, /poly arb, /poly paper, /poly live, '看看 kalshi/polymarket / 跨平台套利 / 有没有错价 / 估个概率 / poly 扫一遍'."
user-invocable: true
---

# /poly — prediction-market AI-edge hunter (Kalshi-first)

Turn Claude + ChatGPT into an edge-finder over prediction markets. Empirically
(see the first-run note in `poly-paper-log.md`) the naive thesis — "AI estimates
true probability better than the market" — **does not beat liquid, well-covered
markets**: they're efficient (the Fed-July market sat exactly on the CME
consensus). So the edge we actually chase is **logic, not prophecy**:

1. **Cross-venue arbitrage** — the *same* event priced differently on **Kalshi**
   vs **Polymarket**. Pure logic, doesn't require out-predicting the crowd. The
   most reliable edge here.
2. **AI probability edge** — only where it can work: **less-efficient corners**
   (lower-liquidity / low-attention markets, or fresh news the crowd hasn't
   priced). On efficient markets, expect ~0 edge and skip.

Money runs as the **mother's side business** (KYC/funding under her name, like
`/hunt`). The skill never touches KYC, keys, or payment credentials.

## Venues (why Kalshi-first)

- **Kalshi** — the **legal** US venue: CFTC-regulated exchange, KYC, USD/bank,
  and crucially a **real public+trading API**. This is where `live` money goes,
  and (unlike Polymarket US) automation is actually possible later. Public market
  data needs no auth; **trading uses RSA-PSS-signed requests** (generate an RSA
  keypair, upload the public key at Settings > API Keys; headers
  `KALSHI-ACCESS-KEY/-TIMESTAMP/-SIGNATURE`). Base: `https://api.elections.kalshi.com/trade-api/v2`.
- **Polymarket** — used **only as a price-comparison data source** for cross-venue
  arb. The international crypto platform is **US-geoblocked**; do NOT trade it
  from the US (ToS). Polymarket US exists but its trading API is unconfirmed — not
  our trading venue. Its public Gamma API is fine for reading prices.
- **⚠️ Insider-trading enforcement is active** (CFTC/Kalshi pursued traders who
  bet on their own campaigns / used classified intel). Edges must be **public
  information + logic only**.

## Live auth (Kalshi) — status & setup

- **Connected, read-only, verified 2026-06-20** on **production** ($100 balance,
  0 positions). Order placement is **NOT wired** — pending sandbox validation +
  per-order confirmation (see staging below).
- **Credentials live in `~/.kalshi/`** (NEVER the repo): `kalshi.pem` (RSA private
  key, `chmod 600`) + `kalshi.keyid` (the Key ID). Created at
  kalshi.com/account/profile → API Keys (private key shown once).
- **Signer**: `~/.openclaw/workspace/scripts/kalshi-auth.py` (RSA-PSS, loads from
  `~/.kalshi/`, `--env prod|demo`). Read-only summary:
  `python3 ~/.openclaw/workspace/scripts/kalshi-account.py --env prod`.
  Base URLs: prod `api.elections.kalshi.com/trade-api/v2`, demo
  `demo-api.kalshi.co/trade-api/v2`.
- **Staging to live orders** (do in order): (1) order-flow test on **demo/sandbox**
  (fake money, separate demo key); (2) production read-only [DONE]; (3) production
  **confirmation-gated, tiny** — I show ticker/side/size/price, you OK each, never
  silent. And only once the paper log actually shows an edge — right now it does not.

## Two legs (switchable any time)

Same pipeline; only recording/execution differs. Mode in
`~/.openclaw/workspace/hunt/poly-mode.txt` (`paper` default). `/poly paper` /
`/poly live` flip it. **State the active mode at the top of every run.**

- **`paper`** — simulate; record pick + simulated fill at the ask to
  `poly-paper-log.md`. The research leg: builds the track record that says whether
  the edge is real (calibration, hit-rate, P&L).
- **`live`** — real money, small, on **Kalshi**. Output the ranked Kelly-sized
  picks; you/mom **place them manually on Kalshi** and log to `poly-live-log.md`.
  No auto-trading in v1 (RSA-API automation only after it's wired + confirmation-gated).

## Usage

- **`/poly`** — full scan: Kalshi AI-edge candidates + cross-venue arb pairs →
  ranked sized shortlist, recorded per mode.
- **`/poly arb`** — cross-venue arbitrage only (Kalshi vs Polymarket).
- **`/poly <market-url | search terms>`** — deep-estimate ONE market.
- **`/poly paper` / `/poly live`** — switch leg. **`/poly log`** — summarize the
  ledgers (count, calibration, hit-rate, P&L, by mode).

## Execution (the pipeline)

1. **Discover** (public, no auth, no money):
   ```bash
   # Kalshi candidates (researchable, liquid, soon-closing):
   python3 ~/.openclaw/workspace/scripts/kalshi-scan.py --days 45 --min-volume 5000 --limit 40
   # Both venues together for cross-venue arb:
   python3 ~/.openclaw/workspace/scripts/poly-arb.py --days 45
   ```
   `kalshi-scan.py` walks Kalshi `/events` (the `/markets` feed leads with
   unpriced combos) and filters to priced binary, mid-price, researchable
   categories. `poly-arb.py` returns `{kalshi:[...], polymarket:[...]}`. For one
   market's live book: Kalshi `GET /markets/{ticker}/orderbook`, Polymarket
   `https://clob.polymarket.com/price?token_id=<id>&side=buy`.
2. **Cross-venue match (Claude does this — NOT a regex)**: read both lists, find
   pairs that are **genuinely the same question**, then **verify they resolve
   IDENTICALLY** — same threshold, same window, same source. Different windows
   ("leaves by Kalshi close" vs "out by June 30") are **NOT arb**. A real
   divergence after fees/spread = buy YES on the cheaper venue (/NO on the dearer).
3. **AI estimate** (for the non-arb leg, on less-efficient markets only): Claude
   reads resolution rules + **current evidence (web-search — the training cutoff
   is months stale, so fetch fresh facts)** and gives a calibrated P(YES) with a
   written rationale. Cross-check the material ones with ChatGPT
   (`ask-gpt.py <channel> "..."`, verify-don't-transcribe). If liquid+efficient,
   expect ~0 edge → skip.
4. **Edge + sizing** — `edge = AI_prob − ask` (or arb gap after fees). Require
   `|edge| ≥ EDGE_MIN`. Rank by `edge × liquidity × confidence`. Size with
   **fractional Kelly**, clamped to the caps below.
5. **Record / execute** — `paper`: append to `poly-paper-log.md` (simulated fill).
   `live`: present sized picks for **manual placement on Kalshi**; after placement
   append actual fills to `poly-live-log.md`. Never auto-place in v1.
6. **Measure** — every pick logged; `/poly log` computes calibration / hit-rate /
   P&L once markets resolve. Do not scale capital until the log proves an edge.

## Sizing & risk
- **Fractional Kelly** — `KELLY_FRACTION = 0.25`. Per-bet ≤ 2% of bankroll;
  total open exposure ≤ 20%. `EDGE_MIN = 0.07` (AI leg) / arb gap ≥ ~0.04 after
  fees. No illiquid / wide-spread / ambiguous-resolution markets; no un-sized bets.

## Logging
`~/.openclaw/workspace/hunt/poly-paper-log.md` and `poly-live-log.md`, one row per
pick (mirrors `/hunt`):
```
| date | venue | market | side | AI_prob/arb | conf | price | edge | stake | mode | resolved | outcome | pnl |
```
This ledger is the only honest answer to "is the edge real?".

## Reuse (don't reinvent)
- `~/.openclaw/workspace/scripts/kalshi-scan.py` (Kalshi discovery),
  `poly-scan.py` (Polymarket discovery), `poly-arb.py` (dual-fetch for cross-venue).
- **Kalshi API** `api.elections.kalshi.com/trade-api/v2` — data public; trading
  RSA-PSS-signed. **Polymarket** Gamma/CLOB/Data APIs (public) — read-only price
  source.
- `/chatgpt` (`ask-gpt.py`) — ChatGPT cross-check.
- FUTURE automation (Kalshi, after RSA-API wired + confirmation-gated): a Kalshi
  client lib or `pmxt` (multi-venue). Verify any repo before adopting.

## Do NOT
- No auto-trading in v1 — `live` is manual placement on Kalshi only.
- No trading Polymarket from the US (international = geoblocked/ToS); Polymarket
  is a data source only.
- No "arb" without confirming **both** markets resolve identically (threshold +
  window + source). Different windows are not arb.
- No edges from non-public info — insider-trading enforcement is real.
- No AI-probability bet on a liquid/efficient market (edge ≈ 0); no bet without a
  defensible, web-researched rationale; no un-cross-checked number on a material stake.
- No un-sized bets; never exceed per-bet / total caps.
- Never store wallet keys, RSA keys, KYC, or payment credentials in the repo/skill.
- Don't claim the edge works until the log proves it on a real sample.
