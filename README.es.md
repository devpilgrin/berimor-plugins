<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · **[Español](README.es.md)** · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

</div>

# berimor-plugins

Plugins como procesos para [berimor](https://github.com/devpilgrin/berimor). Un plugin es
un artefacto ejecutable independiente con un **manifiesto ACL**, que se ejecuta
en aislamiento de proceso (no en el espacio de direcciones del agente) y se
instala únicamente desde repositorios de confianza con verificación de firma.

## Modelo de seguridad (resumen)

1. **Firma** — sigstore (keyless, OIDC de GitHub Actions); el repositorio y el
   workflow del firmante se fijan en la lista de confianza.
2. **TOFU** — la primera instalación desde un repositorio nuevo requiere
   `--signer-workflow`, `--capability-ceiling`, `--allowed-ref` explícitos.
3. **Manifiesto ACL** — el techo de capacidades del plugin; todo lo que lo
   supere se deniega o requiere una confirmación aparte (`ceiling_review`).
4. **Aislamiento** — el plugin se ejecuta como un proceso independiente.

## Formato de release de un plugin

Un único archivo (tar.gz/zip) que contiene:

```
<name>            # исполняемый файл плагина
manifest.yaml     # ACL-манифест
```

### manifest.yaml

```yaml
name: hello-tool          # [a-z0-9-]+ — валидируется до любых операций
version: 0.1.0
description: Пример плагина — приветствие как инструмент
capabilities:             # ЗАПРАШИВАЕМЫЕ возможности (потолок — trust-запись)
  tools:
    - name: hello.greet
      description: Печатает приветствие
      mutates: false
# external_effect: false  # внешние деструктивные действия — отдельная ось
```

## Instalación

```sh
berimor plugin install <owner>/<repo> \
  --signer-workflow https://github.com/<owner>/<repo>/.github/workflows/release.yml@refs/heads/main \
  --capability-ceiling "hello.greet" \
  --allowed-ref "v*.*.*"
```

Las instalaciones posteriores desde un repositorio de confianza no necesitan
flags (el registro ya está en la lista de confianza). Gestión de la confianza:
`berimor trust list|add`.

## Para autores de plugins

Hay una plantilla de workflow de release en [`templates/release.yml`](templates/release.yml):
compilación → archivo (binario + manifest.yaml) → firma sigstore → GitHub Release.
El plugin se comunica con berimor mediante el contrato de herramientas (véase el
repositorio principal, `docs/arch/` — tool dispatch contract).

## Licencia

[Apache-2.0](LICENSE).
