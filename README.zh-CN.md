<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · **[简体中文](README.zh-CN.md)** · [日本語](README.ja.md) · [한국어](README.ko.md)

</div>

# berimor-plugins

[berimor](https://github.com/devpilgrin/berimor) 的进程式插件。插件是一个
带有 **ACL 清单**的独立可执行产物,在进程隔离环境中运行(不在代理的地址
空间内),并且只能从经过签名验证的可信仓库安装。

## 安全模型(摘要)

1. **签名** — sigstore(免密钥,GitHub Actions OIDC);签名者的仓库和
   workflow 会被固定(pin)在信任列表中。
2. **TOFU** — 首次从新仓库安装时必须显式指定
   `--signer-workflow`、`--capability-ceiling`、`--allowed-ref`。
3. **ACL 清单** — 插件的能力上限;超出部分将被拒绝,或需要单独确认
   (`ceiling_review`)。
4. **隔离** — 插件以独立进程运行。

## 插件发布格式

单个压缩包(tar.gz/zip),内含:

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

## 安装

```sh
berimor plugin install <owner>/<repo> \
  --signer-workflow https://github.com/<owner>/<repo>/.github/workflows/release.yml@refs/heads/main \
  --capability-ceiling "hello.greet" \
  --allowed-ref "v*.*.*"
```

从可信仓库再次安装时无需任何标志(记录已在信任列表中)。信任管理:
`berimor trust list|add`。

## 致插件作者

发布 workflow 模板见 [`templates/release.yml`](templates/release.yml):
构建 → 打包(二进制文件 + manifest.yaml)→ sigstore 签名 → GitHub Release。
插件通过工具契约与 berimor 通信(见主仓库 `docs/arch/` — tool dispatch contract)。

## 许可证

[Apache-2.0](LICENSE)。
