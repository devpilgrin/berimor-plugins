<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · **[Français](README.fr.md)** · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

</div>

# berimor-plugins

Plugins sous forme de processus pour [berimor](https://github.com/devpilgrin/berimor). Un plugin est
un artefact exécutable autonome doté d'un **manifeste ACL**, qui s'exécute en
isolation de processus (pas dans l'espace d'adressage de l'agent) et ne
s'installe que depuis des dépôts de confiance avec vérification de signature.

## Modèle de sécurité (résumé)

1. **Signature** — sigstore (keyless, OIDC GitHub Actions) ; le dépôt et le
   workflow du signataire sont épinglés dans la liste de confiance.
2. **TOFU** — la première installation depuis un nouveau dépôt exige des
   options explicites `--signer-workflow`, `--capability-ceiling`, `--allowed-ref`.
3. **Manifeste ACL** — le plafond de capacités du plugin ; tout dépassement
   est refusé ou nécessite une confirmation séparée (`ceiling_review`).
4. **Isolation** — le plugin s'exécute dans un processus séparé.

## Format de release d'un plugin

Une seule archive (tar.gz/zip) contenant :

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

Les installations suivantes depuis un dépôt de confiance ne nécessitent aucun
flag (l'enregistrement est déjà dans la liste de confiance). Gestion de la
confiance : `berimor trust list|add`.

## Pour les auteurs de plugins

Un modèle de workflow de release se trouve dans [`templates/release.yml`](templates/release.yml) :
build → archive (binaire + manifest.yaml) → signature sigstore → GitHub Release.
Le plugin communique avec berimor via le contrat d'outils (voir le dépôt
principal, `docs/arch/` — tool dispatch contract).

## Licence

[Apache-2.0](LICENSE).
