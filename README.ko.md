<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · **[한국어](README.ko.md)**

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-plugins

[berimor](https://github.com/devpilgrin/berimor)용 프로세스 플러그인. 플러그인은
**ACL 매니페스트**를 갖춘 독립 실행형 아티팩트로, 프로세스 격리 환경(에이전트의
주소 공간이 아닌 곳)에서 실행되며, 서명 검증을 거친 신뢰된 저장소에서만
설치됩니다.

## 보안 모델(요약)

1. **서명** — sigstore(keyless, GitHub Actions OIDC). 서명자의 저장소와
   workflow가 신뢰 목록에 고정(pin)됩니다.
2. **TOFU** — 새 저장소에서의 첫 설치에는 명시적인
   `--signer-workflow`, `--capability-ceiling`, `--allowed-ref`가 필요합니다.
3. **ACL 매니페스트** — 플러그인의 기능 상한. 이를 초과하는 것은 거부되거나
   별도의 확인(`ceiling_review`)이 필요합니다.
4. **격리** — 플러그인은 별도의 프로세스로 실행됩니다.

## 플러그인 릴리스 형식

단일 아카이브(tar.gz/zip)이며, 내용은 다음과 같습니다:

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

## 설치

```sh
berimor plugin install <owner>/<repo> \
  --signer-workflow https://github.com/<owner>/<repo>/.github/workflows/release.yml@refs/heads/main \
  --capability-ceiling "hello.greet" \
  --allowed-ref "v*.*.*"
```

신뢰된 저장소에서의 재설치에는 플래그가 필요 없습니다(레코드가 이미 신뢰
목록에 있습니다). 신뢰 관리: `berimor trust list|add`.

## 플러그인 작성자를 위한 안내

릴리스 workflow 템플릿은 [`templates/release.yml`](templates/release.yml)에 있습니다:
빌드 → 아카이브(바이너리 + manifest.yaml) → sigstore 서명 → GitHub Release.
플러그인은 도구 계약(tool contract)을 통해 berimor와 통신합니다(메인 저장소의
`docs/arch/` — tool dispatch contract 참조).

## 라이선스

[Apache-2.0](LICENSE).
