# Non-Linear Brightness Curve Enhancement

## Goal

Enhance the existing dynamic brightness calculation by replacing its linear solar-noon curve with an optional non-linear curve.

The implementation must keep the current `sensor.sun_next_noon` reference and must not switch to sunrise, sunset, solar elevation, or a clock-only calculation.

This automation coexists with HACS Adaptive Lighting. Adaptive Lighting may control the light before the configured clock window begins. This enhancement only affects the dynamic brightness calculation when the existing dynamic control is enabled and the current time is inside its configured clock range.

## Current Behavior

The automation calculates brightness once when it runs.

`transition: 2` only smooths the change from the current brightness to the calculated target brightness over two seconds. It does not continuously recalculate or gradually adjust brightness during later hours.

The current brightness model is linear:

```text
raw = 100 - abs(hours_from_solar_noon) * (100 / 12)

brightness = min_brightness + (raw / 100) * (max_brightness - min_brightness)
```

This produces a symmetric triangular curve:

- Maximum brightness at solar noon.
- Brightness decreases at a constant rate as time moves away from solar noon.
- Minimum brightness near 12 hours before or after solar noon.

The existing clock window behavior must remain unchanged:

- Dynamic control enabled and current time inside the configured range: calculate and apply dynamic brightness.
- Dynamic control enabled but outside the configured range: use the existing `100%` behavior.
- Dynamic control disabled: do not send a new brightness value.

## Required Enhancement

Add one optional blueprint input:

```yaml
in_brightness_curve_gamma:
  name: Brightness curve gamma
  description: >
    Controls the non-linear brightness curve.
    1.0 keeps the current linear behavior.
    Values above 1.0 dim faster as time moves away from solar noon.
  default: 1.0
  selector:
    number:
      min: 0.5
      max: 4.0
      step: 0.1
      mode: box
```

The default value must be `1.0` to preserve the exact existing behavior for all current blueprint instances unless the user explicitly changes the setting.

## New Brightness Formula

Keep the existing `raw` value calculation.

Normalize `raw` to a value between `0` and `1`:

```text
normalized = clamp(raw / 100, 0, 1)
```

Apply a power curve using the configurable gamma value:

```text
curved = normalized ^ gamma
```

Map the curved value to the configured brightness range:

```text
brightness = min_brightness + curved * (max_brightness - min_brightness)
```

Finally, round and clamp the result to a valid integer brightness percentage.

## Expected Gamma Behavior

|       Gamma | Result                                                                                     |
| ----------: | ------------------------------------------------------------------------------------------ |
|       `1.0` | Current linear behavior. No functional change.                                             |
|       `1.5` | Moderately faster dimming away from solar noon.                                            |
|       `2.0` | Clearly faster dimming away from solar noon. Recommended starting point for hallways.      |
|       `3.0` | Aggressive dimming; lights become dim early in the evening.                                |
| Below `1.0` | Keeps more brightness for longer. Valid but likely less useful for hallway-style lighting. |

For example, if `normalized` is `0.5`:

```text
gamma = 1.0 -> curved = 0.50
gamma = 2.0 -> curved = 0.25
gamma = 3.0 -> curved = 0.125
```

With `min_brightness = 20` and `max_brightness = 100`:

```text
gamma = 1.0 -> brightness = 60
gamma = 2.0 -> brightness = 40
gamma = 3.0 -> brightness = 30
```

## Template Implementation

Use this pattern where the existing dynamic `brightness_pct` is calculated:

```jinja
{% set min_brightness = min_brightness | float %}
{% set max_brightness = max_brightness | float %}
{% set gamma = in_brightness_curve_gamma | float(1.0) %}

{% set normalized = [0, [raw / 100, 1] | min] | max %}
{% set curved = normalized ** gamma %}
{% set brightness = min_brightness + curved * (max_brightness - min_brightness) %}

{{ [1, [brightness | round(0) | int, 100] | min] | max }}
```

Replace `min_brightness` and `max_brightness` with the actual existing variable names in the blueprint.

The normalization and final clamping are required:

- `normalized` must stay between `0` and `1` before applying the exponent.
- The final `brightness_pct` must stay between `1` and `100`.
- Avoid returning `0` because `brightness_pct: 0` may turn the light off instead of setting a dim level.

## Constraints

- Keep `sensor.sun_next_noon` as the solar-noon reference.
- Do not introduce sunrise, sunset, solar elevation, or external astronomical calculations.
- Do not add a forced low-brightness time period such as `02:00` to `05:00`.
- Do not change the existing clock-window conditions or their behavior outside the window.
- Do not change Adaptive Lighting configuration or behavior.
- Do not add periodic recalculation triggers. Brightness must still be calculated only when the current automation or script runs.
- Keep the existing transition behavior unchanged.
- Preserve backwards compatibility with `gamma: 1.0`.

## Intended Usage

Use the default `gamma: 1.0` for rooms that should preserve the current linear brightness behavior.

Use a higher gamma value, such as `2.0` or `3.0`, for hallways or passage areas where the light should already be noticeably dim at the beginning of the configured evening window.