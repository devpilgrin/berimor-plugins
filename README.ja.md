<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · **[日本語](README.ja.md)** · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-plugins

[berimor](https://github.com/devpilgrin/berimor) 用のプロセス型プラグイン。プラグインは
**ACL マニフェスト**を持つ独立した実行可能アーティファクトで、プロセス
分離された環境(エージェントのアドレス空間ではない)で動作し、署名検証を
経た信頼済みリポジトリからのみインストールされます。

## セキュリティモデル(概要)

1. **署名** — sigstore(keyless、GitHub Actions OIDC)。署名者のリポジトリと
   workflow がトラストリストにピン留めされます。
2. **TOFU** — 新しいリポジトリからの初回インストールでは、明示的な
   `--signer-workflow`、`--capability-ceiling`、`--allowed-ref` が必要です。
3. **ACL マニフェスト** — プラグインの能力上限。それを超えるものは拒否
   されるか、個別の確認(`ceiling_review`)が必要です。
4. **分離** — プラグインは独立したプロセスとして実行されます。

## プラグインのリリース形式

単一のアーカイブ(tar.gz/zip)で、内容は次のとおりです:

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

## インストール

```sh
berimor plugin install <owner>/<repo> \
  --signer-workflow https://github.com/<owner>/<repo>/.github/workflows/release.yml@refs/heads/main \
  --capability-ceiling "hello.greet" \
  --allowed-ref "v*.*.*"
```

信頼済みリポジトリからの再インストールにはフラグは不要です(レコードは
すでにトラストリストにあります)。信頼の管理: `berimor trust list|add`。

## プラグイン作者向け

リリース workflow のテンプレートは [`templates/release.yml`](templates/release.yml) にあります:
ビルド → アーカイブ(バイナリ + manifest.yaml)→ sigstore 署名 → GitHub Release。
プラグインはツール契約を通じて berimor と通信します(メインリポジトリの
`docs/arch/` — tool dispatch contract を参照)。

## ライセンス

[Apache-2.0](LICENSE)。
