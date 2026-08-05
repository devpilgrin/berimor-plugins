# berimor-plugins

Плагины-процессы для [berimor](https://github.com/devpilgrin/berimor). Плагин —
отдельный исполняемый артефакт с **ACL-манифестом**, работающий в изоляции
процесса (не в адресном пространстве агента) и ставящийся только из
доверенных репозиториев с проверкой подписи.

## Модель безопасности (сводно)

1. **Подпись** — sigstore (keyless, OIDC GitHub Actions); репозиторий и
   workflow подписанта пинятся в trust-списке.
2. **TOFU** — первая установка с нового репозитория требует явных
   `--signer-workflow`, `--capability-ceiling`, `--allowed-ref`.
3. **ACL-манифест** — потолок возможностей плагина; всё сверх — отказ или
   отдельное подтверждение (`ceiling_review`).
4. **Изоляция** — плагин исполняется отдельным процессом.

## Формат релиза плагина

Один архив (tar.gz/zip), внутри:

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

## Установка

```sh
berimor plugin install <owner>/<repo> \
  --signer-workflow https://github.com/<owner>/<repo>/.github/workflows/release.yml@refs/heads/main \
  --capability-ceiling "hello.greet" \
  --allowed-ref "v*.*.*"
```

Повторные установки с доверенного репозитория — без флагов (запись уже в
trust-списке). Управление доверием: `berimor trust list|add`.

## Для авторов плагинов

Шаблон релизного workflow — в [`templates/release.yml`](templates/release.yml):
сборка → архив (бинарник + manifest.yaml) → подпись sigstore → GitHub Release.
Плагин общается с berimor по контракту инструментов (см. основной репозиторий,
`docs/arch/` — tool dispatch contract).

## Лицензия

[Apache-2.0](LICENSE).
