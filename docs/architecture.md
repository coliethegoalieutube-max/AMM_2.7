# Architecture Overview

## Goal

Predict which Kalshi NCAAB markets will see excess retail-driven price drift in the final ~24–30 hours before tip-off, capture that drift with passive maker orders, and flatten inventory before tip so PnL is independent of game outcome.

## High-level flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA LAYER (Phase 1)                        │
│  Kalshi historical  │  Odds API (sharp/retail)  │  Context     │
│  + Live L2 recorder │  T-36h … T-30m snapshots  │  polls/TV   │
└────────────┬────────────────────┬───────────────────┬───────────┘
             │                    │                   │
             ▼                    ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│              FEATURES + MODEL (Phase 2)                         │
│  Universe filter → Excess-drift label → GBT (walk-forward)      │
│  Rank slate → trade only top 5–10% conviction                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│         BACKTEST → SHADOW → LIVE (Phases 3–5)                   │
│  Maker fill sim + fees → Demo plumbing → Live paper → Capital   │
│  Sharp kill switch · Ask glide path · Hard flatten · Limits     │
└─────────────────────────────────────────────────────────────────┘
```

## Why two data tracks?

| Track | Source | Purpose |
|-------|--------|---------|
| Historical backfill | Kalshi REST historical APIs + Odds API + context | Train/validate the inflow/drift model |
| Live L2 logger | Kalshi production WebSocket `orderbook_delta` / `trade` | Queue-position and maker fill realism for backtest & shadow |

Historical trade logs do **not** capture resting depth. Without L2 snapshots, maker fill assumptions are fiction.

## Core strategy mechanics

1. **T−36h to T−24h** — Run inference; filter to top-decile + liquidity floor; post passive maker bids near top of book.
2. **On fill** — Immediately post resting target ask (+3¢ / +4¢ / +5¢ above entry, tuned in backtest).
3. **Glide path** — Step ask down at T−2h, T−1h, T−30m to maximize passive exits.
4. **Hard flatten** — Force flat by T−30m / T−15m / T−5m (never T−1m into MM widening).
5. **Kill switch** — Cancel unfilled bids if sharp consensus drops ≥2¢.

## Module ownership (planned)

| Package | Responsibility |
|---------|----------------|
| `src/collect` | Backfill jobs, Odds API pulls, L2 WebSocket logger |
| `src/features` | Join timelines, build feature matrix & labels |
| `src/models` | LightGBM/XGBoost train, walk-forward, slate ranking |
| `src/backtest` | Fill logic, fee schedule, glide path, metrics |
| `src/execution` | REST/WS clients, RSA signing, order lifecycle |
| `src/risk` | Exposure caps, kill switches, mark-out audit |

## Environments

| Purpose | Kalshi REST | Kalshi WebSocket |
|---------|-------------|------------------|
| Demo plumbing | Demo API | `wss://…demo…/trade-api/ws/v2` |
| Live data / shadow / capital | Production API | Production WS |

Demo order books lack real retail flow and sharp adverse selection — use demo only for API correctness.
