# Platform and virtual memory

What the operating system and hardware provide, behind one interface (the **Bridge / Adapter** pattern over
`VirtualAlloc`, `mmap` and their equivalents).

## Virtual memory

Each process sees its own large **address space**. The OS maps **pages** of it (usually 4 KB, sometimes 2 MB "large
pages") to physical memory on demand, and can move rarely used pages to disk.

- Addresses an application holds are virtual; two processes can use the same address for different data.
- Touching a page for the first time costs a small OS fault while the OS provides physical memory for it.

## Reserve and commit

An application can:

- **reserve** a large range of addresses (cheap: no physical memory is used yet), then
- **commit** pages inside it as it needs them

Arenas and pools use this to **grow without moving**: reserve 1 GB of addresses once, commit a few megabytes at a
time as the arena fills. Addresses handed out earlier stay valid.

## Alignment

Some data must start at addresses divisible by a power of two:

- SIMD types (16, 32 or 64 bytes)
- GPU upload buffers (often 256 bytes or more)
- cache lines (64 bytes), to avoid sharing them between threads ([Layout and performance](06-layout-and-performance.md))

Allocators take an alignment parameter; general-purpose allocation usually guarantees only 8 or 16 bytes.

## Memory pressure

- **Mobile** operating systems warn when memory runs low, and kill applications that don't free memory in response.
- **Desktops** usually page to disk first (everything becomes slow), then fail allocations.
- **Consoles** have fixed memory with no paging: running out is a crash.

The memory service turns OS warnings into **pressure events** that caches and services respond to
([Budgets, tracking and pressure](07-budgets-tracking-and-pressure.md)).

## The hardware cares about layout

The CPU reads memory in **cache lines** (usually 64 bytes) and keeps recently used lines in fast caches. Reading data
that is already cached is far faster than going to main memory. So:

- data used together should sit together
- walking memory in order is much faster than jumping around
- memory **bandwidth**, not arithmetic, is often what limits performance

This is the basis of [Layout and performance](06-layout-and-performance.md).
