# Autonomous AI Trading Agent — Setup Guide
*Step-by-step from zero to first autonomous trade*

---

## Before You Start

Read the Overview document first. Come back here when you have:
- [ ] A Robinhood Gold account funded and active
- [ ] A Claude Pro subscription ($20/month at claude.ai)
- [ ] A rough investment thesis in mind
- [ ] Starting capital set aside for the Agentic account ($1,000 minimum, $5,000–$10,000 recommended)

Estimated time to complete this guide: **4–8 hours over 1–2 sessions**

---

## Phase 1 — Robinhood Setup

### Step 1: Enable Agentic Trading in Robinhood

1. Open the Robinhood app on your phone
2. Navigate to **Investing** → **Agentic** (or search "Agentic" in the app)
3. Follow the on-screen prompts to create a dedicated **Agentic account**
4. This creates a separate, sandboxed account — the AI can only trade here, not in your other accounts
5. **Fund the account** — transfer your starting capital in. The AI can only use money you explicitly put here

> Your Agentic account will show up separately from your other Robinhood accounts. Keep it that way. Never move your entire portfolio here.

---

## Phase 2 — Claude Setup

### Step 2: Download Claude Desktop

1. Go to **claude.ai/download** in your browser
2. Download and install the app for your operating system (Mac or Windows)
3. Sign in with your Claude Pro account

### Step 3: Connect Robinhood to Claude

**In the Claude Desktop app:**

1. Go to **Settings → Connectors** (or **Customize → Connectors**)
2. Find **Robinhood** in the list and click **Connect**
3. Follow the OAuth flow — it will open a browser window and ask you to authorize Claude's access to your Robinhood account
4. Complete the authorization on your phone in the Robinhood app when prompted
5. Return to Claude Desktop — Robinhood should now show as **Connected**
6. Go to **claude.ai/customize/connectors** in your browser and click on **Robinhood**
7. Under **Tool permissions**, click **Always allow** at the top right to enable all tools — they are off by default and the agent cannot trade without them enabled
8. Confirm all 34 tools show as enabled

**Verify the connection works:**

In Claude (chat tab), type:
```
Check my Robinhood account balance
```
If it returns your account information, the connection is live.

---

## Phase 3 — Build Your Strategy

This is the most important phase. Use **High effort + Thinking ON** throughout.

### Step 4: Develop Your Thesis with Claude

Open a new chat in Claude and start the conversation. Don't overthink the opener — just tell Claude what you're thinking:

> *"I want to build an autonomous trading strategy around [your thesis]. Help me develop it into a complete strategy I can give to an AI agent to execute."*

Claude will ask you questions and help you build out:
- Which specific stocks fit your thesis (your universe)
- How to organize them by conviction level (tiers)
- Target allocation percentages for each position
- Entry rules — when to buy and when to add
- Exit and trim rules — when to reduce a position
- Risk limits — maximum position sizes, cash reserves
- A drawdown pause threshold — what loss percentage triggers a full stop
- Earnings rules — how to handle earnings events
- Macro red flags — conditions that pause new buying

**This is a conversation, not a form to fill out.** Push back, ask questions, stress-test the ideas. The time you invest here determines how well the agent performs.

> **Tip:** Ask Claude to play devil's advocate — "What's the bear case for this thesis?" and "What would make you wrong about this?" are some of the most valuable questions you can ask.

---

## Phase 4 — Create the Strategy File

### Step 5: Install Claude Code CLI

Claude Code is the command-line tool that will run your agent. You need to install it once.

**On Mac — open Terminal and run:**
```
curl -fsSL https://claude.ai/install.sh | bash
```

**On Windows — open PowerShell and run:**
```
winget install Anthropic.ClaudeCode
```

After installation completes, close and reopen your terminal, then verify it worked:
```
claude --version
```
You should see a version number. If you get "command not found," run this and try again:

