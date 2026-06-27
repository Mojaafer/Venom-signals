# ICT Venom Model 2025 — TradingView Indicator

A Pine Script v5 indicator that automates **Michael Huddleston's (Inner Circle Trader) 2025 Venom Model** and prints **BUY / SELL signals** complete with **entry price, stop-loss and two take-profit targets** directly on the chart.

> ⚠️ **Disclaimer**: This is an educational tool, not financial advice. ICT concepts are discretionary in nature — no indicator can perfectly replicate them. Always back-test and forward-test on a demo account before risking real capital.

---

## What is the Venom Model?

The Venom Model is ICT's 2025 intraday strategy, originally designed for **US index futures (NQ, ES, YM)**. It is a time-and-price version of the AMD / Power-of-3 cycle:

| Phase | ICT Name | What happens |
|-------|----------|--------------|
| 1 | **Accumulation** | A 90-minute window (default `08:00 → 09:30 NY`) builds a range. |
| 2 | **Manipulation** | After the regular open (`09:30 NY`) price **sweeps** the range low (bullish day) or range high (bearish day), grabbing liquidity and leaving an FVG. |
| 3 | **Distribution** | Price reverses sharply, leaving an opposing FVG → together they form a **BPR (Balanced Price Range)**. An **MSS / CISD** confirms the reversal. Enter on the retracement into the BPR / FVG (PD array). |

*Targets:* opposite extreme of the 90-min range (TP1), then 2R or prior-day extreme (TP2).
*Stop:* 10–20 ticks beyond the swept extreme.

---

## Features

- ✅ **Automatic 90-minute range detection** (box drawn on chart) with selectable NY timezone.
- ✅ **Liquidity-sweep detection** after the regular open (marked on chart).
- ✅ **FVG + BPR recognition** (sweep-leg FVG and reversal FVG drawn as boxes).
- ✅ **MSS & CISD confirmation** (toggleable — ICT recommends waiting for these).
- ✅ **BUY / SELL labels** with the full trade plan printed in one label:
  `Entry • SL • TP1 • TP2`.
- ✅ **Stop-loss** = swept extreme ± 10–20 ticks (configurable).
- ✅ **TP1** = opposite range extreme · **TP2** = fixed R:R (default 2R) or prior-day H/L.
- ✅ **Live status panel** (top-right) showing current state, range H/L, sweep direction and confirmation ticks.
- ✅ **Alert conditions** for BUY and SELL (works with TV alerts / webhooks).
- ✅ Supports the **alternate Venom windows**: `01:30–03:00`, `12:00–13:30` NY.

---

## Installation

1. Open [TradingView](https://www.tradingview.com/) and open any chart.
2. At the bottom of the screen click **Pine Editor**.
3. Open the file [`ict_venom_model.pine`](./ict_venom_model.pine) from this repo and **copy its entire contents** into the Pine Editor.
4. Click **Save**, give it a name, then click **Add to chart**.
5. The indicator appears on the chart with the settings panel shown below.

> Recommended chart: **1-minute or 3-minute** timeframe on **NQ1! / ES1! / YM1!** (US index futures). Other instruments and timeframes work but may need tuning.

---

## Settings (Inputs)

### ① Time Window (90-min Accumulation)
| Input | Default | Description |
|-------|---------|-------------|
| Accumulation window (NY local) | `0800-0930` | Session string for the 90-min range. Alternates: `0130-0300`, `1200-1330`. |
| Timezone | `America/New_York` | Timezone the session strings refer to. |
| Trade window (after open) | `0930-2400` | Signals only fire inside this session. |

### ② Sweep & Confirmation
| Input | Default | Description |
|-------|---------|-------------|
| Entry on BPR only | `false` | Fire as soon as the reversal FVG (BPR) forms — earlier but riskier. |
| Require MSS confirmation | `true` | Wait for a Market Structure Shift. |
| Require CISD confirmation | `true` | Wait for an engulfing reversal candle. |
| Minimum FVG size (points) | `0` | Filter out noise gaps (e.g. `2` for NQ). |
| Max bars to wait for confirmation | `40` | Invalidate the setup if no confirmation in time. |

### ③ Stop Loss & Targets
| Input | Default | Description |
|-------|---------|-------------|
| Stop-loss buffer (ticks) | `20` | ICT recommends 10–20 ticks beyond the swept extreme. |
| Tick size in points | `0` | `0` = auto from `syminfo.mintick`. NQ=0.25, ES=0.25, YM=1.0. |
| TP1 = opposite range extreme | `true` | Standard ICT first target. |
| Use fixed R:R for TP2 | `true` | If on, TP2 = entry ± `rrMult` × risk. |
| TP2 R:R multiple | `2.0` | Venom scalp target is commonly 2R. |
| TP2 = prior-day opposite extreme | `false` | Alternative TP2 = prev-day high/low. |

### ④ Visuals
Toggle the range box, sweep marker, FVG/BPR boxes, Entry/SL/TP lines, and colors.

### ⑤ Alerts
Enable alert conditions for the BUY / SELL signals.

---

## How to read the signals

When a signal fires you will see:

- A **`BUY ▲`** or **`SELL ▼`** label on the bar.
- A multi-line label showing **Entry / SL / TP1 / TP2** prices.
- Four horizontal lines extending to the right:
  - **Solid green/red** = Entry
  - **Dashed red** = Stop loss
  - **Dotted green/red (thin)** = TP1
  - **Dotted green/red (thick)** = TP2
- The status panel (top-right) shows the live state of the setup as it builds.

### Trade management
- **Enter** with a limit order at the Entry price (middle of the BPR / reversal FVG) when price retraces into it.
- **Stop** goes at the SL line.
- **Take profit**: scale out at TP1, runner to TP2 — or close all at TP1 for a mechanical scalp.

---

## Venom trade flow the indicator follows

```
08:00–09:30 NY  →  build range (state: Building range)
09:30 open      →  wait for sweep of range low/high (state: Awaiting sweep)
Sweep + FVG     →  track reversal FVG + MSS + CISD (state: Swept — confirming)
All confirm     →  BUY / SELL signal + Entry/SL/TP1/TP2 (state: Done)
```

If no confirmation arrives within `maxBars` the setup is invalidated for the day and the indicator waits for the next 90-minute window.

---

## Back-testing tips

1. Use TradingView's **Strategy Tester** by converting `indicator(...)` to `strategy(...)` if you want stats, or use the **Replay** bar-by-bar tool with this indicator overlaid.
2. Start on **NQ1! 1-min** with default settings — that is the cleanest Venom market.
3. Tune `fvgMinSize` and `maxBars` per instrument: forex pairs need smaller FVG filters, index futures larger ones.
4. Always check higher-timeframe bias before taking a signal — ICT emphasises alignment with the daily/4H narrative.

---

## Files

| File | Description |
|------|-------------|
| [`ict_venom_model.pine`](./ict_venom_model.pine) | The TradingView Pine Script v5 indicator. |
| [`README.md`](./README.md) | This documentation. |

---

## References

- ICT Venom Trading Model 2025 — [innercircletrader.net](https://innercircletrader.net/tutorials/ict-venom-trading-model-2025/)
- ICT's 2025 Venom Model — [FX Replay](https://fxreplay.com/strategies/icts-2025-venom-model)
- Original ICT lectures — [YouTube @innercircletrader](https://www.youtube.com/innercircletrader)

This indicator is an independent implementation of publicly documented ICT concepts and is not affiliated with or endorsed by Michael Huddleston.
