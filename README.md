# Prego · Blueprints para Home Assistant

Automatizaciones de movimiento, horarios solares y despertador, y un script de iluminación que calcula el brillo. La [guía del repositorio](docs/repository-guide.md) identifica los blueprints activos, sus variantes y qué automatizaciones llaman al script.

## Trabajar con Codex

Abre Codex en la raíz del repositorio. [AGENTS.md](AGENTS.md) recoge las reglas de compatibilidad y versionado y enlaza las dos skills locales para mantener blueprints y revisar inputs compartidos.

## Entorno de desarrollo

Las [comprobaciones locales](docs/repository-guide.md#local-verification) usan PyYAML para leer las etiquetas de Home Assistant. También puedes usar yamllint para revisar el estilo y ruamel.yaml para editar YAML conservando comentarios. Si necesitas preparar estas herramientas en tu entorno Python, ejecuta en PowerShell:

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
