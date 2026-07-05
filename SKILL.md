---
name: home-assistant-blueprint-generation
description: Generate, refine, or review Home Assistant blueprints in YAML for domestic and local use cases.
---

# Skill: Home Assistant Blueprint Generation

## Purpose

Generate, refine, or review Home Assistant blueprints in YAML for domestic and local use cases, with a strong focus on lights, switches, plugs, motion-based flows, occupancy logic, restart-safe behavior, and user-specific customization.

These blueprints are not intended as mass-market templates for thousands of users. Optimize them for the requesting user's real home, real devices, naming conventions, and practical constraints.

## Core Principles

- Produce valid, readable, and maintainable Home Assistant blueprint YAML.
- Prefer the best available Home Assistant-native mechanisms for inputs, selectors, variables, conditions, actions, and configuration structure.
- Research the documentation carefully before deciding on the final structure when the user's request is non-trivial, ambiguous, or depends on edge-case Home Assistant behavior.
- Favor clarity and reliability over cleverness.
- Keep the blueprint practical, local, and tailored to the user's household.

## Language Rules

Use US English for all generated blueprint content.

This applies to:

- Blueprint names
- Descriptions
- Input labels
- Input descriptions
- Variable names
- Aliases
- Action labels
- Generated comments
- Revised comment text
- Any additional text written inside YAML files

Rules:

- All generated code and blueprint files must be written in US English.
- All generated comments inside blueprint files must be written in US English.
- If an existing blueprint already contains comments in another language, preserve them unless the user explicitly asks to translate or rewrite them.
- If the user asks to improve or update existing comments, rewrite them in US English unless the user requests a different language.
- Do not mix Spanish and English inside generated blueprint code or comments.
- Keep naming technically clear, short, and consistent with Home Assistant conventions.

## YAML Preservation Rules

When editing an existing `.yaml` file:

- Preserve the date comment at the top of the file exactly as it exists.
- Preserve all existing user comments.
- Never remove existing comments under any circumstance unless the user explicitly asks for that exact removal.
- Treat all existing comments as intentional, including decorative, structural, spacing, banner, separator, and inline comments.
- Preserve comment-only formatting blocks exactly as they exist whenever possible, including indentation, blank lines around them, capitalization style, and separator shape.
- Do not "clean up", compress, normalize, or rewrite comment blocks just because they look redundant or stylistic.
- Do not remove decorative separators or section comments such as:

  ```yaml
  #
  #---------------
  # input
  #---------------

  #---------------
  # variables
  #---------------

  #---------------
  # Case SAFE - Turn OFF after Home Assistant restart
  #---------------
  ```

- Decorative section comments must be preserved exactly, for example:

  ```yaml
  #---------------
  # Case SAFE - SafeOff after Home Assistant restart
  #---------------

  #---------------
  # Inputs
  #---------------
  ```

- Keep existing internal comments in place.
- Add new comments only when strictly necessary for clarity or to explain non-obvious logic.
- Never add noisy or redundant comments.
- Style or formatting comments must remain in the YAML even when they are purely organizational and have no execution impact.
- If explicitly requested, comment text may be improved or updated, but it must not be removed. Preserve the original structural style and separator format.
- If comment wording could be improved, do not remove it from the YAML. Instead, keep the original comment in the generated code and, outside the code block, optionally add a short `Improvement notes` section suggesting improved wording.
- When in doubt, keep the original comment in the YAML and place rewrite suggestions outside the generated code.

## Blueprint Design Rules

### Metadata

- Use a proper `blueprint:` block with clear `name`, `description`, and `domain`.
- Use `domain: automation` unless the user explicitly needs a script blueprint.
- Add `author` when appropriate.
- Set `homeassistant.min_version` when the blueprint depends on newer features.

### Inputs

Prefer explicit, UI-friendly inputs for every configurable part of the blueprint.

Use the most appropriate selector for each case:

- `entity` for a single entity.
- `target` when the user may choose entities, devices, or areas.
- `device` when device-level selection is truly required.
- `area` when area-based behavior is part of the design.
- `number` for thresholds, lux, brightness, delay values, timers, offsets, and percentages.
- `duration` for wait times and timeout-like values.
- `boolean` for feature toggles.
- `select` for mode choices.
- `text` only when no better structured selector exists.
- `object` with `fields` when grouped structured configuration is needed.
- `action` only when the user explicitly needs custom action injection.
- `condition` only when the user explicitly needs custom condition injection.

Rules for inputs:

- Always prefer a selector over plain free text.
- Filter selectors as much as possible to the real expected domains and device classes.
- For lights, plugs, and similar household automations, prefer narrow filtering instead of generic selectors.
- Add sensible defaults where they improve usability and reduce user error.
- Use input sections when the blueprint has enough inputs to benefit from grouping.
- If input sections are used, ensure compatibility requirements are handled correctly.
- Name input-related variables with the `in_` prefix.
- Keep `in_` names descriptive, stable, and easy to map to the original blueprint input.
- Avoid mixing input values directly into templates when the same value will be reused or normalized later.

## Variables Strategy

Blueprint inputs are not automatically template variables. When an input is used in templates, expose it through top-level variables first.

Naming convention:

- Use `in_` for variables that directly map to blueprint inputs.
- Use `var_` for derived, computed, helper, or normalized variables.
- Use `var_bol_` for derived boolean variables.
- Do not mix these prefixes.
- Keep the naming consistent throughout the blueprint.

Preferred approach:

