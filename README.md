# MyConfig Plugin for AI Coding Agents

A skill for Claude Code, Codex CLI and Gemini CLI that works with config files whose secret values are
encrypted by [MyConfig](https://myconfig.app/). Your agent can run services, encrypt plaintext secrets
and fill in new values with the `myconfig` command line tool, without a decrypted value ever entering
the conversation.

## What the agent does with it

- Runs services and tests with decrypted values in their environment only (`myconfig run`).
- Finds plaintext secrets and encrypts them in place, and checks staged changes before a commit.
- Never asks you for a secret in the chat: you type it into the MyConfig app (`myconfig ask`), or the
  app makes a random one (`myconfig gen`). Only encrypted tokens go into the file.
- Moves values to another key, and works with the MyConfig app's library.

Every request that opens, creates or changes values is approved by you in the MyConfig app (with Touch ID).

## Prerequisites

The MyConfig app and the `myconfig` command line tool, 0.8.0 or later (macOS, Apple silicon).
Organizing the library from the agent (`lib mkdir`, `lib mv`, `lib rename`, several items in one
`lib rm`) needs 0.9.0 or later:

```bash
brew tap myconfig-app/myconfig
brew trust --tap myconfig-app/myconfig
brew install --cask myconfig-cli    # installs the app first if needed
```

Then open MyConfig and turn on Settings > Security > **Accept command-line requests**.
Check the connection with `myconfig status`.

## Install

### Claude Code

```bash
claude plugin marketplace add myconfig-app/myconfig-plugin
claude plugin install myconfig@myconfig
```

### OpenAI Codex CLI

```bash
codex plugin marketplace add myconfig-app/myconfig-plugin
codex plugin install myconfig
```

### Google Gemini CLI

```bash
gemini extensions install https://github.com/myconfig-app/myconfig-plugin
```

## Update

```bash
claude plugin update myconfig
codex plugin upgrade myconfig
gemini extensions update myconfig
```

The plugin version follows the `myconfig` command line tool. Update both together.

## Uninstall

```bash
claude plugin uninstall myconfig
codex plugin uninstall myconfig
gemini extensions uninstall myconfig
```

## Issues

This repository is published from MyConfig's private source repository, so pull requests can't be
merged here. Bug reports and suggestions are welcome as issues.

## License

Apache 2.0. See [LICENSE](LICENSE).

## 한국어

[MyConfig](https://myconfig.app/)로 비밀값을 암호화한 설정 파일을 AI 코딩 도구(Claude Code, Codex CLI,
Gemini CLI)가 다룰 수 있게 하는 스킬입니다. 서비스 실행, 평문 비밀값 암호화, 새 값 채우기를 `myconfig`
명령줄 도구로 하며, 풀린 값은 대화에 들어가지 않습니다.

- 서비스·테스트는 풀린 값을 환경변수로만 받아 실행합니다(`myconfig run`).
- 평문 비밀값을 찾아 그 자리에서 암호화하고, 커밋 전에 검사합니다.
- 비밀값을 대화로 묻지 않습니다. 값은 MyConfig 앱 입력 창에 직접 넣고(`myconfig ask`), 새 값은 앱이
  만듭니다(`myconfig gen`). 파일에는 암호화된 토큰만 들어갑니다.
- 값을 푸는 요청, 새로 만들거나 바꾸는 요청은 모두 MyConfig 앱에서 승인합니다(Touch ID).

### 준비

MyConfig 앱과 `myconfig` 명령줄 도구 0.8.0 이상(macOS, Apple silicon). 에이전트가 라이브러리를 정리하는 명령
(`lib mkdir`, `lib mv`, `lib rename`, 여러 항목 `lib rm`)은 0.9.0 이상:

```bash
brew tap myconfig-app/myconfig
brew trust --tap myconfig-app/myconfig
brew install --cask myconfig-cli    # 앱이 없으면 앱부터 설치
```

MyConfig를 열고 설정 > 보안 > **명령줄 도구 요청 받기**를 켭니다. `myconfig status`로 연결을 확인합니다.

### 설치

```bash
# Claude Code
claude plugin marketplace add myconfig-app/myconfig-plugin
claude plugin install myconfig@myconfig

# Codex CLI
codex plugin marketplace add myconfig-app/myconfig-plugin
codex plugin install myconfig

# Gemini CLI
gemini extensions install https://github.com/myconfig-app/myconfig-plugin
```

플러그인 버전은 `myconfig` 명령줄 도구를 따릅니다. 함께 업데이트하세요.
이 저장소는 비공개 원본 저장소에서 내보낸 결과물이라 풀 요청은 받지 않고, 이슈로 문제와 제안을 받습니다.
