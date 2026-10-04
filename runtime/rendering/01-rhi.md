# The RHI (Rendering Hardware Interface)

The lowest part of a renderer, and strictly speaking part of the **platform layer**: one interface over the
different GPU APIs (OpenGL, Vulkan, DirectX, Metal).

## What it offers

The RHI exposes what every GPU API can do, without the API's own types:

```
interface RHI {
    BufferHandle   createBuffer(BufferDesc desc);           // vertices, indices, uniforms
    TextureHandle  createTexture(TextureDesc desc);         // images and render targets
    ShaderHandle   createShader(ShaderDesc desc);
    PipelineHandle createPipeline(PipelineDesc desc);       // shaders + blending + depth + vertex layout
    CommandList    beginCommands();
    void           submit(CommandList list);
    Swapchain      createSwapchain(NativeSurface surface, SwapchainDesc desc);  // optional: only to show on screen
    void           resizeSwapchain(Swapchain swapchain, int width, int height);
    void           present(Swapchain swapchain);
    void           destroy(Handle h);                       // deferred, see GPU resources
}

interface CommandList {
    void beginPass(RenderPassDesc pass);   // which targets to draw into, clear or keep them
    void setPipeline(PipelineHandle p);
    void setVertexBuffer(BufferHandle b);
    void setTexture(int slot, TextureHandle t);
    void draw(int vertexCount, int instanceCount);
    void endPass();
}
```

## Windows, surfaces and swapchains

The RHI does not create windows. The windowing layer creates the window and hands over a **native surface**: an
opaque handle to "the part of this window the GPU may draw into". From it, the RHI creates a **swapchain**: the small
set of images that are drawn into and shown on screen in turn. Without a surface, the RHI still works fully: it renders
into offscreen textures. How the two services meet: [Windowing and rendering](../windowing/06-windowing-and-rendering.md).

## Two patterns

- **Bridge / Adapter.** The renderer is written against `RHI`; `OpenGLRHI`, `VulkanRHI` and `MetalRHI` implement
  it. Porting to a new GPU API means writing one backend, and nothing above it changes.
- **Command pattern at the GPU level.** Drawing doesn't happen when `draw` is called: it is **recorded** into a
  command list and **submitted** later. Modern APIs (Vulkan, DirectX 12, Metal) work this way natively, and it is
  what allows several threads to record commands at the same time ([5](05-frames-and-threads.md)). For an older
  API like OpenGL, the RHI can record into its own list and replay it on the main thread.

## Pipeline state objects

Modern APIs bundle most drawing state (shaders, blending, depth testing, vertex layout) into one immutable
**pipeline** object, created up front. Changing state means binding a different pipeline. This makes state changes
cheap and predictable, and it is why renderers sort draws by pipeline ([4](04-frame-pipeline.md)).

## Rules

- **Nothing above the RHI calls a GPU API directly.** One stray call ties the whole renderer to that API again.
- **Keep it thin.** The RHI translates; it doesn't decide what to draw or in which order. Policy belongs above it.
- **Lowest common denominator, with optional features.** Expose what every backend can do; offer the rest as
  capabilities that can be queried (`rhi.supports(Feature.COMPUTE)`).
