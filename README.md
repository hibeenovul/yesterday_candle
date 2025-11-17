Objective
Trade breakouts relative to yesterday’s candle body (not wicks) on the Daily timeframe. Enter long only when the market is above yesterday’s body high; enter short only when the market is below yesterday’s body low. Directional bias must be confirmed by two higher timeframes (HTF1 and HTF2), each using a Fast vs. Slow moving average filter. Include full exit logic: stop loss, take profit, and two-stage trailing stop (two activation distances → two different trailing distances/steps).

1) Core Definitions
    • Yesterday’s Body High (YBH): the larger of yesterday’s daily Open and Close.
    • Yesterday’s Body Low (YBL): the smaller of yesterday’s daily Open and Close.
    • Signal TF: the chart or chosen “execution timeframe” that evaluates bar closes and triggers entries.
    • HTF Filters: Two configurable higher timeframes (HTF1 and HTF2). On each HTF:

        ◦ Long allowed only if Fast MA > Slow MA on the last closed bar of that HTF.

        ◦ Short allowed only if Fast MA < Slow MA on the last closed bar of that HTF.

    • No repainting rule: All decisions must be based on closed bars on the respective timeframes.


2) Entry Logic (Market Execution, no stop orders)
    1. Daily gate: On each new trading day (new D1 bar):

        ◦ Compute YBH and YBL once and cache them for the day.

        ◦ Reset “one-per-day” controls (if that option is enabled).

    2. Directional permission (HTFs):

        ◦ Calculate Fast and Slow MAs on HTF1 and HTF2 (user-chosen MA type and periods).

        ◦ Longs permitted only if both HTFs have Fast > Slow on their most recent closed bars.

        ◦ Shorts permitted only if both HTFs have Fast < Slow on their most recent closed bars.

        ◦ Option: a setting to choose “both must agree” (default) vs. “either HTF may allow”.

    3. Body breakout condition (on Signal TF):

        ◦ Long trigger: First time the Signal TF closed bar finishes strictly above YBH for the day.

        ◦ Short trigger: First time the Signal TF closed bar finishes strictly below YBL for the day.

        ◦ Optional buffer (points) can be added to YBH/YBL comparisons, if desired (default 0).

    4. Re-entry policy (configurable):

        ◦ Default: One breakout per direction per day (once a long is taken above YBH, no more longs that day; same for shorts below YBL).

        ◦ Option to allow re-entries only if the market fully returns to the opposite side of the body (closes back below YBH for long side or above YBL for short side) and then re-breaks again; still capped to a user-defined Max Trades per Day.

    5. Session filter (optional):

        ◦ If enabled, only take entries between StartHour and EndHour (broker time).

    6. Spread/volatility filters (optional but recommended):

        ◦ Max allowable spread (points).

        ◦ Min and/or Max ATR filter on Daily or Signal TF to avoid dead or extreme regimes.


3) Exit Logic
3.1 Stop Loss (SL)
    • Two modes:

        ◦ ATR-based: SL distance = (ATR on D1 with configurable period) × (user multiplier).

        ◦ Fixed Points: a user-defined points value.

    • SL is placed immediately upon entry.

    • If the broker enforces a minimum stop distance, ensure the SL meets that requirement (adjust outward if necessary or reject the trade with a clear log).

3.2 Take Profit (TP)
    • Two modes:

        ◦ R-multiple: TP distance = (SL distance) × (user R multiple, e.g., 1.5R, 2.0R).

        ◦ ATR-based: TP distance = (ATR on D1) × (user multiplier).

    • TP is placed immediately upon entry.

3.3 Break-Even (optional stage)
    • If enabled: at BE Activate Distance (points in profit), move SL to Entry ± BE Offset (offset ≥ spread for longs/shorts respectively).

3.4 Two-Stage Trailing Stop
    • Stage 1:

        ◦ Activation distance: when price moves Activate1 points in favor from entry.

        ◦ Trail distance: maintain SL at TrailDist1 points behind current price (bid for buys, ask for sells).

        ◦ Trail step (hysteresis): modify SL only if price has advanced at least TrailStep1 points since the last SL update (reduces modification spam).

    • Stage 2:

        ◦ Activation distance: when price moves Activate2 points in favor (≥ Stage 1 activation).

        ◦ Trail distance: maintain SL at TrailDist2 points behind price.

        ◦ Trail step: TrailStep2 (same purpose as above).

    • Stage precedence: Once Stage 2 is active, it overrides Stage 1. If price retreats, do not downgrade from Stage 2 to Stage 1.


4) Order/Position Management
    • Execution type: Market orders only (no stop/limit orders).

    • One position at a time per symbol (netting and hedging accounts should be treated as single-position logic; if a position exists, skip new entries).

    • OCO behavior is implicit (since entries are market and one-side-per-day is enforced).

    • Daily reset: On a new D1 bar, recompute YBH/YBL. Do not retroactively affect open trades.


5) Inputs (Settings UI)
5.1 General
    • Symbol: default current chart symbol.

    • Signal Timeframe: default chart timeframe.

    • Magic Number / Slippage / Comments toggles.

    • Use Session Filter (bool), StartHour, EndHour.

    • Max Trades per Day: default 1 long + 1 short (or 1 total), selectable.

    • Allow Re-entries After Full Re-cross (bool).

    • Body Breakout Buffer (points): default 0.

5.2 HTF Filters (both editable)
    • HTF1 Timeframe (e.g., H1).

    • HTF2 Timeframe (e.g., H4).

    • MA Type (select per-HTF or shared: SMA/EMA/SMMA/LWMA), default EMA.

    • Fast Period and Slow Period (editable, per-HTF or shared).

    • Agreement Mode: “Both must agree” (default) or “Either HTF may allow”.

    • Use Closed Bar Only: enforced (no intrabar values).

5.3 Risk Management
    • Risk Mode: Fixed Lots or Risk % of balance.

    • Fixed Lots value.

    • Risk % value (if using risk-based sizing).

    • SL Mode: ATR×Mult or Fixed Points.

        ◦ ATR Period (D1) and ATR Multiplier.

        ◦ Fixed SL (points).

    • TP Mode: R-Multiple or ATR×Mult.

        ◦ R Multiple (e.g., 2.0R).

        ◦ ATR Multiplier (if chosen).

5.4 Trailing & BE
    • Use Break-Even (bool), BE Activate Distance (pts), BE Offset (pts).

    • Use Trailing (bool).

    • Stage 1: Activate1 (pts), TrailDist1 (pts), TrailStep1 (pts).

    • Stage 2: Activate2 (pts), TrailDist2 (pts), TrailStep2 (pts).

5.5 Filters (optional)
    • Max Spread (points).

    • ATR Regime Filter: Min ATR, Max ATR thresholds.