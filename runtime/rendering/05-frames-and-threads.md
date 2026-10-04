# Frames and threads

The CPU and the GPU work in parallel, and a renderer is organized around keeping both busy without either
waiting for, or corrupting, the other.

## Frames in flight

The CPU prepares frame N+1 while the GPU is still drawing frame N:

```
CPU:  [ prepare 1 ][ prepare 2 ][ prepare 3 ][ prepare 4 ]
GPU:               [  draw 1   ][  draw 2   ][  draw 3   ]
```

Usually two or three frames are "in flight". More frames means smoother throughput but more delay between input
and what appears on screen; fewer means lower delay but more waiting.

## Per-frame data: double and triple buffering

Data the CPU writes every frame (camera matrices, per-object data, staging memory for uploads) can't simply be
overwritten: the GPU may still be reading last frame's copy. So each such buffer exists **once per frame in
flight**, and the CPU writes the copy for the frame it is preparing:

```
frame 1 -> buffer A,  frame 2 -> buffer B,  frame 3 -> buffer C,  frame 4 -> buffer A again (after frame 1 finished)
```

Before reusing a copy, the CPU waits on a **fence**: a signal the GPU raises when it has finished that frame. The
same fences drive deferred deletion ([GPU resources](02-gpu-resources.md)).

## Where rendering code runs

Three common arrangements, from simplest to most parallel:

| Arrangement | How it works | Trade-off |
|---|---|---|
| **Single thread** | the main loop updates the app, then renders | simplest; the CPU work of rendering adds to the frame time |
| **Render thread** | the main thread updates and produces frame data; a dedicated thread consumes it and talks to the GPU | the app and the renderer overlap; data must be handed over (proxies, command queues) |
| **Parallel recording** | worker threads record command lists for different passes or object groups at the same time | uses many cores; needs an API that allows it (modern ones do) |

The render thread is the **producer / consumer** pattern: the main thread produces a frame's worth of data
(proxy updates, view settings), the render thread consumes it. This only works because of render proxies
([3](03-render-world-and-proxies.md)): each thread has its own data.

## Rules

- **Never block the main thread on the GPU** except at the fence for the oldest frame in flight.
- **Hand over, don't share.** Data crosses threads as messages or snapshots, not as objects both threads modify.
- **Resize and device loss are frame events.** Recreate swapchains and size-dependent targets between frames,
  through the handles, so nothing holding a handle notices.
