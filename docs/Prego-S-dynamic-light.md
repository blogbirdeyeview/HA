# Prego Dynamic Light Control

[Prego-S-dynamic-light.yaml](../Prego-S-dynamic-light.yaml) turns on switches or lights. For dimmable lights, it chooses a brightness for each call.

## Inputs

| Script field | Value |
| --- | --- |
| `in_target_entity_control` | One entity ID or a nonempty list of IDs. Use one domain per call, either `switch` or `light`. |
| `in_enable_internal_dynamic_brightness` | Boolean that enables the dynamic calculation. |
| `in_clock_start_time`, `in_clock_end_time` | Local time window for dynamic brightness. |
| `in_min_brightness_pct`, `in_max_brightness_pct` | Brightness limits from 1% to 100%, in ascending order or equal. |
| `in_brightness_curve_gamma` | Curve exponent. The selector ranges from `0.1` to `5.0` in steps of `0.1`. The calling automation defaults it to `1.0`. |
| `in_light_transition_seconds` | Transition duration in seconds. The selector ranges from `0` to `60`. |
| `in_low_brightness_mode` | Optional boolean. Defaults to `false`. |

The caller must supply the eight required fields. The script stops with an error for missing required fields, empty targets, mixed or unsupported domains, or invalid brightness limits on the light path. It checks `supported_color_modes` on the first light to choose the path for the entire list.

## Turn-on behavior

| Target and mode | Action | Brightness and transition |
| --- | --- | --- |
| Switches | `switch.turn_on` | None |
| Lights classified as non-dimmable | `light.turn_on` | None |
| Dimmable lights with dynamic brightness enabled and either inside the window or in low-brightness mode | `light.turn_on` | Calculated brightness and the supplied transition |
| Other dimmable lights | `light.turn_on` | `brightness_pct: 100` and the supplied transition |

The local clock window includes its start and excludes its end. It can cross midnight, and equal start and end times make it empty. Low-brightness mode bypasses that window while dynamic brightness is enabled.

## Brightness calculation

The dynamic path reads `sensor.sun_next_noon`, converts it to local time, and uses its clock time as today's solar noon. It calculates the signed difference from the current time in hours, subtracting 24 hours above `+12` and adding 24 hours below `−12`. Low-brightness mode triples the absolute offset before applying the curve.

```text
darkness_factor = abs(hours_from_solar_noon) * darkness_multiplier
raw_pct = 100 - darkness_factor * (100 / 12)
normalized = clamp(raw_pct / 100, 0, 1)
curved = normalized ** gamma
brightness = min_pct + curved * (max_pct - min_pct)
brightness_pct = clamp(round(brightness), 1, 100)
```

The multiplier is `3` in low-brightness mode and `1` otherwise. Gamma `1.0` gives a linear curve. Higher gamma lowers brightness between the same endpoints, and lower gamma raises it. The script calculates one target per call. The transition sets how long the light takes to reach the requested level.

For a range of 20% to 100%, these examples use the same formula.

| Hours from solar noon | Low mode | Gamma | Brightness |
| ---: | --- | ---: | ---: |
| 6 | Off | 1.0 | 60% |
| 6 | Off | 2.0 | 40% |
| 6 | Off | 3.0 | 30% |
| 2 | On | 2.0 | 40% |
| 4 | On | 2.0 | 20% |

## Diagnostics

The light branches write debug entries starting with `Prego dynamic light calculation:`. The dynamic entry includes the limits, gamma, darkness multiplier, solar-noon offset, intermediate values, and final percentage. The other light branches log either `calculated=100%` or `calculated=unchanged`.
