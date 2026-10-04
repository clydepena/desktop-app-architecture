# Execution

When scripts run, how much they may do, and what happens when they fail.

## Where scripts attach

| Attachment | What it is | Example |
|---|---|---|
| **Script component** | behavior on one object | a door that opens when approached |
| **Script system** | a rule over many objects | "every burning object loses health each second" |
| **Event handler** | a reaction to something happening | "player entered zone" starts a cutscene |
| **Command** | something run on request | a console command, an editor menu item |

## When scripts run

Scripts run inside the **update phases** of [Time and scheduling](../time/05-update-phases-and-order.md). A gameplay
script in the fixed-step phase sees ticks; a UI script runs with real time. Scripts never run "whenever": the host
calls them at defined points, which keeps their effects in a predictable order.

## Coroutines

```
routine intro() {
    showDialog("Welcome!");
    yield waitSeconds(2, gameTime);
    door.open();
    yield waitUntil(() -> player.isOutside(room));
    door.close();
}
```

Writing sequences over time as plain sequential code is the main reason designers find scripting approachable. The
coroutines are provided and resumed by the time service ([Timers, delays and coroutines](../time/06-timers-and-coroutines.md)),
and owned by the script's object so they stop when it is destroyed.

## Budgets

A runaway script (an endless loop, a huge search) must not freeze the application:

- **instruction or time limits** per call or per frame
- **time slicing** for long work ([Time slicing and budgets](../time/07-time-slicing-and-budgets.md))
- a script over budget is **suspended or stopped**, with a message naming it

## Errors never crash the host

A script error is caught **at the boundary**, logged with the script's own **stack trace and source location**
([Tooling](08-tooling.md)), and only that script, object or mod is disabled. The application keeps running, and the
error appears in the problems list.

## Determinism

Scripts inside a deterministic or networked simulation follow its rules
([Determinism and replay](../time/08-determinism-and-replay.md)):

- ticks, not real time
- the simulation's seeded random numbers, not the language's own
- stable iteration order: script features with unpredictable order (iterating some hash maps) are avoided or replaced

A script that reads the real clock inside a lockstep simulation desynchronizes every player.
