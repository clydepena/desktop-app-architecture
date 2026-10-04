# Assets

Which assets a renderer uses, and at which layer. Assets are the files created by people (artists, technical
artists) or tools, loaded through the asset system ([Resources](../07-resources.md)) and handed to the renderer
through its resources doorway.

How source files are imported, cached and cooked into the runtime formats below is described in the
[asset pipeline](../../tool-framework/asset-pipeline/README.md).

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **RHI** | **shaders** | GPU programs (vertex, fragment/pixel, compute), compiled for each GPU API |
| | **pipeline descriptions** (often generated from materials) | blending, depth testing, culling, vertex layout |
| **GPU resources** | **textures** | images: color, normal, roughness, masks, UI images, icons |
| | **texture atlases / sprite sheets** | many small images packed into one texture, with the rectangle of each |
| | **meshes** | vertices and indices of 3D models, or of 2D shapes |
| | **fonts** | glyph shapes, rasterized into a glyph atlas (or signed distance fields for sharp scaling) |
| | **cube maps / environment maps** | the surroundings, for skies, reflections and image-based lighting |
| | **lookup tables (LUTs)** | color grading, precomputed lighting data |
| **Render world and proxies** | **materials** | which shader, which textures, and parameter values (color, roughness); shared by many objects |
| | **sprites** | a texture (or atlas region), pivot, size in world units |
| | **models / prefabs** | meshes plus the materials on each part, often with a skeleton |
| | **animations** | keyframes for skeletons, sprite frame sequences, material parameter curves |
| | **particle systems** | emitter rules: spawn rate, lifetime, motion, size and color over time |
| | **tile sets and tile maps** (2D) | which atlas regions form tiles, and grids of tile ids |
| | **light and camera presets** | color, intensity, shadow settings; field of view, clipping |
| **Frame pipeline** | **render pipeline / render graph descriptions** | which passes run, in which order, with which targets |
| | **post-processing profiles** | bloom, exposure, color grading, vignette settings, often blended by area |
| | **render quality presets** | resolution scale, shadow quality, anti-aliasing per quality level |

## Source formats and runtime formats

Rendering assets are usually **authored in one format and shipped in another**:

| | Source (what people edit) | Runtime (what the renderer loads) |
|---|---|---|
| Textures | PNG, PSD, TIFF, EXR | GPU-compressed formats (BC/DXT, ETC, ASTC) with mipmaps, ready to upload |
| Meshes and models | FBX, glTF, OBJ, a modeling tool's own file | packed vertex and index buffers, optimized for the GPU's caches |
| Shaders | GLSL / HLSL / a shader graph | compiled bytecode per GPU API (e.g. SPIR-V, DXIL), plus a reflection description of its inputs |
| Fonts | TTF / OTF | glyph atlases, or signed distance field atlases |
| Materials, particles, pipelines | editable descriptions (JSON, a node graph) | compact binary descriptions |

An **import step** converts sources into runtime formats: compression, mipmap generation, atlas packing, mesh
optimization, shader compilation for each target. The settings for each source (compression, filtering, pixels per
unit, sRGB or linear, how to slice an atlas) are **import settings**, stored next to the source file (a sidecar),
not in it.

## Shared vs per-object data

Most rendering assets are **shared**: one texture, mesh or material used by thousands of objects (the
**Flyweight** pattern). Per-object differences (a tint, a position, an animation time) live in the render proxy,
not in the asset. Editing a material asset changes every object using it; giving one object a different look means
a **material instance** (a small asset that overrides some parameters of a parent material) or a per-proxy
parameter.

## Assets vs settings vs runtime state

- **Assets** describe how things look: textures, materials, pipelines. Created by artists, saved with the project.
- **Settings** are the user's choices: resolution, vsync, quality preset. Saved per user
  ([Settings](../04-settings.md)); they select between quality presets but don't change the assets.
- **Runtime state** is what is happening now: proxies, render targets, frame data. Never saved as an asset.

## Dependencies and hot reload

Materials reference shaders and textures; models reference meshes and materials; atlases are referenced by sprites
and tile sets. The asset system loads dependencies first, reports missing ones as problems
([Diagnostics](../05-diagnostics.md)), and on a change (a texture re-exported, a shader edited) reloads the asset
behind its handle, so every proxy using it updates without being rebuilt.
