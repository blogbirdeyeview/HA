---
name: home-assistant-blueprint-generation
description: Maintain or review the Prego Home Assistant automation and script blueprints, including motion, solar schedules, and dynamic brightness.
---

# Home Assistant blueprints

## Workflow

- Start with the [repository guide](../../../docs/repository-guide.md) and read the current YAML for the requested flow. For brightness work, check the script and its automation callers together. Use the guide to find the main and release files.
- Apply focused edits in place. Preserve the original date headers, comments, separators, and layout, including existing comments in other languages. Keep trigger IDs stable for existing instances.
- Use the [input consistency skill](../home-assistant-input-consistency/SKILL.md) when adding or editing inputs, script fields, or their shared constants.
- Match the file's Home Assistant syntax and check its `homeassistant.min_version` when using a newer feature. Use the official documentation linked in the repository guide for version-sensitive behavior.

## Style

- Use US English for new YAML text, including aliases, descriptions, and comments. Keep user-facing descriptions short and describe what the blueprint does in everyday language.
- Keep selectors narrow and use compatible defaults for optional inputs.
- Add concise, action-oriented aliases to triggers, branches, conditions, and actions where they help identify the path in a Home Assistant trace. Keep simple flows sequential and use `if` or `choose` to match the decision.

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
