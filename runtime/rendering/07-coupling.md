# Coupling: what may depend on the renderer

Rendering is one of the easiest services to couple everything to by accident, and one of the most expensive to
untangle later. One rule decides almost every case.

## The rule: direction

**Upper layers may depend on the renderer's public interface. The renderer depends only on what is below it**
(the RHI, jobs, logging, math). Anything that must flow upward ("the device was lost", "a frame finished",
"statistics updated") goes out as **events**, never as calls into application code.

## Safe to couple

These depend on the renderer's interface, in the allowed direction:

| System | Why it's safe |
|---|---|
| **Lifecycle** | owns the renderer's lifetime and drives frames: one well-defined caller |
| **RHI, and native surfaces** | below the renderer: the renderer depends on the RHI, and on a window's surface only when it presents to one, never on the window itself |
| **Resource loaders for GPU data** | their finishing step must create GPU resources; they only use the resource doorway |
| **Bridges / render components** | connecting the domain to the render world is their whole purpose |
| **UI backend** | turns UI draw data into a pass; small and replaceable |
| **Editor viewports, picking, overlays, thumbnails** | the tool layer sits above the renderer and uses views, targets, readback and passes |
| **Debug and diagnostics panels** | read statistics, use immediate drawing; usually editor or debug builds only |

## Keep decoupled

These must not know the renderer, or the renderer must not know them:

| System | How it reaches rendering instead |
|---|---|
| **Domain logic** (game rules, the document model) | through bridges; domain code sets a position and an image, never GPU state |
| **Undo, documents, persistence, actions** | they change the domain; bridges carry those changes to proxies. Undo never stores render data |
| **Selection** | hands a list of ids to an outline pass owned by the editor; the renderer doesn't know "selected" |
| **Physics, audio, networking** | independent services; to draw debug shapes or a waveform they use immediate drawing or a bridge |
| **Input and camera controls** | input moves a camera object; the camera becomes view data. The renderer never reads input |
| **IO and file loading** | the renderer receives decoded data and never opens files |
| **Jobs** | the renderer may use jobs to record in parallel; jobs never call rendering on their own threads unless the renderer allows it |

## Mistakes in the wrong direction

- **The renderer calling application code**, e.g. asking each object how to draw itself every frame. The answer
  belongs in the proxy.
- **Domain objects holding GPU handles** or calling the graphics API.
- **The renderer knowing tool concepts**: selection, gizmos, "is the editor running?". Those are passes and views
  the editor registers.
- **Saved files containing render data** (GPU ids, batches, sort keys) instead of domain data.
- **Global access to the renderer** from anywhere, which hides who depends on it (see
  [Services and scopes](../02-services-and-scopes.md)).

## Testing the boundary: run without a renderer

Give the renderer a **null implementation** (the **Null Object** pattern): every call accepted, nothing drawn, ids
handed out as usual.

```
class NullRenderer implements Renderer {
    int createSprite(SpriteDesc d) { return nextId++; }
    void setTransform(int id, Matrix m) {}
    void render(List<View> views) {}
    ...
}
```

If the domain, documents, undo, saving and the test suite all run on it (a command-line converter, a dedicated
server, automated tests on a machine without a GPU), the decoupling holds. Anything that crashes or demands a GPU
reveals a hidden coupling, and shows exactly where it is.
