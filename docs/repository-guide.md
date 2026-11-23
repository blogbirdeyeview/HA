# Home Assistant repository guide

## Active blueprints

| Blueprint | Role |
| --- | --- |
| [Prego-A-TargetBy-1movement.yaml](../Prego-A-TargetBy-1movement.yaml) and [custom](../Prego-A-TargetBy-1movement-custom.yaml) | Motion-based switch and light control with an optional keep-alive sensor and restart SafeOff. The custom variant accepts configured turn-on actions. |
| [Prego-A-TargetBy-movement-sun.yaml](../Prego-A-TargetBy-movement-sun.yaml) and [custom](../Prego-A-TargetBy-movement-sun-custom.yaml) | Motion control with morning and night helpers. The custom variant declares configurable turn-on actions and unused quiet-window inputs. |
| [Prego-A-TargetBy-brightness-sun-rankTime.yaml](../Prego-A-TargetBy-brightness-sun-rankTime.yaml) | Main blueprint for motion, darkness and quiet hours, ZHA buttons, and brightness coordination. |
| [Prego-A-TargetBy-brightness-sun-rankTime_release.yaml](../Prego-A-TargetBy-brightness-sun-rankTime_release.yaml) | Release snapshot. Start regular edits in the main file and compare both files when working on a release. |
| [Prego-S-dynamic-light.yaml](../Prego-S-dynamic-light.yaml) | Script blueprint instantiated and called by the main and release automations to turn on targets. |
| [Prego-A-Config-rank-bySun.yaml](../Prego-A-Config-rank-bySun.yaml) | Writes time-only sunrise and sunset helpers using solar events, offsets, indoor LDR, and sun elevation. |
| [Prego-A-Config-lackSunlight-bySun.yaml](../Prego-A-Config-lackSunlight-bySun.yaml) | Updates a lack-of-sunlight boolean from the configured time helpers, away status, and stealth mode. |
| [Prego-A-Config-Wake-Up-Light.yaml](../Prego-A-Config-Wake-Up-Light.yaml) | Schedules configurable actions from a mobile alarm timestamp and an offset, using a date-and-time helper. |

## Motion, sun, and brightness automation

- The main automation resolves the trigger into a turn-on or turn-off request. Automatic turn-on must satisfy every enabled restriction. The darkness interval runs from sunset to sunrise, and the quiet interval blocks turn-on. The brightness window affects brightness only. The optional keep-alive sensor holds targets on during motion. The daily 02:00 shutdown and optional restart SafeOff have separate branches.
- Darkness uses the configured `in_time_morning` and `in_time_night` helpers, falling back to `input_datetime.prego_sunrise` and `input_datetime.prego_sunset` when those selections are blank. The helpers must be maintained externally. The rank-by-sun blueprint can write them when configured with the same entities.
- The ZHA button bypasses darkness and quiet hours. A single click toggles the targets, and a double click uses the script's full-brightness path for dimmable lights. The SNZB-01P command triggers share the blueprint with value-based triggers. The branch `Skip value triggers matched from the SNZB-01P` suppresses duplicates for that model.
- The automation synchronizes the selected HACS Adaptive Lighting brightness switch before routing the target action. Its brightness-window check mirrors the script for synchronization, including the low-mode bypass. The automation also turns the HACS brightness switch off for forced-brightness double clicks. The script owns the brightness applied to the targets.

## Dynamic light script

The automation calls the script instance selected in `in_dynamic_brightness_script`, falling back to `script.prego_script_dynamic_light_control_v4_18_2`. It maps `in_clock_start_brightness` and `in_clock_end_brightness` to the script's `in_clock_start_time` and `in_clock_end_time` fields. It resolves the optional `input_boolean` helper to the `in_low_brightness_mode` boolean before calling the script.

See the script reference for required fields, brightness paths, the solar-noon formula, and debug logs. [Prego-S-dynamic-light.md](Prego-S-dynamic-light.md)

## Variable scopes

Direct `!input` bindings for Jinja include `_ji_` in their names. The keep-alive trigger reads `trigger_var_ji_motion_sensor_keep_alive`, and the execution template reads `in_ji_motion_sensor_keep_alive`. Both bindings refer to `!input in_motion_sensor_keep_alive`. The derived boolean `var_bol_motion_sensor_keep_alive` holds the sensor's motion state. [Repository variable policy](../AGENTS.md#blueprint-variables) · [Variable design workflow](../.agents/skills/home-assistant-blueprint-generation/SKILL.md#variables-strategy)

## Local verification

From the repository root, parse each edited YAML with PyYAML. Replace the example filename with the file you edited. `yaml.compose` reads the YAML structure and preserves Home Assistant tags such as `!input`.

```powershell
python -c "from pathlib import Path; import sys, yaml; yaml.compose(Path(sys.argv[1]).read_text(encoding='utf-8'))" Prego-S-dynamic-light.yaml
git diff --check
```

If changes are staged, also run `git diff --cached --check`. For input or script field changes, check references against their declarations and compare the automation payload with the receiving script fields. YAML parsing alone cannot resolve those contracts. Import, save, and trace checks require a Home Assistant instance.

## References

- [Home Assistant blueprint schema](https://www.home-assistant.io/docs/blueprint/schema/) · [Selectors](https://www.home-assistant.io/docs/blueprint/selectors/) · [Creating an automation blueprint](https://www.home-assistant.io/docs/blueprint/tutorial/) · [Using automation blueprints](https://www.home-assistant.io/docs/automation/using_blueprints/)
- Community examples include [sbyx's wake-up light](https://gist.github.com/sbyx/96c43b13b90ae1c35b872313ba1d2d2d#file-wake-up-light-alarm-with-sunrise-effect-yaml), [Blackshome's smart light](https://gist.github.com/Blackshome/e6c8f1bf846bab2fa4431934a0a85770), and [Blackshome's sensor light](https://gist.github.com/Blackshome/6edfec0ff6a25c5da0d07b88dc908238).
