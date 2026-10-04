# GPU resources

Buffers, textures, shaders, pipelines and render targets live in GPU memory, have to be created and freed
explicitly, and may still be in use by the GPU when the CPU wants to change them. Three ideas keep this safe.

## 1. Handles instead of raw objects

Code holds a `TextureHandle`, never an OpenGL id or a Vulkan pointer. This is the **Proxy pattern**: the handle
stands in for the real resource, so the renderer can:

- create the resource later (when its data arrives)
- recreate it (the window was resized, a shader was hot-reloaded, the device was lost)
- move it between memory pools
- free it

and every holder of the handle keeps working.

### Index + generation

A common handle is two numbers packed together:

```
handle = (index into a pool of slots, generation of that slot)
```

When a slot is freed, its generation is increased. An old handle to that slot now has the wrong generation, so it
is detected as **stale** instead of silently pointing at whatever was created in the slot afterwards. This is a
**pool** (or free list) of slots, and it makes handles small, cheap to copy and safe.

## 2. Deferred deletion

The GPU runs a frame or two **behind** the CPU ([5](05-frames-and-threads.md)). A texture "deleted" by the CPU
now may still be read by a frame the GPU hasn't finished. So deletion is queued:

```
destroy(handle)  ->  put it in the queue for the current frame
when the GPU reports that frame finished  ->  actually free everything queued for it
```

The same applies to **changing** data the GPU might be reading: write to a new buffer (or the next of several
buffers) instead of overwriting one in use.

## 3. Uploads

Getting data into GPU memory is its own step:

- Data goes through a **staging buffer** (CPU-visible memory), then is copied into fast GPU-only memory.
- Large uploads (textures, meshes) are prepared on worker threads ([Jobs](../06-jobs.md)) and finish a frame or
  more later; until then, the resource is "not ready" and a placeholder is used.
- The asset system's handles ([Resources](../07-resources.md)) wrap these GPU handles: decoding a file happens in
  the asset system, creating the GPU resource happens here, on the thread the RHI allows.

## Sharing

One mesh, material or texture is usually used by many objects. The renderer stores it **once** and every user holds
the same handle (the **Flyweight** pattern). Materials are often split into a shared part (shader, textures) and a
small per-object part (a color tint), so thousands of objects can share one material and still differ.
