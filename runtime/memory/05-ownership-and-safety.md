# Ownership and safety

Most memory bugs are ownership bugs: freed while still used ("use after free"), freed twice, or never freed.

## Every piece of memory has one owner

The owner (an object, a scope, a system) is responsible for freeing it. Others may **borrow** it (use it while the
owner keeps it alive) but never free it. Shared data is either:

- owned by a **scope that outlives all its users** (a workspace owning assets used by several documents), or
- **reference counted**

## Reference counting

Data is freed when the last user lets go.

- Suits shared assets and resources with several independent users.
- **Cycles** are never freed: two objects holding counted references to each other keep each other alive. Break
  cycles with weak (non-counting) references, usually from child to parent.
- Counting from several threads needs atomic operations, which are slower; avoid counting in hot loops.

## Handles with generations

From [GPU resources](../rendering/02-gpu-resources.md): instead of raw pointers, hand out

```
handle = (slot index, generation of that slot)
```

When a slot is freed, its generation increases. A stale handle no longer matches, so using it is **detected** (an error
or a null result) instead of silently reading whatever reused the slot. Use after free becomes a clear error instead of
memory corruption. Handles are also small, copyable and safe to store in other systems and in serialized data (as ids).

## Deferred destruction

Memory still read by the GPU, another thread or an in-flight frame is freed **later**, at a safe point:

- after a GPU fence for that frame
- at a frame boundary, after a render thread has moved on
- when an asynchronous request referencing it has completed or been canceled

Freeing immediately "because I'm done with it" is a classic crash when someone else isn't.

## Scoped cleanup

Language constructs that free automatically at the end of a scope make the owner's job hard to forget:

| Language | Construct |
|---|---|
| C++ | RAII: destructors, smart pointers (`unique_ptr`, `shared_ptr`) |
| Java | `try-with-resources` with `AutoCloseable` |
| C# | `using` with `IDisposable` |
| Rust | ownership and `Drop`, checked by the compiler |

## Across service boundaries

No service frees another service's memory, or keeps raw pointers into it past a frame. Services exchange **handles**,
**copies**, or data owned by a **shared scope** whose lifetime is clear to both.
