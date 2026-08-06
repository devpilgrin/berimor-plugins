<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · **[English](README.en.md)** · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

</div>

# berimor-plugins

Process plugins for [berimor](https://github.com/devpilgrin/berimor). A plugin is
a standalone executable artifact with an **ACL manifest**, running in process
isolation (not in the agent's address space) and installed only from trusted
repositories with signature verification.

## Security model (summary)

1. **Signature** — sigstore (keyless, GitHub Actions OIDC); the signer's
   repository and workflow are pinned in the trust list.
2. **TOFU** — the first installation from a new repository requires explicit
   `--signer-workflow`, `--capability-ceiling`, `--allowed-ref`.
3. **ACL manifest** — the plugin's capability ceiling; anything beyond it is
   denied or requires separate confirmation (`ceiling_review`).
4. **Isolation** — the plugin runs as a separate process.

## Plugin release format

A single archive (tar.gz/zip) containing:

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

## Installation

```sh
berimor plugin install <owner>/<repo> \
  --signer-workflow https://github.com/<owner>/<repo>/.github/workflows/release.yml@refs/heads/main \
  --capability-ceiling "hello.greet" \
  --allowed-ref "v*.*.*"
```

Repeat installations from a trusted repository need no flags (the record is
already in the trust list). Trust management: `berimor trust list|add`.

## For plugin authors

A release workflow template lives in [`templates/release.yml`](templates/release.yml):
build → archive (binary + manifest.yaml) → sigstore signature → GitHub Release.
The plugin talks to berimor via the tool contract (see the main repository,
`docs/arch/` — tool dispatch contract).

## License

[Apache-2.0](LICENSE).