*Mac:*
```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

*Windows:* Restart PowerShell.

### Step 6: Connect Robinhood to Claude Code

In Terminal (Mac) or PowerShell (Windows), run:
```
claude mcp add robinhood-trading --transport http https://agent.robinhood.com/mcp/trading
```

You should see a confirmation that the MCP server was added.

### Step 7: Create Your Project Folder

**Mac:**
```
mkdir ~/trading-agent
cd ~/trading-agent && claude
```

**Windows:**
```
mkdir %USERPROFILE%\trading-agent
cd %USERPROFILE%\trading-agent && claude
```

This opens Claude Code in your project folder.

### Step 8: Create Your CLAUDE.md Strategy File

The CLAUDE.md file is the brain of your agent. Claude Code reads it automatically on every run. It contains everything from Phase 3 — your universe, rules, risk limits, and red flags — written in a format the agent can execute.

**In the Claude Code terminal, type:**
```
Based on the strategy we discussed, create a CLAUDE.md file 
in this folder with my complete trading strategy. Include my 
universe with tier structure, target allocations, entry/exit 
rules, earnings rules, risk controls, drawdown pause threshold, 
macro red flags, and a run procedure for the agent to follow 
each session.
```

Claude Code will generate the file. Review it carefully — this is your strategy in executable form. Make sure it reflects what you actually want.

> **Tip:** If anything looks wrong or missing, tell Claude Code to fix it before moving on. This file drives every trade the agent will make.

---

## Phase 5 — Test Before You Trade

### Step 9: Run a Supervised Test

Before letting the agent trade autonomously, run it manually and watch what it does.

**In Claude Code, type:**
```
Review my Agentic Robinhood account per the strategy in 
CLAUDE.md. Tell me what trades you would make today and why, 
but do not execute anything yet.
```

Review the output carefully:
- Does it understand your universe correctly?
- Are the position sizing calculations right?
- Is it applying your rules the way you intended?
- Does anything look off?

Adjust the CLAUDE.md if needed and re-run until the reasoning looks sound.

### Step 10: Execute Your First Trades

When you're satisfied with the test output, authorize the first session:

```
Execute the strategy per CLAUDE.md for my Agentic Robinhood account.
```

Claude Code will run through the checklist, pull live quotes, calculate trade sizes, and place orders. You'll see each step in real time and receive trade notifications in the Robinhood app.

> **Recommendation:** Don't deploy 100% of your capital on Day 1. A staged build over 3–5 sessions reduces timing risk and gives you a chance to catch anything unexpected before you're fully deployed.

---

## Phase 6 — Automate It

### Step 11: Set Up the Daily Routine

Once you've confirmed the agent is executing correctly, you can automate the daily run so it fires without you opening anything.

**In Claude Code, click Routines in the sidebar (or type `/routines`), then:**

1. Select **Create**
2. When asked what the agent should do, paste something like:
```
Read CLAUDE.md in my trading-agent folder and execute the 
daily trading review for my Robinhood Agentic account. 
Follow all rules in CLAUDE.md exactly.
```
3. Set the schedule — **11:00 AM ET daily, weekdays only** is recommended (90 minutes after market open gives time for price discovery)
4. Confirm Robinhood is listed under MCP Connections
5. Save the Routine

The agent will now run automatically every market day. You'll receive a Pushover notification on your phone after each session summarizing what happened — including validator status, trades executed, account value, and any flagged items.

> **Note:** The Routine runs in Anthropic's cloud — your computer does not need to be on.

---

## Phase 7 — Ongoing Management

### What to Do Daily
- Check your Pushover notification after 11 AM ET
- Review any flagged items the agent raised for your attention
- No action needed if everything ran normally

### What to Do Weekly
- Read the Monday thesis review in your session output
- Confirm the agent's reasoning still aligns with your intent
- Make any strategy adjustments in CLAUDE.md and update the Routine if needed

### What to Do Monthly
- Review overall performance against your thesis
- Evaluate whether any positions should be added or removed from your universe
- Adjust risk limits if your account size has changed significantly

### When to Intervene Manually
- The agent flags something for your review
- A macro red flag triggers and the agent pauses
- You want to make a position change outside the normal rebalancing cycle
- The agent hits the drawdown pause threshold

---

## Key Reminders

- The AI only has access to your dedicated Agentic account — your other accounts are read-only
- You are responsible for all trades the agent places — Robinhood does not supervise the AI
- You can disconnect the agent from Robinhood instantly at any time from the app
- Real-time trade notifications from Robinhood are your first line of visibility
- The strategy is only as good as the thesis and rules you put into it

---

*For the investment thesis and strategy framework: github.com/frogman263/trading-agent-showcase*
*For account setup questions: robinhood.com/us/en/support/agentic-trading*
