# Autonomous AI Trading Agent — Advanced Architecture
*For developers and technically curious readers who want to understand what's under the hood*

---

## The Problem With Prompt-Only Control

The initial version of this agent was entirely prompt-driven. The rules — position limits, cash reserves, drawdown thresholds, universe restrictions — lived in CLAUDE.md as instructions for Claude to follow.

A software engineer reviewing the project correctly identified the gap:

> "The control layer is prompt-driven, not code-enforced. I would want a separate deterministic validator between Claude and Robinhood before I'd consider this production-ready."

He's right. Prompt-driven controls are flexible but brittle. The same market state and prompt can yield different outputs across runs due to LLM non-determinism. More practically, Claude might:

- Make an arithmetic error calculating a post-trade position weight
- Pull a slightly stale price and size a trade based on outdated data
- Interpret an ambiguous rule differently than intended
- Confabulate a value in a complex multi-step calculation

None of these are the AI "going rogue" — they're the kind of errors any analyst might make under time pressure. The difference is that a human analyst's output gets reviewed before execution. Without a validator, Claude's output goes straight to Robinhood.

**v2.3 addresses this with a hybrid architecture:** Claude handles reasoning, thesis evaluation, and trade proposal generation. A deterministic Python validator enforces the rules in code before any order reaches Robinhood MCP.

---

## v2.3 Architecture Overview

```
Session start
    │
    ├── Load state.json (persistent account state)
    ├── Pull live data via Robinhood MCP
    │
    ├── Claude reasons through strategy
    │   (thesis check, positions vs targets, earnings rule,
    │    drawdown tier, proposed trades)
    │
    ├── Claude outputs proposals.json (structured JSON)
    │
    ├── validator.py runs (deterministic Python)
    │   ├── PASS → execute trades via Robinhood MCP
    │   └── FAIL → log violations, notify user, halt
    │
    ├── Post-execution: update state.json
    ├── Write session log to GitHub
    └── Push notification to phone
```

**Key principle:** Claude proposes. The validator enforces. Robinhood executes.

---

## The Three Files

### validator.py

A standalone Python script (~200 lines) that enforces all trading rules in code. It runs after Claude generates proposed trades and before any order is placed.

**What it checks:**
- Account is the correct Agentic account (not Income, Growth, or Grok)
- Market is open (9:30 AM – 4:00 PM ET, weekdays, non-holiday)
- Drawdown tier (see below)
- Daily trade count does not exceed maximum
- Total session cash deployment within cap
- Each proposed trade is within approved universe
- Each trade meets minimum size
- Large trades flag for confirmation
- NVDA trim within session cap
- Post-trade position weights within max allocations
- Cash reserve stays above minimum after all trades

**How to install:**
```bash
# Copy validator.py to your trading-agent folder
# Test with a dry run (no state updates)
python3 validator.py --proposals proposals.json --state state.json --dry-run
```

**Requirements:** Python 3.9+ (uses `zoneinfo` for timezone handling)

**Return codes:**
- Exit 0: PASS — safe to execute
- Exit 1: FAIL — do not trade, violations logged

---

### state.json

A persistent JSON file that carries account state between sessions. Without this, the agent can't accurately track the high-water mark for drawdown calculations or daily trade counts across multiple runs.

**Structure:**
```json
{
  "account_number": "YOUR_ROBINHOOD_AGENTIC_ACCOUNT_NUMBER",
  "account_value": 0,
  "buying_power": 0,
  "positions": {
    "NVDA": {"value": 0, "pct": 0},
    "AVGO": {"value": 0, "pct": 0}
  },
  "high_water_mark": 0,
  "trades_today": 0,
  "last_trade_date": null,
  "last_updated": null,
  "schema_version": "1.0",
  "build_phase": "Day 1",
  "notes": "Initialize after first session. Agent updates automatically."
}
```

**Key fields:**
- `high_water_mark` — highest account value ever recorded. Used to calculate drawdown percentage. Never decreases.
- `trades_today` — resets automatically when `last_trade_date` rolls to a new calendar day.
- `positions` — stores both dollar value and percentage for each holding. Updated after every session.
- `last_updated` — ET timestamp of the last state write. Used for auditing.
- `schema_version` — tracks the state file format version for future compatibility.

**In the cloud Routine:** state.json lives in the private GitHub repo. The Routine fetches it at session start, injects live MCP data, runs validation, then pushes the updated file back to GitHub after execution. This gives you a version-controlled history of account state.

---

### proposals.json

A temporary file written by Claude at the end of its reasoning step, before any execution. Contains all proposed trades in structured JSON format.

**Format:**
```json
{
  "proposals": [
    {
      "symbol": "CEG",
      "action": "BUY",
      "amount": 350,
      "rationale": "Day 3 Tier 2 top-up; below 5% target; 5-day move +2.6% clears chase rule"
    },
    {
      "symbol": "NVDA",
      "action": "SELL",
      "amount": 500,
      "rationale": "Trim from 34.4% toward 25% max. Session cap $500."
    }
  ]
}
```

