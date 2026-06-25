# Pine Script Indicators

## Trend-Filtered Structure & Volume Swing Indicator (TF-SVS)

`swing_trading_indicator.pine` — a Pine Script v5 `indicator()` for Daily/4H swing trading, built from the evidence review in this repo's research blueprint.

### Design

A long/short signal requires **all** of the following to align on a confirmed (closed) bar:

1. **Regime gate** — `close` above EMA200 and EMA50 > EMA200 (long; mirrored for short). On intraday charts, optionally also requires the same alignment on the last *closed* Daily bar (`Confirm with Daily EMA regime`), fetched via a single non-repainting `request.security` call.
2. **Trigger** — either a confirmed break of swing structure (BOS, requiring a candle **close** beyond the prior pivot, not just a wick), or a pullback into an active bullish/bearish Fair Value Gap or Order Block located in the discount/premium half of the most recent swing range.
3. **Confirmation** — relative volume (RVOL) at or above threshold (default 1.5x the 20-bar average) on the trigger bar, closing in the bias direction.
4. **Stop** — selectable: fixed ATR multiple, a ratcheting Chandelier trailing stop, or the tighter of ATR vs. the nearest FVG/OB zone boundary. Target defaults to a fixed R-multiple of the stop distance.

FVG and Order Blocks are used only as entry-location and stop-placement refinements — they never trigger a signal on their own, per the research review's finding that they lack peer-reviewed standalone validation.

### Inputs

Grouped in the indicator's settings panel: Trend Filter, Swing Structure, FVG / Order Blocks, Volume, Stops & Targets, Display. All zone/structure/marker drawing can be toggled independently without affecting the underlying signal logic.

### Alerts

Two `alertcondition()`s are exposed (`TF-SVS Long Entry`, `TF-SVS Short Entry`) for use with TradingView's alert system. This is an `indicator()`, not a `strategy()`, so it has no Strategy Tester backtest stats — it's intended for visual analysis and live alerting.

### Limitations

- Swing-range midpoint (discount/premium zone) uses the most recently confirmed pivot high and low independently, which may not be a strictly matched pair in choppy structure — treat it as an approximation.
- No backtest equity/drawdown stats are produced by an `indicator()`. A `strategy()` port sharing the same gating logic would be a natural follow-up if quantitative validation is needed.
- Verify in TradingView's Pine Editor: paste the script, check it compiles with no errors, and use Bar Replay to confirm signals only appear on closed bars (no repainting) before relying on it live.

## VWAP Deviation Scalping Indicator (VWAP-SCALP)

`vwap_scalping_indicator.pine` — a Pine Script v5 `indicator()` for 5-minute equities/crypto scalping, built from this repo's scalping evidence review.

### Design

A long/short signal requires **all** of the following to align on a confirmed (closed) bar:

1. **Trend gate** — higher-timeframe (default 1H, configurable) EMA50/EMA200 alignment, fetched via a single non-repainting `request.security` call on the last *closed* HTF bar.
2. **Trigger** — either a mean-reversion cross back inside a VWAP deviation band after tagging it (default 2x StdDev), or — in Trend Continuation / Both modes — a close back across VWAP with an RVOL spike, in the bias direction. Liquidity-sweep/SMC patterns and naive unfiltered momentum are intentionally excluded as standalone triggers: the evidence review found a naive 5-min momentum rule lost money (Sharpe -0.94, AAPL, 194 trades) without a real filter.
3. **Confirmation** — RVOL at or above threshold (default 1.2x the 20-bar average), optionally combined with an approximate Cumulative Volume Delta agreement filter (CVD here is a per-bar intrabar proxy, since true tick-level CVD isn't available via Pine).
4. **Cooldown** — a minimum bar gap (`ta.barssince`) between signals, so choppy 5-min conditions can't spam repeated entries off the same stretch.
5. **Stop/Target** — ATR-based stop (shorter length than the Daily/4H script) and a fixed R-multiple target.

A **Market Type** toggle (Equities/Crypto) widens the deviation bands for crypto (default +12.5%, since intrabar crypto volatility runs structurally higher) and, for equities only, can exclude the first/last N minutes of the regular session to avoid the open/close liquidity-surge distortion that doesn't apply to 24/7 crypto markets. Everything else — VWAP, EMA stack, volume confirmation, stop/target math — is one shared codebase across both market types.

### Inputs

Grouped in the indicator's settings panel: Trend Filter, VWAP, Market Type, Trigger, Volume, Cooldown, Stops & Targets, Display.

### Alerts

Two `alertcondition()`s are exposed (`VWAP-SCALP Long Entry`, `VWAP-SCALP Short Entry`) for TradingView's alert system. This is an `indicator()`, not a `strategy()` — visual analysis and live alerting only. Given 5-min execution latency sensitivity (alert fire + webhook delivery delay is a much larger fraction of an expected 5-min move than on Daily/4H), budget for that latency before automating off these alerts.

### Limitations

- CVD here is an **intrabar approximation**, not true tick-level cumulative volume delta — treat the optional CVD filter as a secondary nudge, not a precise order-flow read.
- Funding rate / open interest data is intentionally excluded — not reliably available via Pine's `request.security` for most exchanges, and it's a different strategy category (funding arbitrage, not directional scalping) anyway.
- No backtest equity/drawdown stats are produced by an `indicator()`. As with the swing script, a `strategy()` port sharing the same gating logic would be a natural follow-up if quantitative validation is needed — the evidence review's strongest caution (naive 5-min momentum producing Sharpe -0.94) makes backtesting this composite before any live automation especially important.
- Verify in TradingView's Pine Editor: paste the script on a 5-min equities and a 5-min crypto chart, confirm it compiles with no errors, and use Bar Replay to confirm signals only appear on closed bars (no repainting) before relying on it live.
