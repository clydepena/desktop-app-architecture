# Timers, delays and coroutines

Running something **later**: in 2 seconds, every half second, after a door finishes opening.

## Timers

- **One-shot** ("call this in 2 seconds") and **repeating** ("every 0.5 seconds").
- Stored in a **priority queue** ordered by when they fire, or a **timer wheel** when there are very many.
- Checked once per frame (or per tick), at a fixed point in the update phases.

### Every timer runs on a clock

A timer measured on **game time** pauses with the game and slows down in slow motion; one on **real time** doesn't
([Time domains](04-time-domains.md)). The clock is chosen when the timer is created, never assumed.

### Owned and cancellable

A timer belongs to an **owner** (an object, a panel, a scope) and is **canceled** when the owner is destroyed.
Otherwise it fires on something that no longer exists: one of the most common crashes in event-driven code. Timers
return a handle that can cancel them, and owners cancel theirs automatically when their scope closes
([Services and scopes](../02-services-and-scopes.md)).

## Coroutines

Sequences of steps spread over time, written as one sequential function:

```
routine openDoor() {
    playSound("door_unlock");
    yield waitSeconds(1.0, gameTime);
    door.open();
    yield waitUntil(() -> player.isOutside(room));
    door.close();
}
```

The scheduler resumes the routine each frame until its wait is over. The same logic written with timers and
callbacks is much harder to read and to cancel. Coroutines are owned and canceled like timers.

Many languages provide this as `async` / `await` or generators; game engines often provide their own, resumed by the
frame loop.

## In deterministic simulations

Timers and coroutines inside a deterministic simulation count **ticks**, not seconds ("fire on tick 4,210"), so every
machine fires them on exactly the same step ([Determinism and replay](08-determinism-and-replay.md)).
