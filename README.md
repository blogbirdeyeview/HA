# HA-blueprints

IA read:
SKILL.md
x-IA-info.md

- yamllint: comprobar sintaxis y buenas prácticas.
- PyYAML: parsear YAML desde scripts Python.
- ruamel.yaml: modificar YAML conservando comentarios y formato.

# Just developer (No IA)

```powershell
python -m pip install --upgrade pip
$env:PYYAML_FORCE_LIBYAML = "0"
python -m pip install --upgrade PyYAML yamllint
Remove-Item Env:PYYAML_FORCE_LIBYAML
python -m pip install --upgrade ruamel.yaml
```

```powershell
python -m yamllint --version
python -c "import yaml; import ruamel.yaml; print('PyYAML:', yaml.__version__); print('LibYAML:', yaml.__with_libyaml__); print(yaml.safe_load('enabled: true')); print('ruamel.yaml:', ruamel.yaml.__version__)"
python -m pip check
```

yamllint 1.38.0
PyYAML: 6.0.3
LibYAML: False
{'enabled': True}
ruamel.yaml: 0.19.1
No broken requirements found.
