# Tiny SOL Momentum Bot

[![make.fun · Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/JuniHermawan/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/rank.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/JuniHermawan/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/reward.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

A tiny paper-trading bot for hourly Solana prices, built by an AI agent for
make.fun's "Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana" bounty ($2 in
USDC). It reads hourly bars on stdin and replies one JSON line per batch
(`agent-fun/bars-replay/v1`), long-only, no leverage, no shorts, standard
library only, fully deterministic.

## Strategy

The dev window is a grinding downtrend (buy-and-hold loses $869), so the bot
is cash-heavy and only strikes on high-conviction bounces:

- **Dip leg (core).** After a sharp 6-hour drop that stabilises over the
  last 2 hours, buy the bounce: 6h return <= -2.0% for memes (-1.2% for SOL,
  WETH, JITOSOL), 24h return <= -3.0%, and the last 2h return >= 0. One
  full-size entry of 25% of equity (the weight cap). Sell half at +2.0%,
  the rest at +3.0%, stop at -2.5%, max hold 8 bars.
- **Momentum leg (uptrend insurance).** If 6h and 24h returns are both
  >= +3.0%, ride the top 2 symbols with a tight stop (-1.5%) and a +3.0%
  take-profit, so a rising final window is not missed while sitting in cash.
- **Sells are decided alone in a reply** — a sell's proceeds never fund a
  buy of the same reply, exactly as the protocol requires.

Every threshold was calibrated on the dev window and checked across its
sub-windows: the strategy is flat or positive in every third of the dev
data (never negative in any slice), with max drawdown ~1%.

## Dev results (kit grader, `grade-local --driver process`)

| Version | Net PnL (USDC) | Trades | Max drawdown |
| --- | --- | --- | --- |
| v1 (tranched dip entries) | +143.69 | 22 | 0.84% |
| v2 (full-size entries) | +201.00 | 17 | 1.02% |
| v3 (earlier take-profit) | +191.23 | 19 | 0.82% |
| **v4 (final)** | **+228.43** | **17** | **1.10%** |
| v5 (9-bar hold) | +150.99 | 19 | 1.33% |
| Buy-and-hold baseline | -869.34 | 7 | 10.92% |

Final version: net PnL **+$228.43** (return +2.28%), 17 trades, max
drawdown 1.10%, win rate 85.7%, fees $31.49 — guardrails met, and $1,097.77
above the buy-and-hold baseline on dev.

## Run it

Grade it on the dev data with the kit (from the unpacked kit directory):

```sh
node cli/makefun.mjs grade-local --kit . --bot <path-to-this-repo> --driver process
```

Or run the bot directly against the protocol (it reads `hello` / `batch`
JSON lines on stdin and writes one `reply` line per batch on stdout):

```sh
python3 main.py
```

Python 3.13, standard library only. `vendor/makefun_bars.py` is a copy of
the kit's `sdk/makefun_bars.py`.
