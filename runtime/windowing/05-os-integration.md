# OS integration

Everything else the operating system offers around windows. Each item is small, but together they decide whether an
application feels native.

## Identity and status

- **Title and icon** per window; application icons in several sizes for the taskbar, dock, window corner and file
  manager.
- **Taskbar / dock** progress bars ("exporting 40%"), badges (unread counts), and "flash for attention" when a long
  task finishes in the background.
- **Notifications** from the system, for events while the application isn't focused.

## Dialogs

- **Native file dialogs:** open, save, choose folder, with file type filters. They know the user's recent folders and
  cloud drives, so they are usually better than custom ones.
- **Message boxes** for rare, blocking questions.
- Prefer the **non-blocking** forms where the platform offers them: blocking dialogs run a modal loop that stops the
  application's frames ([Platform and the message pump](01-platform-and-message-pump.md)). Results arrive as futures.

## Clipboard and drag and drop

The platform side of [Clipboard and drag-and-drop](../../tool-framework/08-clipboard-and-drag-drop.md): reading and
writing several formats on the system clipboard, receiving files and data dropped from other applications, and
starting drags out of the application.

## System preferences

- **Theme:** dark or light, accent color; follow it, or let the user override.
- **Accessibility preferences:** high contrast, reduced motion, larger text. The application should respect them.
- **Locale:** language, number and date formats (used by localization).

All of these can change while the application runs, and are reported as events.

## Opening things outside

- opening a **URL** in the default browser
- opening a **folder** in the file manager, or showing a file selected in it ("show in folder")
- opening a file with its **default application**

## File associations

Registering file types (".drawing opens with this application") is done by the installer or the platform's packaging;
the application receives the path as a command-line argument or an "open file" message, and handles it like a file
opened from inside ([Lifecycle](../01-lifecycle.md)).
