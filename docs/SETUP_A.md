# Setup A Specification v0.1

**Project:** Edge
**Setup:** A — Bearish trend continuation after a failed VWAP/resistance pullback
**Instrument:** MES (Micro E-mini S&P 500 futures)
**Version:** v0.1 (draft)
**Date:** 2026-10-29

> Edge detects and explains. The trader decides. No automatic entries or exits.
>
> Every rule below is split into three layers:
> - **Observe:** what is on the chart
> - **Interpret:** what it means
> - **Program:** how the detector measures it
>
> Thresholds in this document are hypotheses. Nothing is assumed profitable until enough labeled data exists.
> Changing a confirmed rule requires a stated reason and the trader's approval.

---

## Scope

| In scope | Out of scope (for now) |
|---|---|
| Setup A, bearish side | Setup A2 (micro double-rejection; a separate setup) |
| 1m decision chart with 5m + 1D trend context | Setup B (countertrend scalp) |
| Detection, scoring, alerts, exit *warnings* | Setup C (no-trade / exhausted continuation state) |
| | Bullish mirror of Setup A |
| | Any automated order entry or exit |

---

## 1. Confirmed rules

These were confirmed by the trader on 2026-10-29.

### 1.1 Charts and VWAP

| Item | Rule |
|---|---|
| Decision chart | 1-minute |
| Trend context | 5-minute and daily only (30-minute not used) |
| VWAP anchor | Session (resets at the CME futures session start, 3pm PT) |
| VWAP source | (H + L + C) / 3 |
| VWAP band | One band at 1 standard deviation |
| Chart session | Extended hours (ETH) |

### 1.2 Trend and alignment

- **Observe:** the trader draws a trend line through **two** swing highs on the 5m and daily charts and trades in that direction.
- **Interpret:** price below a down-sloping line through two lower highs = bearish trend.
- **Program:** Pine cannot read hand-drawn lines; the line is rebuilt from confirmed swing highs (method to be tested, see §3).

| 5m trend | 1D trend | Alignment | Detector action |
|---|---|---|---|
| Bearish | Bearish | Aligned | Normal Setup A alert |
| Bearish | Not bearish | Mixed | CAUTION alert |
| Not bearish | Bearish | Mixed | CAUTION alert |
| Bullish | Bullish | Against | No bearish alert |

Higher-timeframe values must come from **completed** 5m/1D candles only, so historical results do not "see the future."

### 1.3 Setup sequence

Setup A is a chain of events, in order:

1. 5m + 1D trend check is bearish or mixed.
2. Price pulls back up toward VWAP or resistance (the **test**).
3. Price does **not** meet the acceptance condition (§1.5).
4. A 1m trigger (T1–T5) fires at or very near the level.
5. Alert sent: normal (aligned) or CAUTION (mixed).
6. Trader decides: TAKE / WAIT / SKIP.

### 1.4 Entry triggers (1-minute)

- The trigger must occur **at or very near** VWAP/resistance.
- **A touch alone is not enough.** The candle must show price attempted to move through the level and failed.
- A tiny wick or a random red candle does not count.
- **Rejection** (the level held) and **trigger** (the event acted on) are measured separately.
- Each trigger is stored as its **own label** (T1–T5), never merged into a generic "rejection," so trigger performance can be compared later.

| ID | Trigger | Observe |
|---|---|---|
| T1 | Rejection candle | Meaningful upper wick into or through the level; closes back below it |
| T2 | Bearish engulfing | Red candle engulfs the last green pullback candle |
| T3 | Failed breakout | Trades or briefly closes above the level, fails the acceptance condition, then closes back below |
| T4 | Strong bearish candle after test | Price tests the level, fails to continue up, then a strong red candle forms |
| T5 | Lower high, then continuation | A smaller bounce fails below the test high, then price closes below the low between the two highs |

- **T3 is defined as the opposite of acceptance.** Same measurement, opposite outcome.
- **T5 vs. A2:** T5 waits for the breakdown close. A2 enters before it. That is what keeps them separate setups.

### 1.5 Acceptance (hard cancel)

- **Observe:** price pushes from below VWAP to above it, holds, and shows strength.
- **Interpret:** buyers accepted price above VWAP. The bearish thesis is dead.
- **Program:** acceptance **cancels Setup A outright**. It is not a score penalty.
- Acceptance = **multiple closes above VWAP AND meaningful distance above it** (Option B). Consecutive closes alone are not enough; two closes one tick above VWAP do not invalidate the thesis.

### 1.6 RSI

- RSI is **context only**: is the move stretched, or is there room to continue?
- RSI never enters or rejects a trade by itself.

### 1.7 Risk

| Trade type | Risk | Exit style | Adding |
|---|---|---|---|
| Aligned | Normal planned risk | Normal | Only within the original max-loss budget |
| Caution (mixed) | ~50% of normal planned risk | Quicker | **Not allowed** |

- Maximum risk is defined **before** entry and never raised after.
- **Caution risk is reduced with fewer contracts, keeping the same structurally valid stop.** The stop is never tightened just to reduce dollar risk. Reason: exposure goes down without moving the point where the thesis is invalidated.
- The 50% figure is adjustable later from collected data.

