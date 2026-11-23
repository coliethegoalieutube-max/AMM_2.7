# Fee & Fill Model

Reference for backtest and live cost accounting. Always prefer series-level `fee_type` / `fee_multiplier` from Kalshi metadata over hardcoded assumptions.

## Taker fee (quadratic)

\[
\text{fee} = \mathrm{round\_up}\bigl(M \times 0.07 \times C \times P \times (1 - P)\bigr)
\]

Where:

| Symbol | Meaning |
|--------|---------|
| \(M\) | Fee multiplier from series metadata |
| \(C\) | Contract count |
| \(P\) | Price in dollars (e.g. 0.50 for 50¢) |

Approximate cap: up to **~$1.75 per 100 contracts** at 50¢ (confirm against live fee schedule).

## Maker fee

| Series flag | Maker fee |
|-------------|-----------|
| Standard (maker free) | **$0** |
| `quadratic_with_maker_fees` | **25% of taker rate** (up to ~$0.44 per 100 contracts at 50¢) |

Order **cancellations are free**.

## Fill realism rules

1. Do **not** fill at mid.
2. Passive bid fills only on:
   - Trade prints **strictly through** the bid, or
   - Depletion of resting size ahead in L2 snapshots
3. Emergency taker exits (hard flatten) pay full taker fees + any spread cross.
4. Prefer passive ask exit via glide path to minimize taker liquidations.

## Implementation notes (future)

- Centralize fee math in one module (e.g. `src/backtest/fees.py`) shared by backtest and live accounting
- Unit-test known price/size combinations against Kalshi’s published examples when available
- Persist fee metadata with every market so historical replays stay accurate if schedules change
