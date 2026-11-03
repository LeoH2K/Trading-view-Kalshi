# Kalshi BTC 15m: Price to Beat & Fair Odds (TradingView indicator)

A Pine Script v6 indicator for Kalshi's **15-minute Bitcoin up/down markets** (series `KXBTC15M`).
It shows the price BTC has to beat, how much time is left, and a fair probability (and so a fair
price in cents) that the market settles **UP**. It also shows the highest YES/NO price still worth
paying after Kalshi's fee.

File: [`kalshi_btc_15m.pine`](kalshi_btc_15m.pine)

> This is an analysis tool, not financial advice. Prediction markets are risky. The indicator
> estimates Kalshi's settlement index; it does not read it.

---

## How these markets settle (and why it matters)

| | |
|---|---|
| **Window** | 15 minutes, aligned to :00, :15, :30, :45 ET |
| **Target ("price to beat")** | Simple average of the 60 one-second **CF Benchmarks BRTI** values in the minute **before** the window opens |
| **Settlement** | Simple average of the 60 one-second BRTI values in the **last minute** of the window |
| **YES (UP) wins if** | settlement ≥ target |

This follows Kalshi's crypto contract terms as publicly described. If Kalshi changes them, the
market's rules page wins, and the window length and averaging period are adjustable in the
settings.

Three consequences the indicator builds in:

1. **Each window's settlement is the next window's target.** It is the same minute of data.
2. **The final minute is an average, not a single print.** With 20 seconds left, two-thirds of the
   settlement is already locked in. A late spike only moves the result by its share of the
   remaining seconds. Most "last-second flips" come from this.
3. **BRTI is a USD index built from several exchanges' order books.** One exchange's price, and
   especially a USDT pair, can sit a few dollars away from it. That is enough to change the call
   when the price is near the target.

## Install

1. Open a **BTC/USD** chart on TradingView. `COINBASE:BTCUSD` is recommended.
2. Set the chart to **1 minute**. If your plan has seconds charts, 15s or 5s works too; any
   timeframe from 1s to 15m that divides evenly into 15 minutes is supported.
3. Open the **Pine Editor** (bottom panel), create a new indicator, delete the template, and paste
   in the full contents of `kalshi_btc_15m.pine`.
4. Click **Save**, then **Add to chart**.

It opens a small probability pane under the chart and draws the target lines and dashboard on the
price chart itself.

## What you see

### Price chart
- **Gold line**: the current window's price to beat, from the window's open to its close. A label at
  the right edge shows the close time (ET) and the target.
- **Thin green/red lines**: past windows' targets, colored by how they settled. Hover a **▲/▼**
  marker for that window's target, estimated settlement, the model's call at minute 10, and the
  momentum lean at the open.
- **Gold shading**: the averaging minutes. The last minute of every window is both that window's
  settlement period and the next window's target period.
- **Dashed/dotted cone**: the expected ±1σ / ±2σ range of the settlement average from now until the
  close. It narrows as time runs out and flattens in the final minute as the average locks in.
- **Dashboard** (top right): see below.

### Probability pane
- **P(UP) line** (0–100): green above 50, red below. This is the fair YES price in cents.
- **Dashed lines** at 65 and 35: the "UP favored" / "DOWN favored" thresholds (adjustable).
- **Columns** around 50: the momentum bias (up = bullish, down = bearish).

### Dashboard rows

| Row | Meaning | How to use it |
|---|---|---|
| Time left | Countdown to the window close | |
| Price to beat | Estimated target (the tooltip says whether it came from live ticks or bars) | **Compare with the target Kalshi shows.** If they differ by more than a few dollars, factor that in. |
| BTC now | Current price of the BRTI proxy | |
| Distance | Price minus target, in $ and in σ of the move still possible | ±1σ is about 84/16 odds; ±2σ about 98/2 |
| Locked in / Rest must avg | *Final minute only.* Average so far, and the average the remaining seconds need for YES | The most useful rows in the last 60 seconds |
| P(UP) | Model probability of UP | |
| Fair YES / NO | The same, as contract prices | |
| Value buy ≤ | Highest price that still leaves your required edge (default 3¢) after the fee | Only buy if Kalshi's **ask** is at or below this |
| Signal | UP/DOWN favored past your thresholds, otherwise "Toss-up" plus the momentum lean | |
| Momentum | Trend-following bias from −1 to +1 | Check its hit rate before trusting it |
| Settle est. ±1σ | Median expected settlement and its 1σ range | |
| Volatility | Typical 1σ move over a full window | Higher vol means odds move more slowly |
| Last N windows | Share of recent windows that settled UP | |
| Momentum@open hit | How often the momentum lean at the open called the window correctly | ~50% means no edge. Even a random walk shows a few points above 50%, because at the open the price often already sits slightly above or below the target. Look for a clear margin over 100+ calls. |
| Calib @10m | Model's average confidence at minute 10 vs. how often it was right | If **real > pred**, the model is too cautious: lower *Volatility multiplier*. If **real < pred**, raise it. |

