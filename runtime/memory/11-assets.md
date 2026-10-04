# Assets

Memory uses no content of its own. What it uses is **configuration**, set per platform.

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Budgets and tracking** | **memory budgets** | limits per category and per platform; warning thresholds |
| **Allocation strategies** | **pool capacities** | voices, messages, particles, objects per type |
| | **arena and buffer sizes** | frame arena sizes, scratch sizes, ring buffer sizes |
| **GPU memory** | **streaming budgets** | texture pool size, mesh budgets, per quality preset |
| **Tools** | **memory snapshots** | saved for comparison between builds or sessions; tool data, not shipped |

## User settings vs assets vs runtime state

- **User settings:** quality options (texture quality, view distance, particle density) that choose **which budget
  preset** applies ([Settings](../04-settings.md)). Users don't set byte counts.
- **Assets:** the budgets, capacities and sizes themselves. Saved with the project, per platform; changing them is a
  design decision with consequences for content.
- **Runtime state:** live counters, arenas, pools. Never saved.

## Budgets drive content

Budgets are only useful if content is checked against them. Build tools can report, per level or per asset group,
how much memory it needs in each category, and fail the build (or warn) when a level exceeds its platform's budget,
long before it crashes on that platform.
