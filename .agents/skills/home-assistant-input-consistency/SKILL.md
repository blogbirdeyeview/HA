---
name: home-assistant-input-consistency
description: Validate and reuse Prego Home Assistant blueprint inputs, script fields, shared constants, UI labels, and descriptions across automation, custom, and release variants. Use when adding or editing an input or auditing naming differences.
---

# Home Assistant input consistency

Use the [repository input policy](../../../AGENTS.md#input-compatibility) and the active blueprints in the [repository guide](../../../docs/repository-guide.md).

## Find the existing contract

1. Identify what the input represents and how the YAML uses it. Check the entity domain or value type, whether the blueprint reads or writes it, and how it handles empty or omitted values.
2. Search active YAML for the exact key, then search related terms from its label, description, selector, and usage to find equivalent inputs with different keys. Inspect `blueprint.input`, including nested input sections, and top-level script `fields`. Use the current YAML as the reference.
3. Read the definitions and their call sites, including `!input`, Jinja mappings, and script action data. Compare the main blueprint with its release and custom variants when they share the input.

## Compare and verify

- Compare the input key, visible `name`, and `description` independently. For equivalent contracts, reuse the established wording and check for extra variants after ignoring YAML folding and whitespace.
- Check selector domain and type, required or optional status, default, and fallback. Reuse shared named values and fixed entity IDs when they serve the same purpose. Keep wording accurate when a blueprint adds an actual requirement or fallback.
- If several existing keys represent the same input, cite the matching blueprints and use the established key from the closest equivalent contract for a new input. Report existing conflicts with the file, key, and specific difference.
- Verify that each touched `!input` reference resolves and each touched script action payload matches the receiving `fields`. When aliases or mappings have a purpose, verify the conversion. For example, an automation can read an `input_boolean` entity and pass its state to the script as a boolean.
- Report the established key, matching definitions, and any differences in wording or behavior. If a migration is requested, verify the declarations and affected references together, including the impact on saved automation or script instances.

## Sunrise and sunset example

`in_time_morning` and `in_time_night` are the established keys for the sunrise and sunset helpers. In the main brightness blueprint and its release, the inputs read optional helpers and fall back to `input_datetime.prego_sunrise` and `input_datetime.prego_sunset`. The rank-by-sun blueprint writes to explicitly selected helpers. Reuse the keys and check the descriptions against each role.
