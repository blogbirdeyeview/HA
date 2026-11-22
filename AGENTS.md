# Home Assistant repository

## Input compatibility

Reuse established input keys, script field keys, and shared constants for equivalent inputs across active blueprints. Keep existing keys and shared constants stable unless the maintainer explicitly requests a change. Use consistent visible names and descriptions for equivalent behavior. Use the [input consistency skill](.agents/skills/home-assistant-input-consistency/SKILL.md) when adding or editing these definitions or reviewing variants.

## Blueprint versions

For each script or automation YAML you edit, increment `YY` in the `vX.YY` suffix of `blueprint.name` by one as part of the change. Keep `X` as written and zero-pad `YY` to two digits. For example, change `v5.35` to `v5.36` or `vX.87` to `vX.88` in a release file.

## Repository layout

- Keep repository-wide rules in `AGENTS.md` and reusable workflows in `.agents/skills/<name>/SKILL.md`. For blueprint and script work, use the [Home Assistant skill](.agents/skills/home-assistant-blueprint-generation/SKILL.md).
- Keep technical documentation in `docs/`. Find active files and dependencies in the [repository guide](docs/repository-guide.md).
- Leave `legacy/` untouched. It holds historical versions.

<!-- postman-cli:installation:start -->
Postman CLI is installed on this device, and can be used for API Engineering work.
<!-- postman-cli:installation:end -->
