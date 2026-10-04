# Accessibility

Many accessibility needs are met in the input layer, and the action layer ([Action mapping](04-action-mapping.md))
is what makes them possible without changing gameplay code. Players with limited mobility, one usable hand, tremors
or specialized hardware should be able to use every feature.

## What to offer

- **Full rebinding** of every action, on every device, including mouse buttons and gamepad buttons, with no action
  left unbindable.
- **One-handed layouts:** defaults that fit on one side of the keyboard, or one side of a controller.
- **Hold vs toggle** for every held action (sprint, aim, crouch): holding a button for a long time is hard for many
  people.
- **Adjustable timings:** double-tap windows, hold durations, how fast repeated presses must be ("press repeatedly
  to escape" should have an alternative).
- **Sensitivity, inversion and deadzones** per device and per axis.
- **Alternative devices:** adaptive controllers, switches, foot pedals, eye trackers and mouth-operated controllers
  appear as devices whose controls bind to the same actions.
- **No input only one device can produce:** every menu works with keyboard alone, gamepad alone and pointer alone.
- **Remappable chords:** an action that needs Shift + click should also be bindable to a single control.

## How the architecture supports it

| Need | Supported by |
|---|---|
| rebinding, one-handed layouts | bindings as data, layered over defaults |
| hold vs toggle, adjustable timings | interactions as interchangeable rules with parameters |
| sensitivity, inversion | processors on bindings |
| alternative devices | the device layer's normalization; actions don't care which device |
| prompts that match the player's setup | prompts resolved from the current binding and device |

## Settings and defaults

Accessibility options are **user settings** ([Settings](../04-settings.md)): saved per user, applied at startup,
offered early (ideally before the first gameplay), and changeable at any time without restarting.
