---
description: "Polymarket AI-edge hunter. /poly = scan liquid, clearly-resolving markets, estimate true probability with Claude (cross-checked via /chatgpt), flag mispricings, size with fractional Kelly, and record the pick. Two legs, switchable any time: PAPER (simulated, for measuring whether the AI edge is real) and LIVE (small real money, manual placement on the legal Polymarket US). Goal is money (mother's side business); honest EV — most traders lose, the AI edge is unproven until the log says otherwise. Trigger: /poly, /poly paper, /poly live, '看看 polymarket / 有没有错价 / 估个概率 / 下注 polymarket / poly 扫一遍'."
user-invocable: true
---

# /poly — Polymarket AI-edge hunter

Turn Claude + ChatGPT into a probability engine over Polymarket. The thesis:
humans can't beat the bots on **speed** (sub-3s arbitrage windows), but a careful
**information/logic edge** — read the evidence, estimate a calibrated P(YES),
bet only when the market price is clearly off — is the one corner left to us.
This is **unproven**: ~84% of Polymarket traders lose money. So the skill is
built to **measure the edge before scaling capital**, not to assume it.

Money runs as the **mother's side business** (KYC/funding under her name — same
arrangement as `/hunt`). The skill never touches KYC, wallet keys, or payment
credentials.

## Two legs (switchable any time)

The pick pipeline is identical; only how a pick is recorded/executed differs.

- **`paper`** (default) — simulate. Record the pick + a simulated fill at the
  current ask to `poly-paper-log.md`. Zero money, zero legal exposure. This is
  the research leg: it accumulates the track record that tells us if the AI edge
  is real (calibration, hit-rate, simulated P&L).
- **`live`** — real money, small. The skill outputs the same ranked, Kelly-sized
  picks; you (or mom) **place them manually on Polymarket US** and log them to
  `poly-live-log.md`. Automated execution stays OFF until the Polymarket US
  trading-API question is resolved (see Legal & accounts).

Mode lives in `~/.openclaw/workspace/hunt/poly-mode.txt` (contents: `paper` or
`live`; absent = `paper`). `/poly paper` and `/poly live` flip it. **Always state
the active mode at the top of every run.** Run both legs in parallel: keep paper
logging everything for calibration even while live bets a subset.

## Legal & accounts (read once)

- **Polymarket US** (CFTC-regulated since Dec 2025) is the **legal** venue for a
  US person: KYC (gov ID, SSN), **USD settlement via bank** (debit/ACH/wire).
  This is where `live` real money goes. State-law challenges are still in flux
  (NV/AZ/CT/IL/OH/MD) — note it, not a blocker.
- **International Polymarket** (crypto, USDC-on-Polygon) is **geoblocked for US
  IPs**. Do **not** VPN into it — ToS violation, account-closure + forfeited
  protections. We never trade there.
- **Public DATA APIs work regardless of geo** — discovery/analysis is always fine.
- **Funding**: Polymarket never takes BTC directly. On Polymarket US you fund via
  bank (USD); BTC as a source means sell → USD → bank (taxable event; keep the
  money trail under mom clean). Account/funding is set up manually, outside the skill.
- **Open item**: whether Polymarket US exposes a programmatic trading API is
  unconfirmed. Verify before ever wiring `py-clob-client` for auto-execution; until
  then, `live` = manual placement only.

## Usage

- **`/poly`** — scan → rank → present a shortlist of mispriced markets with
  AI_prob, market price, edge, confidence, and a Kelly-sized stake. Record per mode.
- **`/poly <market-url | search terms>`** — deep-estimate ONE market (full
  research + ChatGPT cross-check + sizing).
- **`/poly paper` / `/poly live`** — switch the active leg.
- **`/poly log`** — summarize the ledgers: count, calibration (predicted vs
  realized), hit-rate, P&L, by mode.

## Execution (the pipeline)

1. **Discover** (public, no auth, no money):
   ```bash
   python3 ~/.openclaw/workspace/scripts/poly-scan.py \
     --min-liquidity 20000 --min-volume 100000 --days 45 \
     --min-price 0.08 --max-price 0.92 --limit 40
   ```
   Filters to binary Yes/No, order-book-enabled, liquid, mid-priced (edge room),
   soon-resolving markets and emits JSON (id, question, yes_price, bid/ask,
   spread, liquidity, end date, yes_token_id, resolution `description`, url).
   Prefer **researchable** markets (politics, econ, geopolitics, dated events);
   skip pure-noise/longshot sports favorites and anything whose resolution rule is
   ambiguous. For one market, fetch live book/price from the CLOB API:
   `https://clob.polymarket.com/price?token_id=<yes_token_id>&side=buy`.
