# Edge

**Semi-systematic decision support for discretionary futures trading.**

Edge watches the chart for setups that resemble how I already trade, scores them with transparent rules, explains why they qualify, and alerts me. **I make the final decision.** Edge never places, changes, or exits trades.

> Status: research phase. Setup A is specified (v0.1); no code yet. Nothing here is a claim of profitability or financial advice.

---

## Why

Discretionary trading works on judgment that is hard to measure. Edge turns that judgment into explicit, testable rules so that:

- setups are found consistently, not only when I happen to be watching
- every alert explains its evidence (trend, VWAP, volume, price action, RSI, reward-to-risk)
- decisions are logged, so the rules can be checked against real data instead of memory

**Working pipeline:** trend / structure → location → participation → rejection or acceptance → reward vs. risk → human decision.

## Principles

- **Human in the loop.** Alerts ask for TAKE / WAIT / SKIP. No automated entries or exits.
- **No single-indicator signals.** RSI, a candle shape, or volume alone never triggers a trade.
- **Observe → interpret → program.** Every rule is written in three layers: what is on the chart, what it means, and how code measures it.
- **Risk first.** Maximum risk is defined before entry and never raised after.
- **Clean data.** Real trades, skipped setups, replay trades, and observations are stored separately and never mixed in statistics.
- **Hypotheses, not truths.** Thresholds stay provisional until labeled data supports them.

## Current scope: Setup A

**Bearish trend continuation after a failed pullback to VWAP or resistance** on MES (Micro E-mini S&P 500 futures).

- 1-minute decision chart; 5-minute and daily charts for trend
- Session VWAP (HLC3) with a 1-standard-deviation band
- Five separately labeled 1m triggers (T1–T5)
- Acceptance above VWAP cancels the setup outright
- Mixed timeframe alignment → CAUTION alert: ~50% risk via fewer contracts, same structural stop, quicker exit, no adding

Full specification: [`docs/SETUP_A.md`](docs/SETUP_A.md)

## Roadmap

| Version | Goal |
|---|---|
| **V1** | TradingView Pine Script detector: Setup A detection, score, chart labels, alerts |
| **V2** | Alerts → Python / FastAPI webhook → Supabase |
| **V3** | React dashboard to review alerts and record decisions |
| **V4** | Statistical analysis and scoring improvements |
| **V5** | Machine learning, only after enough clean labeled data exists |

**Next step:** hand-label 15–20 historical 1m MES pullbacks and compare the v0.1 threshold candidates against those labels before writing any Pine code.

## Repository layout (planned)

```
edge/
├── README.md
├── docs/
│   ├── SETUP_A.md          # Setup A specification
│   └── RESEARCH_LOG.md     # findings from labeling and testing
├── pine/                   # V1 TradingView detector
├── backend/                # V2 FastAPI webhook receiver
├── supabase/               # V2 schema and migrations
├── frontend/               # V3 React dashboard
└── analysis/               # V4 notebooks and reports
```

## Disclaimer

Edge is a personal research and software project. It is not financial advice, and it makes no claim that any strategy described here is profitable.
