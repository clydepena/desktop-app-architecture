# GPU memory

Video memory (VRAM) follows the same ideas as CPU memory (lifetimes, pools, budgets, deferred freeing) but is managed
by the renderer and its hardware interface ([Rendering](../rendering/README.md)), not by general-purpose allocators.

## A separate budget

- VRAM is usually separate from main memory and smaller.
- It holds textures, meshes, render targets, buffers and the swapchain.
- It has its own **budget**, tracked by category (textures, meshes, targets) like CPU memory.

## Allocation on the GPU

- GPU APIs offer large **heaps**; renderers suballocate from them with their own pools and arenas (allocating each
  resource separately from the driver is slow and limited in number).
- **Per-frame data** (uniforms, dynamic vertices, uploads) uses ring buffers or arenas per frame in flight
  ([Frames and threads](../rendering/05-frames-and-threads.md)).
- **Freeing is deferred** until the GPU has finished every frame that might use the resource
  ([GPU resources](../rendering/02-gpu-resources.md)).

## Staying under budget: streaming

The renderer keeps textures and meshes at the detail it can afford:

- **mip streaming:** each texture is loaded at the highest detail level that fits the budget and is visible at the
  object's size on screen; distant or small objects use smaller levels
- when over budget, the least recently needed high-detail levels are evicted first
- memory pressure on the CPU side and GPU budgets are often coordinated: lowering texture quality frees both

## Residency

On modern APIs, the application decides (more or less directly) which resources are **resident** in VRAM. Resources
not used for a while can be evicted to main memory by the driver or the renderer, and brought back when needed. Missing
this shows up as sudden stutters when a resource has to be uploaded again.

## Unified memory

Some hardware (many laptops, phones, consoles and some desktop chips) shares one memory between CPU and GPU. There is
no copy between them, but also no separate budget: CPU and GPU budgets are combined, and CPU-side growth reduces what
the GPU can use.
