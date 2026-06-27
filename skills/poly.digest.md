# poly skill — DIGEST (auto-generated, do not edit)
# Full version: ~/repos/zinan-skills/skills/poly.md


## Venues (why Kalshi-first)
- **Kalshi** — the **legal** US venue: CFTC-regulated exchange, KYC, USD/bank,
  Key: Polymarket | US-geoblocked | only as a price-comparison data source | real public+trading API | trading uses RSA-PSS-signed requests | ⚠️ Insider-trading enforcement is active

## Live auth (Kalshi) — status & setup
- **Connected, read-only, verified 2026-06-20** on **production** ($100 balance,
  Key: Credentials live in `~/.kalshi/` | NOT wired | Staging to live orders | confirmation-gated, tiny | demo/sandbox

## Two legs (switchable any time)
Same pipeline; only recording/execution differs. Mode in
  Key: State the active mode at the top of every run. | place them manually on Kalshi

## Usage
- **`/poly`** — full scan: Kalshi AI-edge candidates + cross-venue arb pairs →
  Key: `/poly <market-url | search terms>` | `/poly arb` | `/poly log` | `/poly paper` / `/poly live`

## Execution (the pipeline)
1. **Discover** (public, no auth, no money):
  1. **Discover**
  2. **Cross-venue match (Claude does this — NOT a regex)**
  3. **AI estimate**
  4. **Edge + sizing**
  5. **Record / execute**
  6. **Measure**

## Sizing & risk
- **Fractional Kelly** — `KELLY_FRACTION = 0.25`. Per-bet ≤ 2% of bankroll;

## Logging
`~/.openclaw/workspace/hunt/poly-paper-log.md` and `poly-live-log.md`, one row per

## Reuse (don't reinvent)
- `~/.openclaw/workspace/scripts/kalshi-scan.py` (Kalshi discovery),
  Key: Kalshi API | Polymarket

## Do NOT
- No auto-trading in v1 — `live` is manual placement on Kalshi only.
