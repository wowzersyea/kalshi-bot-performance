# Kalshi NFL bot: performance feed

Machine-readable performance record for an automated betting strategy on Kalshi NFL markets.
The bot updates this repository on its own after any run that changes the numbers. Every
commit is a timestamped snapshot, so the git history doubles as an audit trail.

**Feed URL** (stable, no auth):

```
https://raw.githubusercontent.com/wowzersyea/kalshi-bot-performance/main/performance.json
```

Schema id: `lazyedge-bot.performance/v1`

## Strategy

- **Venue:** Kalshi (CFTC-regulated event contracts). Markets: NFL moneyline and the main spread market of each game.
- **Signal:** LazyEdge model probabilities labelled `PRICE_GAP`, where the model's probability clears Kalshi's ask on a tight quote.
- **Entry rules:**
  - Edge after Kalshi's fee must be at least `strategy.rules.min_edge_after_fee_pp` probability points, re-checked against the live Kalshi ask.
  - Entries happen 5 to 120 minutes before kickoff, and only if the price hasn't moved 5¢ or more.
  - At most one moneyline and one spread bet per game.
- **Sizing:** flat stake of about $1 per bet, in whole contracts. Orders are limit orders at the ask, immediate-or-cancel.
- **Risk limits:** weekly caps, a cash reserve and a hard loss stop. `status` is `Halted` when the bot is stopped.

## How the numbers are computed

- **P&L per bet:** taken from Kalshi's own settlement record (`/portfolio/settlements`) as revenue minus cost basis minus fees. Each bet's `pnl_source` says whether it came from `kalshi_settlement` or, before Kalshi settles, from the bot's ledger (`bot_ledger`).
- **`roi_on_stake_pct`:** P&L divided by the money staked (cost including fees).
- **`return_on_capital_pct`:** P&L as a percent of the account balance when the live bot started. The dollar balance itself isn't published.
- **`max_drawdown_pct`:** peak-to-trough drop in account equity (starting capital plus cumulative settled P&L), in percent.
- **Period rollups:** `daily`, `weekly` (ISO week), `monthly` and `nfl_weeks`, all keyed by NFL game day in US Eastern time. A Sunday-night game counts on Sunday.
- **`clv_cents`:** closing-line value, the last Kalshi ask before kickoff minus the entry price.
- **Open positions:** a bet appears only after its game has kicked off, so the feed never shows a position that could still be copied.
- **Verification:** each bet carries `kalshi_order_id` for reconciliation through Kalshi's API with a read-only key.

## Top-level fields

`schema, generated_at, strategy, status, capital, summary, daily, weekly, monthly, nfl_weeks, bets, verification`
