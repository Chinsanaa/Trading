# Trend-Filtered Structure & Volume Swing Indicator (TF-SVS)

`swing_trading_indicator.pine` — a Pine Script v5 `indicator()` for Daily/4H swing trading, built from the evidence review in this repo's research blueprint.

## Design

A long/short signal requires **all** of the following to align on a confirmed (closed) bar:

1. **Regime gate** — `close` above EMA200 and EMA50 > EMA200 (long; mirrored for short). On intraday charts, optionally also requires the same alignment on the last *closed* Daily bar (`Confirm with Daily EMA regime`), fetched via a single non-repainting `request.security` call.
2. **Trigger** — either a confirmed break of swing structure (BOS, requiring a candle **close** beyond the prior pivot, not just a wick), or a pullback into an active bullish/bearish Fair Value Gap or Order Block located in the discount/premium half of the most recent swing range.
3. **Confirmation** — relative volume (RVOL) at or above threshold (default 1.5x the 20-bar average) on the trigger bar, closing in the bias direction.
4. **Stop** — selectable: fixed ATR multiple, a ratcheting Chandelier trailing stop, or the tighter of ATR vs. the nearest FVG/OB zone boundary. Target defaults to a fixed R-multiple of the stop distance.

FVG and Order Blocks are used only as entry-location and stop-placement refinements — they never trigger a signal on their own, per the research review's finding that they lack peer-reviewed standalone validation.

## Inputs

Grouped in the indicator's settings panel: Trend Filter, Swing Structure, FVG / Order Blocks, Volume, Stops & Targets, Display. All zone/structure/marker drawing can be toggled independently without affecting the underlying signal logic.

## Alerts

Two `alertcondition()`s are exposed (`TF-SVS Long Entry`, `TF-SVS Short Entry`) for use with TradingView's alert system. This is an `indicator()`, not a `strategy()`, so it has no Strategy Tester backtest stats — it's intended for visual analysis and live alerting.

## Limitations

- Swing-range midpoint (discount/premium zone) uses the most recently confirmed pivot high and low independently, which may not be a strictly matched pair in choppy structure — treat it as an approximation.
- No backtest equity/drawdown stats are produced by an `indicator()`. A `strategy()` port sharing the same gating logic would be a natural follow-up if quantitative validation is needed.
- Verify in TradingView's Pine Editor: paste the script, check it compiles with no errors, and use Bar Replay to confirm signals only appear on closed bars (no repainting) before relying on it live.
