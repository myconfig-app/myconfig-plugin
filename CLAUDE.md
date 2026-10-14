# CLAUDE.md

MyConfig CLI(`myconfig`)를 위한 AI 코딩 도구 플러그인(Claude Code·Codex CLI·Gemini CLI)의 **공개 배포 저장소**입니다.
빌드·테스트할 코드는 없고, 설정 파일과 스킬 마크다운만 있습니다.

## 원본은 비공개 저장소

플러그인 파일의 원본은 비공개 저장소 `my-configs/myconfig-flutter`의 `plugins/myconfig/`입니다
(로컬 `../../myconfig/myconfig-flutter/plugins/myconfig/`, 설계 `docs/cli.md` 6장).
**아래 파일은 여기서 고치지 않고 원본을 고친 뒤 복사합니다.** 원본 쪽 테스트(`skill_sync_test.dart`)가 스킬의
명령·옵션이 실제 CLI에 있는지 검사하기 때문입니다.

| 원본에서 복사 | 이 저장소에만 있음 |
|---|---|
| `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `gemini-extension.json`, `skills/myconfig/SKILL.md` | `.claude-plugin/marketplace.json`, `README.md`, `LICENSE`, `CLAUDE.md`, `.gitignore` |

- 버전은 `myconfig` CLI와 같습니다. 올릴 때 `marketplace.json`의 `version`도 함께 바꿉니다.
- 작성자 이메일은 넣지 않습니다.
- 공개 저장소이므로 민감 정보(실제 키·토큰·비밀번호, 개인 경로)를 올리지 않습니다.

## 로컬에서 시험

```bash
claude --plugin-dir .
```

## 사용자 설치

```bash
claude plugin marketplace add myconfig-app/myconfig-plugin
claude plugin install myconfig@myconfig
```