- Create a clear `variables:` block near the top of the blueprint.
- Map each templated `!input` to a well-named `in_` variable first.
- Derive reusable helper values as `var_` variables.
- Derive boolean checks and decision flags as `var_bol_` variables.
- Normalize frequently reused values into variables instead of repeating complex templates.
- Keep variable naming descriptive and stable.
- Avoid unnecessary variable layers.

Good variable design:

- Separate raw input variables from derived variables.
- Use `in_` only for direct input mappings.
- Use `var_` for reusable logic values, transformed values, and helper expressions.
- Use `var_bol_` for readability in conditions, `choose` branches, and safety checks.
- Prefer simple, explicit templates over compact but hard-to-read template expressions.
- Be careful with scope when variables are changed inside nested blocks.

Example naming pattern:

```yaml
variables:
  in_target_light: !input target_light
  in_motion_sensor: !input motion_sensor
  in_auto_off_seconds: !input auto_off_seconds

  var_auto_off_delay: "{{ in_auto_off_seconds | int(0) }}"
  var_target_entity_id: "{{ expand(in_target_light) | map(attribute='entity_id') | list | first }}"

  var_bol_is_light_on: "{{ is_state(var_target_entity_id, 'on') }}"
  var_bol_has_valid_target: "{{ var_target_entity_id is not none }}"
```

## Conditions and Actions

- Keep `triggers`, `conditions`, and `actions` logically separated and easy to scan.
- Prefer explicit and readable `choose` branches for multi-path logic.
- Use `if` when the flow is simple and only has one main decision.
- Use `choose` when there are multiple mutually exclusive cases.
- Avoid unnecessary nesting.
- Keep restart-safe and failure-safe behavior in mind for real household automations.
- Prefer sequential actions unless parallel execution is clearly needed and safe.

### Aliases

Use `alias` on actions and major logic blocks when it improves readability.

Optional improvement rule:

- When an action contains `choose` branches with multiple conditions, add meaningful aliases to the condition branches if readability clearly benefits.
- Do not force aliases into every iteration or every small block.
- Treat this as a readability enhancement, not a mandatory rewrite.
- If the user asks for improvements, aliases should be one of the first readability upgrades to consider.

Alias style:

- Short.
- Descriptive.
- Operational.
- Example: `Check ambient lux below threshold`
- Example: `Case SAFE - Turn off after restart`
- Example: `Skip turn-on while manual override is active`

## Home Blueprint Focus

Assume most requests belong to one of these categories:

- Motion-controlled lights
- Presence-based light or plug control
- Time-window restrictions
- Lux-aware light activation
- Auto-off timers
- Manual override support
- Restart-safe recovery behavior
- Room-specific or area-specific lighting logic
- Smart plug control for local appliances

Bias design toward:

- Simple household maintenance
- Reliable behavior after restart
- Readable YAML for future manual editing
- Fast troubleshooting in traces
- Inputs that match real Home Assistant UI usage

## User-Specific Adaptation

These blueprints are personalized, not generic showcase assets.

Therefore:

- Optimize naming, defaults, and selectors for the user's environment.
- Match the user's device domains and expected entities.
- Prefer practical configuration over abstract reuse.
- Only generalize when it clearly improves the requested blueprint.
- Do not introduce broad framework-like abstractions unless the user explicitly asks for a reusable universal blueprint.

## Research Requirements

Before generating or modifying a blueprint:

1. Identify the exact Home Assistant feature set required.
2. Verify the current official documentation for schema, selectors, actions, variables, conditions, and limitations.
3. Confirm whether the requested behavior is best modeled with inputs, variables, templating, `if`, `choose`, `wait`, `repeat`, or helper entities.
4. Avoid guessing when Home Assistant syntax or behavior is version-sensitive.
5. Prefer official documentation over forum habits when both exist.
6. Use community patterns only when official docs are silent and the pattern is widely proven.

## Editing Existing Blueprints

When the user provides an existing blueprint:

- Edit in place.
- Preserve structure unless restructuring is necessary to solve the problem.
- Preserve comments, spacing intent, and section layout as much as possible.
- Do not silently remove blocks that appear intentional.
- If a structural refactor is needed, keep the same comment organization whenever possible.
- Explain major structural changes briefly and clearly.

## Output Requirements

When generating the final YAML:

- Return complete, valid YAML.
- Keep formatting clean and consistent.
- Keep comments intact.
- Keep the date comment at the top.
- Do not remove existing separators.
- Add only essential new comments.
- Prefer readability over terseness.
- Ensure the blueprint is ready to paste into Home Assistant with minimal follow-up changes.
- Use US English in all generated YAML text, including comments, aliases, labels, and descriptions.
- If comments seem improvable, preserve them in the YAML and place any comment improvement suggestion outside the code in a separate `Improvement notes` section.

## Quality Checklist

Before finalizing:

- Is the YAML valid and coherent?
- Are selectors the best available ones for each input?
- Are templated inputs exposed through variables when needed?
- Are comments preserved?
- Is the top date comment preserved?
- Are user-specific requirements reflected?
- Is the logic readable in `triggers`, `conditions`, and `actions`?
- Are aliases used only where they actually help?
- Is the blueprint practical for a real home setup?
- Is the solution robust enough for daily lights/plugs use?
- Were decorative and structural comments preserved exactly?
- Were comment improvement suggestions, if any, kept outside the generated YAML?

## Preferred Style

- Clear YAML.
- Practical logic.
- Minimal but meaningful comments.
- Strong selector usage.
- Stable variables.
- Readable `choose` / `if` flows.
- Household-first design.
- No unnecessary abstraction.
- Never remove existing comments.
