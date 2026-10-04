# Assets

Which assets an input service uses, and at which layer. Most are **definitions and defaults** created by designers
and programmers, loaded like other assets ([Resources](../07-resources.md)); the player's own choices are user
settings instead.

How source files are imported, cached and cooked into the runtime formats below is described in the
[asset pipeline](../../tool-framework/asset-pipeline/README.md).

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Platform and device layer** | **controller database** | mappings from gamepad models to a standard layout |
| | **device profiles** | deadzones and response curves per device type |
| **Action mapping** | **action definitions** | the actions an application has, and their types (button, axis, 2D axis) |
| | **default bindings / action maps** | contexts and their bindings, per platform and device family |
| | **interaction presets** | hold durations, double-tap windows, thresholds |
| **Feedback** | **haptic patterns** | rumble strength over time per motor; finer haptic effects |
| | **light and trigger effects** | named controller light and adaptive trigger settings |
| **Prompts** | **glyph sets** | button icons per device family (keyboard and mouse, Xbox-style, PlayStation-style, touch) |
| **Touch** | **on-screen control layouts** | virtual joysticks and buttons: positions, sizes, which actions they feed |

## Defaults vs user settings vs runtime state

- **Assets** describe what the application offers: its actions, default bindings, interaction timings, prompts and
  haptics. Saved with the project, the same for everyone, often different per platform.
- **User settings** are the player's choices: rebindings, sensitivity, inversion, deadzones, hold vs toggle,
  vibration on or off ([Settings](../04-settings.md)). They are layered over the default bindings, so a new default
  in an update still reaches players who never changed that action.
- **Runtime state** is what is happening now: device states, the context stack, player assignments. Never saved as
  an asset.

## Tool data

**Recorded input sessions** (sequences of action snapshots) are saved for testing and bug reports. They are tool
data: kept with tests, not shipped with the product.

## Dependencies

Default bindings refer to actions by name; prompts refer to actions and glyph sets; on-screen layouts refer to
actions. Renaming an action is therefore a data migration: the asset system reports bindings and layouts that point
at actions that no longer exist ([Diagnostics](../05-diagnostics.md)), and user rebindings saved under the old name
are migrated or reset with a notice.
