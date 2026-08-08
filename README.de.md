<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · **[Deutsch](README.de.md)** · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-plugins

Prozess-Plugins für [berimor](https://github.com/devpilgrin/berimor). Ein Plugin ist
ein eigenständiges ausführbares Artefakt mit einem **ACL-Manifest**, das in
Prozess-Isolation läuft (nicht im Adressraum des Agenten) und nur aus
vertrauenswürdigen Repositories mit Signaturprüfung installiert wird.

## Sicherheitsmodell (Übersicht)

1. **Signatur** — sigstore (keyless, GitHub-Actions-OIDC); Repository und
   Workflow des Signierers werden in der Trust-Liste gepinnt.
2. **TOFU** — die erste Installation aus einem neuen Repository erfordert
   explizite `--signer-workflow`, `--capability-ceiling`, `--allowed-ref`.
3. **ACL-Manifest** — die Fähigkeits-Obergrenze des Plugins; alles darüber
   wird abgelehnt oder erfordert eine separate Bestätigung (`ceiling_review`).
4. **Isolation** — das Plugin läuft als separater Prozess.

## Plugin-Release-Format

Ein einziges Archiv (tar.gz/zip) mit folgendem Inhalt:

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

Wiederholte Installationen aus einem vertrauenswürdigen Repository benötigen
keine Flags (der Eintrag ist bereits in der Trust-Liste). Verwaltung des
Vertrauens: `berimor trust list|add`.

## Für Plugin-Autoren

Eine Vorlage für den Release-Workflow findet sich in [`templates/release.yml`](templates/release.yml):
Build → Archiv (Binärdatei + manifest.yaml) → sigstore-Signatur → GitHub Release.
Das Plugin kommuniziert mit berimor über den Tool-Vertrag (siehe Haupt-Repository,
`docs/arch/` — tool dispatch contract).

## Lizenz

[Apache-2.0](LICENSE).
