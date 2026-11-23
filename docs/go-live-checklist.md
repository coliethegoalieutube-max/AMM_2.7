# Go-Live Checklist

Promote from shadow-trading to live capital only when every item below is checked.

## Data & model

- [ ] Phase 1 pipelines stable (historical + live L2)
- [ ] Walk-forward model validated on recent held-out weeks
- [ ] Universe / liquidity floor config frozen for first live period

## Backtest

- [ ] Maker fills use trade-through / queue logic only
- [ ] Fees applied with correct `fee_type` / multiplier
- [ ] Adverse-selection metrics reviewed (winner vs. loser fill rates)
- [ ] Preferred entry window, ask target, glide path, and hard cutoff selected

## Plumbing (demo)

- [ ] RSA signing verified
- [ ] Create / amend / cancel orders verified
- [ ] Kill-switch cancel path verified
- [ ] WebSocket reconnect + exponential backoff verified

## Shadow (production feed, paper)

- [ ] 2–4 weeks of NCAAB slates logged
- [ ] Fill rates ≈ backtest band
- [ ] Adverse-selection drag ≈ backtest band
- [ ] Net PnL trajectory consistent with backtest (direction and magnitude)

## Risk (first live capital)

- [ ] Per-game size capped (e.g. 1–2% bankroll)
- [ ] Concurrent open-contract cap set
- [ ] Daily max loss set
- [ ] Sharp ≥2¢ kill switch live
- [ ] Hard flatten cutoff live (no tip-off inventory)
- [ ] +5m / +30m mark-out logging live

## Decision

- [ ] Written go / no-go with date and reviewer
- [ ] Initial capital size recorded
- [ ] Scale-up criteria defined (e.g. N days green mark-outs + PnL)
