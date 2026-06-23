# Autonomous AI Trading Agent — Overview
*A plain-language intro for guys who want to get started*

---

## What Is This?

An autonomous AI trading agent is a setup where an AI model — in this case Claude by Anthropic — connects directly to your Robinhood brokerage account and executes trades on your behalf based on a strategy you define. It runs on a schedule, analyzes your portfolio, makes decisions, executes orders, and reports back to you. No babysitting required once it's running.

This is not a robo-advisor. You define the thesis, the rules, and the risk limits. The AI executes them. You stay in control.

---

## What You Need

### Accounts (all online signups)

| Tool | Cost | Notes |
|------|------|-------|
| Robinhood Gold | $5/month | Brokerage account — Gold tier required for agentic trading |
| Claude (Anthropic) | $20/month (Pro) | Powers the AI agent — Pro tier required |

**Total monthly cost: ~$25/month**

### Software to Download (all free)

- **Claude Desktop** — the app where you set up and manage the agent
  - Mac: claude.ai/download
  - Windows: claude.ai/download (same link, detects your OS)
- **Terminal (Mac) / Command Prompt or PowerShell (Windows)** — already built into your computer. Used for a handful of one-time setup commands — copy/paste only, no coding required.

That's it. GitHub, VS Code, and other developer tools are optional add-ons for logging and file management — not required to get the agent trading.

---

## Tech Knowledge Required

Be honest with yourself on these. You don't need to be a developer but you need to be comfortable with:

**Basic (must have):**
- File system navigation — knowing where files live on your computer, moving and renaming files
- Copy/paste, downloading software, navigating settings menus
- Comfortable having detailed conversations with AI — being specific and clear in your prompts
- Reading and following step-by-step instructions without skipping steps

**Helpful but learnable as you go:**
- Terminal/Command Prompt basics — you'll run a handful of commands during setup, all copy/paste
- Basic understanding of how brokerage accounts work

**Not required:**
- Coding or programming
- Software development experience
- Networking or server knowledge

---

## How It Works (Big Picture)

1. You define the strategy — what to buy, when, and why
2. Claude reads your strategy file on every run
3. Claude connects to Robinhood via MCP (a secure API bridge)
4. Claude analyzes your portfolio, pulls live quotes, and checks rules
5. Claude executes trades within your defined guardrails
6. Claude sends a Pushover notification summary to your phone
7. Repeat daily — automatically, no input needed

---

## Costs and Starting Capital

**Monthly overhead:** ~$25 (Robinhood Gold + Claude Pro)

**Starting capital:** $1,000 minimum, $5,000–$10,000 recommended
- $1,000 is a workable floor for testing the system with real trades
- $5,000+ gives enough room to build meaningful positions and read performance clearly
- The agent builds positions gradually over the first week, not all at once

**No ongoing fees** beyond the $25/month. No percentage of assets, no per-trade commissions on Robinhood.

---

## Developing Your Investment Strategy

This is the most important part — but you don't have to figure it all out yourself. You bring the thesis; Claude helps you build everything else around it.

**What you need to bring:**

- **A thesis** — a specific, reasoned belief about why certain sectors or companies will perform well. Not "I think tech is good." Something more like: "AI infrastructure spending by major cloud companies is a multi-year structural tailwind that benefits chip makers, memory suppliers, and power companies directly."

That's it. One clear idea you actually believe in. Claude takes it from there.

**What Claude helps you build:**

Once you have a thesis, Claude will work through the rest of the strategy with you in conversation — asking the right questions, offering options, and helping you think through things you might not have considered:

- **Universe** — Claude will suggest specific stocks that fit your thesis, explain why each qualifies, and help you organize them by conviction level or category
- **Position sizing and allocation targets** — how much of your portfolio goes into each tier or individual position
- **Entry and exit rules** — when to buy, when to add, when to trim. Claude translates your general intent into specific, executable rules
- **Risk limits** — drawdown thresholds, maximum position sizes, cash reserves. Claude will propose sensible defaults based on your account size and risk tolerance
- **Macro red flags** — conditions that would pause new buying without triggering a full sell-off. Claude helps you define these in advance so the agent isn't flying blind if market conditions shift
- **Earnings rules** — how the agent should treat earnings events. Blackout period, buy the dip, chase the move — Claude explains the tradeoffs and helps you decide