2. **Estimate true P(YES)** — for each candidate worth it, Claude reads the
   resolution rules + current evidence and gives a **calibrated** number with a
   written rationale and the key evidence cited. On the hardest / largest-stake
   ones, **cross-check with ChatGPT** (verify-don't-transcribe):
   ```bash
   python3 ~/.openclaw/workspace/scripts/ask-gpt.py <channel> "Estimate P(YES) for: <question>. Resolution rules: <...>. Give a number 0-1 + 1-paragraph reasoning + the 2-3 facts that move it."
   ```
   If Claude and ChatGPT disagree materially, that IS the signal — dig or skip;
   never bet on a number you can't defend.
3. **Edge + sizing** — `edge = AI_prob − ask` (buying YES) or `bid − AI_prob`
   (buying NO). Require `|edge| ≥ EDGE_MIN` *after* the spread. Rank by
   `edge × liquidity × confidence`. Size with **fractional Kelly**:
   `stake = KELLY_FRACTION × edge / (1 − price)` of bankroll, clamped to the
   per-bet and total caps below. Drop anything where confidence is low or the
   resolution rule is exploitable against you.
4. **Record / execute** (per mode):
   - `paper`: append the pick to `poly-paper-log.md` with a simulated fill at the
     ask. No execution.
   - `live`: present the ranked sized picks for **manual placement on Polymarket
     US**; after Xiang/mom places them, append to `poly-live-log.md` with the
     actual fill. Never auto-place.
5. **Measure** — every pick is logged so that, once markets resolve, `/poly log`
   computes calibration (does AI_prob=0.7 win ~70% of the time?), hit-rate, and
   P&L. **This is the point.** Do not scale capital until the paper log shows a
   real, calibrated edge over a meaningful sample.

## Sizing & risk

- **Fractional Kelly only** — start `KELLY_FRACTION = 0.25` (quarter-Kelly).
- **Per-bet cap**: ≤ 2% of bankroll. **Total open exposure cap**: ≤ 20%.
- **EDGE_MIN** = 0.07 after spread to start (the market is efficient; small
  apparent edges are usually noise or you misread the resolution rule).
- Never bet markets you can't price with a defensible rationale, illiquid books
  (slippage eats the edge), or wide spreads. Conservative first; loosen only when
  `poly-log` proves the edge.

## Logging

`~/.openclaw/workspace/hunt/poly-paper-log.md` and `poly-live-log.md`, one row per
pick (mirrors the `/hunt` log discipline):

```
| date | market (slug) | side | AI_prob | conf | price | edge | stake | mode | resolved | outcome | pnl |
```

Back-fill `resolved/outcome/pnl` as markets settle. This ledger is the calibration
record and the only honest answer to "is the AI edge real?".

## Reuse (don't reinvent)
- `~/.openclaw/workspace/scripts/poly-scan.py` — discovery (this skill ships it).
- **Gamma API** `gamma-api.polymarket.com` (market data) + **CLOB API**
  `clob.polymarket.com` (`/price`, `/book`, `/price_history`) + **Data API**
  `data-api.polymarket.com` (positions, leaderboards) — all public, no auth.
- `/chatgpt` (`ask-gpt.py`) — the ChatGPT half of the probability cross-check.
- For the FUTURE automation leg (only after the US-API question is settled, and
  only confirmation-gated): official `py-clob-client` (github.com/Polymarket/
  py-clob-client) and `pmxt` (multi-venue). For backtesting/calibration: the
  `poly_data` / `prediction-market-analysis` datasets and `polymarket-paper-trader`.
  Verify any repo is real + maintained before adopting; most "polymarket bot"
  repos are abandoned toys or spam.

## Do NOT
- No auto-trading in v1 — `live` is manual placement on Polymarket US only.
- No trading the US-geoblocked international platform (VPN = ToS violation).
- No bet without a written, defensible P(YES) rationale; no acting on an
  un-cross-checked AI number on a material stake.
- No illiquid / wide-spread / ambiguous-resolution markets.
- No un-sized bets; never exceed the per-bet / total caps.
- Never store wallet keys, KYC data, or payment credentials in the repo or skill.
- Don't claim the AI edge works until `poly-log` proves it on a real sample.
