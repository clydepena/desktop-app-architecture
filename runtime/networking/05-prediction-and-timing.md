# Prediction and timing

In a real-time session the other side is always in the past: a message sent now arrives tens or hundreds of
milliseconds later. These techniques hide that delay. They apply mostly to fast games; collaborative editors and
turn-based apps can usually skip them.

## Ticks and clocks

- The simulation runs at a **fixed tick rate** (e.g. 30 or 60 ticks per second), independent of the render frame
  rate. Rendering interpolates between ticks.
- Messages are stamped with **ticks**, not wall-clock time.
- Clients keep an estimate of the **server's clock** (from round-trip measurements), so "tick 4210" means the same
  moment everywhere.

## Client-side prediction

The client applies its own input **immediately**, without waiting for the server's answer. Moving feels instant.
The input is also sent to the server, which runs it authoritatively.

## Reconciliation

When the server's state for tick T arrives, the client:

```
1. sets its own object to the server's state at tick T
2. re-applies every input it sent after tick T (still unconfirmed)
3. if the result differs from what is on screen, blends towards it over a short time
```

If prediction was right (the usual case), nothing visible happens. If it was wrong (the server rejected a move,
another player blocked the way), the correction is smoothed. Badly tuned correction shows as **rubber-banding**.

## Interpolation

Other players' objects can't be predicted (their input is unknown), so they are shown **slightly in the past**,
blending between two received states:

```
render time = server time - interpolation delay (e.g. 100 ms)
position    = blend(state before render time, state after render time)
```

Movement is smooth despite irregular packets, at the cost of seeing others a little late. When packets are missing,
a short **extrapolation** (continuing the last motion) bridges the gap.

## Lag compensation

When a player shoots, they aimed at where they *saw* the target, which was in the past. The server briefly
**rewinds** other players to that moment (it keeps a short history of their positions) to check the hit. Shots feel
fair to the shooter; the target occasionally gets hit "behind a wall" they had already reached.

## Rollback

Used with deterministic simulations (fighting games, some lockstep games): predict the remote player's input (often
"same as last frame"), simulate ahead, and when the real input arrives and differs, **roll back** a few frames and
re-simulate with the right input. Requires a simulation that can snapshot, restore and re-run several frames
within one frame's time.

## What this needs from the rest of the app

- A simulation that runs in **fixed steps**, separate from rendering.
- Cheap **snapshots and restores** of the relevant state: the **Memento** pattern, the same idea as an editor's
  "play, then put everything back", run many times per second.
- Deterministic code where lockstep or rollback is used: the same inputs always give the same results.