The strategy isn't dictated top-down. It's built through a back-and-forth conversation where you make the calls and Claude fills in the details, challenges your assumptions, and flags things you might have missed.

**Tips for developing your thesis:**

- Start with what you actually understand. Military/defense, logistics, energy, healthcare, real estate — whatever sector you've operated in or followed closely
- Don't overthink it upfront. Come in with a rough idea and let Claude help you sharpen it
- Ask Claude to stress-test it: "What's the bear case for this thesis? What could break it?"
- Keep it simple enough to explain in two sentences. If you can't, it's too complex to automate
- Be specific about your risk tolerance early — it shapes every rule Claude will suggest

**If you want a starting point:**
The full strategy used to build this system — including the thesis, universe, rules, risk limits, and red flags — is publicly documented at github.com/frogman263/trading-agent-showcase. Use it as a reference, adapt it to your own thesis, or build something entirely different.

---

## When to Use High Effort vs Low Effort

Claude has adjustable reasoning settings — effort levels (Low, Medium, High, Max) and an optional "Extended Thinking" toggle. Knowing when to use each saves time and gets better results.

**Use High or Max effort + Thinking ON for:**
- Building your investment thesis and stress-testing it
- Developing strategy rules, position sizing, and risk limits
- Deciding which stocks belong in your universe and why
- Adding or removing tickers after a major market event
- Reviewing the strategy after a macro red flag triggers
- Setting up your CLAUDE.md strategy file for the first time
- Any decision involving real money you won't be able to easily reverse

These are consequential decisions. The extra reasoning time is worth it.

**Use Medium effort for:**
- General strategy questions and refinements
- Reviewing session logs and making minor adjustments
- Asking Claude to explain a flagged position or earnings event
- Most day-to-day conversations about the portfolio

**Low effort is fine for:**
- Quick account balance and position checks
- Simple lookups ("what's my buying power?", "what's MU's earnings date?")
- Routine status checks when you already know what you're looking for

**For the automated daily Routine:**
The agent runs on Claude Sonnet 4.6 by default — the right balance of speed and intelligence for executing a defined strategy. It makes multiple tool calls per session (pulling quotes, checking positions, placing orders) and needs to be fast. You don't need Opus or Max for daily execution. Higher-end models are for building and refining the strategy, not running it.

**Rule of thumb:** The higher the stakes and the more open-ended the question, the higher the effort setting. The more routine and well-defined the task, the lower the effort needed.

---
## Safety and Risk

- The AI can only trade in a dedicated Agentic account — it has no access to your other Robinhood accounts
- You set the rules; the AI follows them
- You can disconnect the agent from Robinhood at any time instantly
- Robinhood sends real-time trade notifications to your phone for every order placed
- You define a drawdown limit — if the account drops by a set percentage, the agent pauses automatically and waits for your review

**This is real money. The AI will make real trades. Set your risk limits before you fund the account, not after.**

---

## What This Is Not

- Not a guaranteed profit system
- Not a replacement for financial advice
- Not fully hands-off forever — thesis and rules need periodic human review as markets change
- Not something to fund with money you can't afford to lose

---

## Time Commitment

**Initial setup:** 4–8 hours spread over 1–2 days
**Ongoing:** 15–30 minutes/week reviewing session outputs
**Strategy review:** 1–2 hours/month

The whole point is that day-to-day execution is fully autonomous. Your time goes into the strategy design upfront and periodic reviews — not monitoring trades.

---

*For a step-by-step setup walkthrough, see the companion document.*
*For the full strategy and architecture used in this build: github.com/frogman263/trading-agent-showcase*
