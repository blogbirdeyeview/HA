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
- Keep selectors narrow and use compatible defaults for optional inputs. Expose `!input` values as variables when templates need them.
- Use `in_` for variables mapped directly from inputs, `var_` for derived values, and `var_bol_` for derived booleans. For Jinja `{% set %}`, use descriptive `temp_` names in values under `variables:` and `internal_` names elsewhere, including namespace attributes.
- Add concise, action-oriented aliases to triggers, branches, conditions, and actions where they help identify the path in a Home Assistant trace. Keep simple flows sequential and use `if` or `choose` to match the decision.

## Verification

- Run `git diff --check`. For YAML changes, apply the [blueprint versioning rule](../../../AGENTS.md#blueprint-versions) and check the changed blueprints with a parser that supports or preserves the Home Assistant `!input` tag. Check input references and caller/script field names when the change crosses those boundaries.
- Validate import, save, and the affected traces in Home Assistant when available. For device-specific behavior, use actual trace values and event payloads. Report static checks and live verification separately.
