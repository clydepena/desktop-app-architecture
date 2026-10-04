# Assets

Windowing uses a small set of assets, mostly about the application's identity on the desktop and the defaults for its
windows.

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Window manager** | **window definitions** | default and minimum sizes, resizable or not, decorations, for the main window and each tool window |
| **Displays** | **display mode presets** | supported resolutions or render scales per platform; HDR calibration defaults |
| **OS integration** | **application icons** | the icon in several sizes and formats per platform (taskbar, dock, window corner, file manager) |
| | **splash screen** | the image shown while the application starts |
| | **cursor images** | custom cursors, with their hotspot (the pixel that points) and sizes for different scales |
| | **file type associations** | which extensions open the application, with their icons (used by the installer or package) |

## Settings, UI state and runtime state

- **User settings, not assets:** display mode, chosen monitor, resolution, vsync, render scale, a UI scale override
  ([Settings](../04-settings.md)).
- **UI state, not assets:** window placement, sizes, states and per-window layouts
  ([UI state](../../tool-framework/09-ui-state.md)).
- **Runtime state:** the windows themselves, their surfaces, current scale and focus. Never saved as assets.

## Sizes for every scale

Icons and cursors are needed at several sizes, because monitors use different content scales
([Displays, scale and modes](03-displays-scale-and-modes.md)): an icon drawn only at 32 x 32 pixels looks blurry at
200%. Provide each at the common scales, or as vector images where the platform accepts them.
