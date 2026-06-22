# Autonomous AI Trading Agent

**Claude Code + Robinhood MCP + Claude Routines**

An autonomous trading agent built on Anthropic's Claude Code, connected to Robinhood via the Model Context Protocol (MCP). Executes a daily AI infrastructure investment strategy across 14 positions, runs on a cloud schedule, and logs every session to this repo.

---

## Architecture

```
Claude Code Routine (cloud, 11 AM ET daily)
    │
    ├── Robinhood MCP ──────────► Place trades, pull quotes, check positions
    ├── Strategy (CLAUDE.md) ───► Rules, universe, risk controls
    ├── GitHub REST API ────────► Write session logs to this repo
    └── Push Notification ──────► Summary to phone after each run
```

**Stack:**
- [Claude Code](https://claude.ai/code) — Anthropic's agentic coding environment
- [Robinhood Agentic Trading](https://robinhood.com/us/en/support/articles/agentic-trading/) — live trade execution via MCP
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io) — tool connectivity layer
- GitHub REST API — session log persistence
- Claude Routines — cloud scheduling (no server required)

---

## Investment Thesis

MAG8 hyperscaler CAPEX is a structural, multi-year tailwind for AI chips, memory, power infrastructure, and the data center companies building the physical layer of AI. The portfolio owns the companies cashing those checks across the full stack — from silicon to power to compute infrastructure.

---

## Universe (18 stocks, 4 tiers)

| Tier | Focus | Holdings |
|------|-------|---------|
| Tier 1 | AI Chips & Memory | NVDA, AVGO, MU |
| Tier 2 | Power Infrastructure | CEG, GEV, VST, BE |
| Tier 3 | AI Data Center Infrastructure | IREN, APLD, CORZ, CRWV |
| Tier 4 | Supporting & Opportunistic | ASML, NBIS, RIOT + AMD/AMAT/MRVL/VRT |

---

## How It Works

Every weekday at 11 AM ET the agent wakes up automatically and:

1. Checks if market is open (handles holidays automatically)
2. Pulls live account state and quotes for all 18 universe stocks
3. Checks drawdown — pauses all trading if down 20% from high-water mark
4. Evaluates each position against target allocations
5. Applies the Earnings Rule (opportunity-based, not blackout-based)
6. Calculates rebalancing trades
7. Executes market orders via Robinhood MCP
8. Runs a thesis review every Monday (hyperscaler CAPEX signals, universe scan)
9. Generates a monthly performance report on the first Monday of each month
10. Writes session log to this repo
11. Sends push notification summary to phone

---

## Key Strategy Rules

**Entry:** Buy when position is ≥2% below target weight, cash reserve ≥5%, no earnings chase (≤10% 5-day run for new positions)

**Earnings:** Treated as opportunities, not blackouts. Buy pullbacks into earnings; don't chase runups ≥10% in prior 5 sessions.

**Risk controls:** -20% drawdown pause, max 10 trades/day, max 50% cash deployment per session, equities only, no margin.

**Thesis review:** Every Monday, agent evaluates hyperscaler CAPEX signals (Microsoft, Amazon, Google, Meta each rated Bullish/Neutral/Concern), checks each position's thesis integrity, and scans for new universe candidates — all flagged for human review, never acted on unilaterally.

**Macro red flags** (pause new buying if triggered):
- 2+ hyperscalers cut CAPEX same quarter
- NVDA guidance miss >10% on AI demand
- Fed aggressive rate hikes (>2 in one quarter)
- Credible competitor gains >10% NVDA market share

---

## Session Logs

All session logs are written automatically to the [`logs/`](./logs/) folder after each daily run. Each log includes trades executed, positions vs targets, earnings watch, and flagged items for review.

---

## Strategy File

The full strategy is defined in [`CLAUDE.md`](./CLAUDE.md) — the file Claude Code reads automatically on every run. It contains the complete universe, allocation targets, entry/exit rules, earnings logic, rebalancing triggers, and risk controls.

---

## Built With

- No prior coding experience
- Claude (Anthropic) for strategy development, architecture, and agent logic
- One afternoon

---

## Disclaimer

This is a personal project for educational and portfolio demonstration purposes. Nothing here constitutes financial advice. Real money is involved — trade at your own risk.
