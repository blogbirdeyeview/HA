---
name: home-assistant-blueprint-generation
description: Maintain or review the Prego Home Assistant automation and script blueprints, including motion, solar schedules, and dynamic brightness.
---

# Home Assistant blueprints

## Workflow

- Start with the [repository guide](../../../docs/repository-guide.md) and read the current YAML for the requested flow. For brightness work, check the script and its automation callers together. Use the guide to find the main and release files.
- Apply focused edits in place. Preserve the original date headers, comments, separators, and layout, including existing comments in other languages. Keep trigger IDs stable for existing instances.
- Use the [input consistency skill](../home-assistant-input-consistency/SKILL.md) when adding or editing inputs, script fields, or their shared constants.
- Consult official Home Assistant documentation and release notes when designing, modernizing, or investigating ambiguous blueprint behavior. Verify the syntax and the release that introduced each feature you adopt. Follow the [repository compatibility policy](../../../AGENTS.md#home-assistant-compatibility) when setting `homeassistant.min_version`.
- Check how the flow behaves after a restart and when a needed entity is unavailable or an action fails. Where recovery is needed, base it on current state, as the motion blueprint does by checking for motion before running SafeOff.

## Style

- Use US English for new YAML text, including aliases, descriptions, and comments. Keep user-facing descriptions short and describe what the blueprint does in everyday language.
- Keep selectors narrow and use compatible defaults for optional inputs.
- Give triggers, actions, conditions, and branches short, action-oriented aliases by default so traces show the path taken. Examples include `Run SafeOff after restart` and `Turn off targets after Home Assistant restart`. Simple steps and steps inside generated blocks can keep their default labels.
- Use `if` for a single decision and `choose` for alternative paths. Keep actions sequential unless the flow benefits from running them in parallel and the steps can safely run independently.

## Variables strategy

Follow the [repository variable naming policy](../../../AGENTS.md#blueprint-variables).

1. Place the top-level `variables:` block near the top of the blueprint. Use the policy's `_ji_` naming for direct `!input` bindings and check that each binding refers to a declared input key. Use `trigger_variables` for trigger templates and `variables` for execution templates.
2. Keep raw input bindings distinct from derived values. For example, `in_ji_motion_sensor_keep_alive` holds the configured sensor, and `var_bol_motion_sensor_keep_alive` records whether it detects motion.
3. Compute frequently reused values once in a scope shared by their consumers. Use named boolean checks in conditions, `choose` branches, and safety checks. Prefer simple, explicit templates and add a variable only when it improves readability or avoids repeated work.
4. Keep names descriptive and stable across variants. Check which steps read a variable before moving it or reassigning it inside an `if`, `choose`, or `repeat` block.

### Internal Jinja variables

Choose concise names that describe what the local holds. Replace generic names such as `helper`, `value`, `x`, `t`, or `ns` with names tied to the calculation.

Use the scope prefix on every `{% set %}` assignment, including assignments inside conditionals and loops. For example, use `temp_target_name_list.temp_entity_names` for a namespace attribute inside a `variables:` value and `internal_alarm_state` for a local in a condition template.

## Verification

- Run `git diff --check`. For YAML changes, apply the [blueprint versioning rule](../../../AGENTS.md#blueprint-versions) and check the changed blueprints with a parser that supports or preserves the Home Assistant `!input` tag. Check input references and caller/script field names when the change crosses those boundaries.
- Validate import, save, and the affected traces in Home Assistant when available. For device-specific behavior, use actual trace values and event payloads. Report static checks and live verification separately.
