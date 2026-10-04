# Platform abstraction

The lowest layer: thin wrappers around the operating system and hardware, so everything above runs unchanged on
every platform. Each wrapper is small and does no policy: it exposes what the OS offers behind one interface.

Each wrapper is documented inside the folder of the runtime service that uses it, and linked below.

| Topic | Wraps |
|---|---|
| Windows and surfaces | creating windows, resizing, DPI, fullscreen, multiple monitors; written: [Windowing and display](../runtime/windowing/README.md) |
| GPU API | OpenGL / Vulkan / DirectX / Metal behind one rendering hardware interface; written: [The RHI](../runtime/rendering/01-rhi.md) |
| Audio devices | output and input devices, sample formats, device changes; written: [The device layer](../runtime/audio/01-device-layer.md) |
| File system | paths, reading and writing, directory listing, file watching, memory mapping; written: [Platform and file systems](../runtime/io/01-platform-and-file-systems.md) |
| Sockets | TCP / UDP, TLS, HTTP; written: [Platform and transport](../runtime/networking/01-platform-and-transport.md) |
| Threads and synchronization | threads, priorities, cores, locks, atomics, thread-local data; written: [Platform threads](../runtime/threads/01-platform-threads.md) |
| Virtual memory | pages, reserve and commit, alignment, memory pressure; written: [Platform and virtual memory](../runtime/memory/01-platform-and-virtual-memory.md) |
| Clocks | high-resolution time, sleeping, timers; written: [Platform clocks](../runtime/time/01-platform-clocks.md) |
| Input devices | keyboard, mouse, gamepads, touch, text input and IME; written: [Platform and devices](../runtime/input/01-platform-and-devices.md) |
| OS integration | clipboard, file dialogs, notifications, opening URLs and file managers; written: [OS integration](../runtime/windowing/05-os-integration.md) |

The runtime services built on top of these (rendering, audio, input, IO, networking, time, memory, threads, windowing) are documented in
[runtime/](../runtime/).