**Why structured output matters:** Requiring Claude to output proposals as JSON before execution forces it to commit to a specific, parseable set of actions. The validator reads this file — not Claude's conversational reasoning — so ambiguous language can't slip through. If Claude's JSON doesn't validate, nothing executes.

---

## Tiered Drawdown Logic

The original strategy had a single -20% threshold: if the account dropped 20% from its high-water mark, all trading stopped. v2.3 replaces this with three graduated tiers:

| Tier | Threshold | Response |
|------|-----------|----------|
| Warning | -10% from HWM | Max session deployment reduced to 25% of available cash |
| Pause buys | -15% from HWM | No new BUY orders. Trims and rebalancing sells only |
| Full stop | -20% from HWM | All trading halted. Manual reset required before resuming |

**Why tiered?** A single hard stop at -20% is too binary. A 10% drawdown in a volatile sector doesn't warrant the same response as a 20% drawdown. The graduated approach reduces exposure progressively rather than flipping from "full speed" to "complete stop" in one step.

**Implementation:** The validator enforces all three tiers in code. Claude also checks drawdown in its reasoning (step 5 of the run procedure), but the validator is the hard gate — if Claude misses the check or miscalculates, the validator catches it.

---

## What's Still Missing

This is an honest accounting of known gaps. v2.3 is significantly more robust than the initial version but is not production-grade for large capital.

**Earnings automation:**
The current setup requires Claude to manually check earnings dates via MCP tools. This works but is not reliable. An automated earnings calendar lookup (via a financial data API) would be more consistent.

**Tax lot awareness:**
No tax optimization logic. The agent doesn't consider whether selling a position triggers short-term vs long-term capital gains. High turnover strategies can generate significant tax drag that erodes gross returns.

**Volume and liquidity filters:**
Tier 3 and 4 names can have relatively low average daily volume. No filter currently prevents the agent from sizing into a low-liquidity name during a thin session. A minimum ADV check (e.g., $50M average daily volume) would reduce slippage risk.

**Exact share-count weight calculations:**
The validator uses dollar values to approximate post-trade weights. For fractional share positions, exact weight calculations would require pulling current share prices at validation time — an additional MCP call per validation run.

**Backtesting:**
The strategy has not been backtested against historical data. Live performance since June 2026 is the only evidence base.

**2027 holiday calendar:**
Market holidays are hardcoded for 2026. Update `MARKET_HOLIDAYS_2026` in validator.py in December for the following year.

---

## For Developers

The architecture is intentionally simple and hackable. A few extension points:

**Adding a new validation rule:**
Add a check inside the `validate()` function in validator.py. Return `violations.append("descriptive message")` on failure. The function returns all violations at once — don't use early returns inside the loop.

**Extending state.json:**
Add any persistent field to the JSON. Claude reads and writes the full file each session, so new fields are preserved automatically as long as they don't conflict with existing keys.

**Adding a new tier to the universe:**
Update the `UNIVERSE` set and `MAX_ALLOCS` dict in validator.py. Update the same in CLAUDE.md. Both must stay in sync — the validator enforces what CLAUDE.md describes.

**Connecting a different broker:**
The validator is broker-agnostic. It validates proposals and returns PASS/FAIL. Replace the Robinhood MCP execution step with any broker's API and the validator works unchanged.

**Running tests:**
```bash
# Test with a known-good proposals file
python3 validator.py --proposals proposals.json --state state.json --dry-run

# Test a rule violation (e.g., edit proposals.json to include a non-universe ticker)
# Should return FAIL with specific violation listed
```

---

## The SE's Critique — Where Things Stand

The original critique: "The control layer is prompt-driven, not code-enforced. I would want a separate deterministic validator between Claude and Robinhood before I'd consider this production-ready."

**v2.3 response:**
- Deterministic validator implemented and running in production
- Hard-coded rules in Python, not prompts
- JSON proposal format forces structured output before execution
- State persistence eliminates manual tracking errors
- Tiered drawdown replaces binary threshold

**What remains prompt-driven:**
- Thesis evaluation and stock scoring
- Earnings rule application
- Universe opportunity scanning
- Monthly performance narrative

These are judgment calls that benefit from Claude's reasoning. The validator catches errors in the mechanical execution — it doesn't (and shouldn't) second-guess the investment reasoning.

**Honest assessment:** This is a robust personal trading system for small-to-medium capital. For institutional use or significantly larger capital, additional layers would be warranted: exact weight calculations, earnings calendar automation, tax-aware lot selection, and formal backtesting.

---

*Core strategy and architecture: github.com/frogman263/trading-agent-showcase*
*Validator source: validator.py in the same repo*
