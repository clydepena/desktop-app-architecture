# Platform clocks

The operating system offers two different clocks, and mixing them up is a classic bug.

| Clock | What it is | Use it for | Never use it for |
|---|---|---|---|
| **Monotonic** | a counter that only moves forward, at a steady rate, from an arbitrary starting point | measuring durations: frame time, timeouts, profiling | showing the date |
| **Wall clock** | the calendar date and time | timestamps in logs and files, "last opened 3 days ago" | durations: it **jumps** when the system synchronizes its time, at daylight-saving changes, or when the user changes it |

A timeout measured with the wall clock can fire instantly, or never, when the clock jumps. Frame times measured with
it can be negative.

## Resolution

Monotonic clocks are usually precise to well under a microsecond (`QueryPerformanceCounter` on Windows,
`clock_gettime(CLOCK_MONOTONIC)` on Linux, `mach_absolute_time` on macOS). Reading them is cheap, but not free:
profilers that read the clock millions of times per frame notice.

## Sleeping is imprecise

Asking the OS to sleep for 1 ms may sleep anywhere from 1 to about 16 ms, depending on the platform's **timer
resolution** and how busy the system is. Consequences:

- A frame limiter built only on `sleep` produces uneven frames.
- Precise waits combine a **short sleep** (giving the CPU back for most of the wait) with a **final busy-wait** on the
  monotonic clock, or use the platform's high-resolution waitable timers.
- Raising the system timer resolution makes sleeps more precise at a small power cost; it should be done only while
  needed.

## Pattern

**Bridge / Adapter:** one interface (`now()`, `sleepUntil(t)`, `wallClock()`) over each platform's clock APIs.