## A simple way to use it

1. **At the open**, P(UP) starts near 50%. Fair value can't call the direction yet; only the
   momentum lean can, so check its hit rate first. Many traders wait.
2. **During the window**, compare Kalshi's ask with **Value buy ≤**. The edge usually comes from
   the market overreacting or lagging the math, not from predicting BTC.
3. **In the final minute**, watch **Locked in** and **Rest must avg**. If BTC needs a move the
   remaining seconds can't realistically make, the outcome is close to decided even when the
   price looks near the target.
4. **Around news** (CPI, FOMC, big liquidations), realized volatility lags reality. Treat the odds
   as overconfident until the volatility estimate catches up.

## Alerts

Right-click the indicator → **Add alert**, then pick one of:
- **P(UP) crossed above the UP threshold** / **below the DOWN threshold**
- **New window opened**
- **Any alert() function call**: detailed messages with the window, odds, time left, BTC price and
  target, e.g. `Kalshi BTC 15m · 12:15–12:30 · P(UP) 71.2% · 6:40 left · BTC $… vs target $…`

## Settings

| Group | Setting | Default | Notes |
|---|---|---|---|
| Kalshi market | Window length / averaging period | 15 min / 60 s | Matches KXBTC15M |
| | Time zone for labels | America/New_York | Kalshi uses ET |
| Price source | Price source | USD spot composite | Average of the exchanges below; or *Chart symbol* |
| | Exchanges 1–4 | Coinbase, Kraken, Bitstamp, Gemini (BTCUSD) | Untick or replace any; the *BTC now* tooltip shows how many are live |
| | Refine averages with live ticks | On | Time-weights every tick inside averaging minutes while the chart is open |
| Probability model | Volatility lookback | 60 min | EWMA of 1-minute returns |
| | Volatility multiplier | 1.0 | Tune using the calibration row |
| | Fat tails (Student-t ν) | 6 | 0 = normal curve; lower = more room for late reversals |
| | Momentum tilt | 0 | 0 = pure fair value. >0 shifts the odds toward the momentum bias |
| Trade thresholds | UP / DOWN favored | 65% / 35% | Also used by alerts |
| | Kalshi fee coefficient | 0.07 | Fee ≈ coefficient × P × (1 − P) per contract; check Kalshi's current fee schedule |
| | Required edge | 3¢ | Margin kept in *Value buy ≤* |
| Momentum bias | EMA, momentum, VWAP, buy/sell-pressure lookbacks | 3/12, 5, 60, 10 min | Lean threshold 0.25 |
| Display | Dashboard, compact mode, position, lines, markers, cone, shading | | |
| Self-check stats | Windows to evaluate / checkpoint | 96 (24 h) / 10 min | |

## The model, briefly

- **Price**: an equal-weight average of USD spot exchanges approximates BRTI.
- **Volatility**: EWMA of squared 1-minute log returns, in basis points per √minute.
- **Before the final minute**: log price is a random walk. The settlement average has variance
  σ²·(τ − 2a/3), where τ is the time left and a = 60 s, because averaging removes two-thirds of
  the last minute's variance. P(UP) = F(ln(S/K) / (σ·√(τ − 2a/3))).
- **Inside the final minute**: the elapsed part *e* of the average is known (R). YES needs the
  remaining average to reach K′ = (a·K − e·R)/τ, whose variance is σ²·τ/3.
- **F** is a unit-variance Student-t with ν degrees of freedom (fat tails), or a normal curve
  when ν = 0.
- **From bars** (history, or when ticks aren't available), a bar's time-average is estimated as
  (O + C + 3H + 3L)/8. In a Monte Carlo of per-second random-walk prices its error against the
  true 60-second mean was about 20% smaller than OHLC4's, and still a little smaller when
  exchanges print large wicks.

## Limitations

- **The target and settlement are estimates.** The composite is not BRTI. Expect differences of
  a few dollars, so trust Kalshi's displayed target over this one.
- **Past windows use bar-based estimates.** In simulation a 1-minute bar pins down the 60-second
  average only to about ±$8–11 (1σ), so close windows in history may be marked on the wrong side.
  Live windows use tick sampling, which is much tighter.
- **After a reload** (changing settings, switching timeframes), live tick estimates are gone and
  bar estimates are used.
- **Seconds timeframes** need a TradingView plan that includes them.
- **Momentum has no proven edge.** Use the hit-rate row and judge for yourself.
