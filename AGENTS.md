# Home Assistant repository

The root YAML files are Prego automation and script blueprints for Home Assistant. Start with the [repository guide](docs/repository-guide.md) to identify the active file, its variants, and any script callers.

## Input compatibility

Reuse established input keys, script field keys, and shared constants for equivalent inputs across active blueprints. Keep existing keys and shared constants stable unless the maintainer explicitly requests a change. Use consistent visible names and descriptions for equivalent behavior. Use the [input consistency skill](.agents/skills/home-assistant-input-consistency/SKILL.md) when adding or editing these definitions or reviewing variants.

## Blueprint variables

Blueprint inputs use established `in_` keys. When a template needs an input, expose it with a direct `!input` binding. Use the declared input key after `!input`.

- Name direct bindings in `variables` as `{prefix}_ji_{name}` using the input key's prefix and name. For example, write `in_ji_stealth_mode: !input in_stealth_mode`. A declared `var_<name>` input would use `var_ji_<name>`.
- Use `trigger_var_ji_<name>` for direct input bindings in `trigger_variables` and `trigger_var_` for other values read by trigger templates.
- Use `var_` for derived or normalized execution values and `var_bol_` for derived booleans.
- Give each scope its own binding when both trigger and execution templates need the same input. Use consistent names for equivalent values and remove unused variables.
- Use `temp_` for Jinja locals declared inside the value of a `variables:` definition and `internal_` for locals declared in other templates. Apply the same prefix to namespace objects, their attributes, and all reassignments.

## Blueprint versions

For each script or automation YAML you edit, increment `YY` in the `vX.YY` suffix of `blueprint.name` by one as part of the change. Keep `X` as written and zero-pad `YY` to two digits. For example, change `v5.35` to `v5.36` or `vX.87` to `vX.88` in a release file.

## Repository layout

- Keep repository-wide rules in `AGENTS.md` and reusable workflows in `.agents/skills/<name>/SKILL.md`. For blueprint and script work, use the [Home Assistant skill](.agents/skills/home-assistant-blueprint-generation/SKILL.md).
- Keep technical documentation in `docs/`. For blueprint edits, follow the [local verification steps](docs/repository-guide.md#local-verification).
- Leave `legacy/` untouched. It holds historical versions.

<!-- postman-cli:installation:start -->
Postman CLI is installed on this device, and can be used for API Engineering work.
<!-- postman-cli:installation:end -->
