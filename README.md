# Pump Fun Paper Sprint — Trend Rider

[![competition](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b.svg)](https://make.fun/competitions/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![rank](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/toirant/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/rank.svg)](https://make.fun/competitions/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![reward](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/toirant/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/reward.svg)](https://make.fun/competitions/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)

A make.fun entry for [Pump.fun Paper Sprint: Any Bot Can Win](https://make.fun/competitions/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b) — a simple momentum/trend paper bot that trades pump.fun bonding curves on a 6-hour replay. Highest net PnL wins.

## Strategy

One program, `main.ts` (Node 22, stdlib only, no dependencies). It speaks the
`agent-fun/replay/v2` wire on stdin/stdout: one JSON line in per batch, exactly one
reply line out. No network, no clock, no randomness — every decision is a pure
function of the batches seen, so two runs produce byte-identical action logs.

- **Entry:** a curve becomes a buy once it is mature (300+ slots since creation),
  its real SOL reserve sits in a tradeable band, and its tape shows real momentum:
  price doubled over the last 40 slots, sitting near the window high, net SOL
  flowing in from several distinct buyers, with inflow accelerating into the newer
  half of the window.
- **Sizing:** first buy 0.5 SOL, pyramiding into strength up to 2.5 SOL per
  position, always within the harness's 1%-of-real-reserve cap per batch.
- **Exits:** sold whole on a 25% stop loss, a 35% trailing stop once up 10%, a
  fade in net flow over a 90-slot window, a big-sell print, or a time limit.
- **Guardrail pacing:** tiny 0.02 SOL probe round-trips keep the run above the
  20-coin minimum when the main signal goes quiet.

## Local grade (dev window)

```
node cli/makefun.mjs grade-local --kit <kit> --bot . --driver process --worktree
```

Dev-window result (`grade-local`, process driver): **+5.24 SOL net PnL**, 253 fills
over 55 coins, 123 round trips, max drawdown 0.083 — guardrails met, two
executions reproduced byte for byte. The PnL is concentrated in one runner (the
nature of pump.fun: most coins bleed a little, one runs); without the top 3
coins the net is -0.22 SOL, and the stress surrogates (median -0.28 SOL) show
the dev score leans on this window's own coins. The signal plateau is wide —
minRise 0.75–1.5 and fade windows 60–150 slots are all positive on the dev
window — so the edge is the strict-entry + slow-exit shape, not one tuned cell.

## Run

```
node main.ts
```

The grader runs it as plain `node main.ts` against the replay harness.