### 1.8 Caution-trade exit warning

- **Observe:** after entry, bearish volume fades and price stalls.
- **Interpret:** minimal remaining return for higher bounce risk; not worth staying.
- **Program:** Edge shows a "consider exit" warning. It never exits automatically.

---

## 2. Provisional thresholds (v0.1 candidates)

These are starting points to test, **not locked numbers**. Each will be compared against labeled historical MES setups.

### 2.1 Meaningful upper wick (Option A + minimum candle size)

| Component | Definition | v0.1 test values |
|---|---|---|
| Wick share | upper_wick / candle_range | 40% · 50% · 60% |
| Minimum candle size | candle_range / ATR(1m) | Recorded for every candidate; cutoff to be chosen from labels |
| Penetration | How far the high went past VWAP/resistance | **Feature only**, not a requirement |

- upper_wick = high − max(open, close)
- candle_range = high − low

### 2.2 Near VWAP (ATR fraction, ticks also recorded)

| Component | Definition | v0.1 test values |
|---|---|---|
| Primary distance | (VWAP − candle high) / ATR(1m) | Several cutoffs tested side by side (e.g., 0.25 · 0.50 · 0.75 ATR) |
| Secondary distance | (VWAP − candle high) in ticks (1 tick = 0.25 pt) | Recorded only; not a rule yet |

ATR is preferred because volatility changes through the session. Ticks are kept because MES trades in fixed ticks and both views will be compared.

### 2.3 Weak pullback volume (Option A: pullback vs. preceding bearish impulse)

| Field | Definition |
|---|---|
| avg_pullback_volume | Average volume of the pullback candles |
| avg_impulse_volume | Average volume of the bearish impulse that preceded the pullback |
| pullback_impulse_ratio | avg_pullback_volume / avg_impulse_volume |

- Primary definition. It matches the trader's reasoning better than a generic moving average.
- No cutoff chosen. The ratio that best matches the trader's discretionary reads will be found from labeled data.

### 2.4 Acceptance above VWAP (Option B: closes + distance)

| Component | v0.1 test values |
|---|---|
| Number of closes above VWAP | Variable (e.g., 2 · 3) |
| Minimum distance above VWAP | Variable, in ATR (e.g., 0.25 · 0.50) and recorded in ticks |

### 2.5 Failed breakout (T3)

- Price trades or briefly closes above the level, **does not meet the acceptance condition in §2.4**, then closes back below.
- Window to close back below: variable (e.g., within 1 · 2 · 3 candles).

---

## 3. Variables still requiring testing

| Variable | Why it is still open |
|---|---|
| Trend line method | Auto line through two swing highs vs. swing sequence vs. regression slope vs. manual input |
| Swing-high detection | How many candles on each side confirm a swing high (adds lag) |
| "Not bearish" / flat trend | How an unclear trend on 5m or 1D is detected |
| Impulse and pullback boundaries | Where the bearish impulse starts/ends and where the pullback starts; required for §2.3 |
| Resistance levels besides VWAP | Which ones the trader actually uses (prior swing low, prior high, session levels, profile) |
| Volume profile | Feasibility in Pine is unconfirmed |
| Trigger weights | Whether T1–T5 deserve equal weight in the score |
| T4 strength | Body/range and range/ATR cutoffs for a "strong" red candle |
| T5 structure | How small a swing counts as a lower high on 1m |
| Candidate expiry | How many candles a test stays valid without a trigger |
| Stop placement | Above rejection high + buffer vs. above pullback high vs. VWAP + ATR fraction |
| Target placement | Next support − buffer vs. fixed R multiple vs. prior impulse low |
| Caution exit warning | Volume-fade and stall definitions (no new low for N candles, shrinking range, overlap) |
| RSI usage | Length (default 14) and which context uses to keep |
| ATR length | Default 14 on 1m; to confirm |
| Session filter | All hours vs. cash session only (overnight volume is much thinner) |
| Caution risk % | ~50% for now; adjustable from data |

---

## 4. Data recorded for every candidate

Raw values are stored even when a threshold is not used yet, so definitions can be compared later.

| Group | Fields |
|---|---|
| Identity | event_id, timestamp_utc, timestamp_local, contract, timeframe, session |
| Provenance | event_type: real_trade / skipped / replay / observation (never mixed in statistics) |
| Trend | trend_5m, trend_1d, alignment (aligned / mixed / against) |
| Location | vwap, dist_vwap_atr, dist_vwap_ticks, penetration_ticks, level_type |
| Candle | candle_range, range_atr, upper_wick_pct, body_pct, close_location |
| Volume | avg_pullback_volume, avg_impulse_volume, pullback_impulse_ratio |
| Acceptance | closes_above_vwap, max_dist_above_vwap_atr, max_dist_above_vwap_ticks |
| Trigger | trigger_id (T1–T5; multiple allowed), trigger_bar_time |
| Context | rsi, atr |
| Decision | trader_label, decision (TAKE / WAIT / SKIP), reason |

---

## Change log

| Version | Date | Change |
|---|---|---|
| v0.1 | 2026-10-29 | Initial Setup A specification. Confirmed rules, provisional thresholds, open variables. |
