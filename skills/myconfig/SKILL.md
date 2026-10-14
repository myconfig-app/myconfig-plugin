---
name: myconfig
description: Work with config files that contain MyConfig tokens (`mycfg:e:1:…:end`, `mycfg:p:1:…:end`) using the myconfig CLI. Use when running a service or tests that need those values, when a config file or .env needs a secret value (new or from the user), when a plaintext secret should be encrypted, when tokens should move to another key, before committing config changes, when organizing the MyConfig app's library (adding files, making, moving, renaming or removing folders), or when the user mentions MyConfig, myconfig run, the MyConfig library, or encrypted values in .env/YAML/properties/JSON files.
---

# MyConfig CLI

MyConfig keeps secret values inside config files as tokens. The file stays readable and
committable; only the values are encrypted. The `myconfig` CLI decrypts them at the moment
they are needed, with keys from the MyConfig app (Touch ID approval), a CI key, or a password.
It can also ask the running app to create, collect and re-encrypt values, so you can set up
config files without ever seeing a secret.

**Anything printed into this conversation is sent to the model provider.** Never let a
decrypted value reach the conversation. The rules below exist for that.

## Rules

1. **Run things with `myconfig run`.** Never read the stdout of `myconfig decrypt` or
   `myconfig export`. Inside an AI agent the CLI refuses to print values (exit code 3); do not
   work around it with `--allow-agent-output`.
2. **Never handle passwords.** If a password is needed (`--password-stdin`, a CI key password),
   ask the user to run the command themselves, e.g. with Claude Code's `!` prefix:
   `! myconfig run --password-stdin --env .env -- npm test`. Do not ask the user to paste a
   password or a secret into the chat. To open a locked vault, run `myconfig unlock`; the user
   types the password in the app.
3. **Encrypting is fine to do.** When you find a plaintext secret in a config file, encrypt it
   in place by key name or position. Do not copy the value into the conversation or into a
   command line.
4. **Never ask for a secret in the chat.** When a config needs a value the user has (an API key,
   a password), run `myconfig ask` so the user types it into the MyConfig app. When it needs a new
   random secret (JWT secret, DB password you are creating), run `myconfig gen`. Both put only
   tokens into the file; you never see the value.
5. **Read and edit files directly.** Tokens are ciphertext, so reading a file is safe. Do not
   change, shorten or re-wrap a token. Before editing a file the user may have open, check
   `unsaved` in `myconfig lib ls --json`.
6. **Check before committing** config changes with `myconfig check --staged` (and
   `myconfig lint` for syntax).
7. **Key and file names are information too.** Run `myconfig keys list` or `myconfig lib ls` only
   when the task needs it (the app asks the user once before showing the library).
8. **Approvals happen in the app.** The MyConfig app shows an approval dialog (and Touch ID) for
   each request that opens, creates or changes values. If a command waits, tell the user to look
   at the MyConfig app.

## Run a program with decrypted values

Values with tokens become environment variables of the child process only.

```bash
# .env: keys whose values are tokens become environment variables
myconfig run --env .env -- npm run dev

# Spring Boot: tokens in application.yaml become SPRING_APPLICATION_JSON
myconfig run --spring src/main/resources/application.yaml -- ./gradlew bootRun
myconfig run --spring application.yaml --spring-profiles dev,local -- ./gradlew bootRun

# YAML/JSON/properties keys as environment variables (db.password -> APP_DB_PASSWORD)
myconfig run --env-from config.yaml --env-prefix APP -- python app.py

# A tool that needs a file: a decrypted temporary copy (mode 600), removed when the command ends
myconfig run --file config.json=cfg -- node server.js --config {myconfig:cfg}

# Several sources, in order (later names win)
myconfig run --env .env --env .env.local --spring application.yaml -- ./gradlew test
```

- `run` returns the child's exit code.
- Options: `--all-keys` (also keys without tokens), `--unresolved=empty|abort` (default abort),
  `--env-key-sep __`.
- In CI, `--key-file` (or `MYCONFIG_KEY`) with `--no-agent` opens that key's tokens without the app.

## Find and encrypt plaintext secrets

```bash
# Find candidates (secret-looking names whose value is not a token). Exit code 1 if found.
myconfig check
myconfig check config/ .env --json

# Encrypt one key in place (keeps encoding, BOM, line endings and file mode)
myconfig encrypt --in-place .env --key DB_PASSWORD
myconfig encrypt --in-place application.yaml --key spring.datasource.password --using prod

# Encrypt the value at a position (line:column, 1-based, as check and tokens print it).
# Works where there is no key name (Markdown notes, shell scripts). Repeatable.
myconfig encrypt --in-place notes.md --at 12:9 --at 30:15

# Encrypt every candidate in a file
myconfig encrypt --in-place .env --candidates
```

- Without `--using`, the app picks the file's or folder's default key, then the vault default.
- The app asks for approval every time it encrypts. CI can use `--key-file` instead.
- A value from somewhere else can be encrypted from stdin without showing it:
  `some-command-that-prints-the-secret | myconfig encrypt --stdin --for-file .env`
  (only if the user explicitly asks; never type the value yourself).

## Fill in secret values without seeing them

```bash
# The user types the values into the MyConfig app (up to 20 keys, waits up to 5 minutes)
myconfig ask .env --key STRIPE_SECRET_KEY --key STRIPE_WEBHOOK_SECRET --prompt "Stripe dashboard > Developers > API keys"

# The app makes random values: alnum (default, 32 chars), hex (64), base64 (44), uuid, passphrase
myconfig gen .env --key JWT_SECRET --key SESSION_SECRET
myconfig gen config/application.yaml --key spring.datasource.password --format hex --replace

# A value that comes from another command's output (only if the user asks)
some-command-that-prints-the-secret | myconfig set .env --key DB_PASSWORD --stdin
some-command-that-prints-the-secret | myconfig set .env --key DB_PASSWORD --stdin --using prod
```

- `ask`, `gen` and `set` take `--using KEY` like `encrypt`. `set` also works with a CI key
  (`--key-file`, `--no-agent`).
- A missing key is added (.env, .properties, top-level YAML/TOML keys); a missing file of those
  types is created (mode 600). For JSON and nested YAML paths, add the key to the file first
  with an empty value (`password:` in YAML) and then run the command; the value goes there.
- Keys that already have a value are not overwritten without `--replace` (`set` always replaces).
- If the file has unsaved changes in the app, the command stops (exit 1). Ask the user to save.

## Move tokens to another key

```bash
# Re-encrypt the key tokens in these files with prod (one approval with Touch ID)
myconfig rekey .env config/application.yaml --to prod

# Only tokens of one key
myconfig rekey .env --to prod --from dev
```

Values never leave the app. Password tokens and tokens the vault cannot open stay as they are.
Find the key labels with `myconfig keys list` (only when the user asks for a key change).

## Inspect files without the app

```bash
myconfig tokens .env config/           # where the tokens are: file:line:column, key, kid; no values
myconfig tokens --json config/
myconfig lint config/                  # syntax check with the app's rules (exit 1 on errors)
myconfig lint --json .env
```

Folders are scanned like `myconfig check`.

## The MyConfig app's library

```bash
myconfig lib ls                        # folders, files, real paths, default keys, open tabs
myconfig lib ls Projects/api           # one library folder
myconfig lib ls --json                 # with item IDs, linked folders and unsaved flags
myconfig lib ls --paths                # real file paths only
myconfig lib mkdir Projects/api Projects/batch   # virtual folders, parents included
myconfig lib add config/.env.staging --to Projects
myconfig lib add ~/code/api            # a folder becomes a linked folder
myconfig lib mv Projects/old.env Projects/legacy --to Archive
myconfig lib mv Archive/old.env --top  # to the top of the library
myconfig lib rename Projects/api api-server   # a virtual folder, or the shown name of a file
myconfig lib rename Projects/api-server/app.yaml --reset   # the file item shows its file name again
myconfig lib rm Projects/old.env Projects/legacy   # removes them from the library; the files stay
myconfig lib key Projects prod         # default key for new values in that folder or file
myconfig lib key Projects/api/.env --clear
myconfig open .env                     # show the file to the user in the app (not added)
```

- An item is an item ID, a library path (`Projects/api/.env`) or a real file path. Write a file in
  the current folder as `./.env`. If several items match, the command stops with exit 2; use the
  ID from `myconfig lib ls --json`.
- `--to` takes a virtual folder (not a linked folder) and applies to files only.
- Linked folders are listed by their root path only; list their files yourself.
- `lib mkdir` makes only virtual folders (nothing on disk) and not inside a linked folder. Existing
  folders are fine. Make the folder before `lib add --to`.
- `lib mv` needs `--to FOLDER` or `--top`. A folder moves with everything inside it.
- `lib rm` of a virtual folder removes everything inside it from the library. Look at it with
  `myconfig lib ls FOLDER` first, and remove only what the user asked for.
- `lib mkdir`, `lib mv`, `lib rename` and `lib rm` change the library only; real files and folders
  are never created, moved, renamed or deleted. Several paths or items take one approval.
- Virtual folder names stay unique within a folder, and nothing goes inside a linked folder. If the
  library cannot change that way (a linked folder in the way, the name is already there, a folder
  into itself, the same file already there), the command changes nothing and exits with 1; the
  message names the place. Files in one folder may share a name.
- `lib add`, `lib mkdir`, `lib mv`, `lib rename`, `lib rm` and `lib key` ask for approval with Touch ID.

Files with the same name in several places (each module's `application.yaml`): give each one a
virtual folder, so they are told apart in the app.

```bash
myconfig lib mkdir Work/api-server Work/batch
myconfig lib add api-server/src/main/resources/application.yaml --to Work/api-server
myconfig lib add batch/src/main/resources/application.yaml --to Work/batch
```

## Before committing

```bash
myconfig check --staged
```

As a git pre-commit hook (`.git/hooks/pre-commit`):

```bash
#!/bin/sh
exec myconfig check --staged
```

## Status, keys and the vault

```bash
myconfig status            # CLI version, app connection and version, vault locked/open
myconfig status --json
myconfig unlock            # show the app's unlock window (the user types the password there)
myconfig lock              # lock the vault when you are done
myconfig keys list         # only when the user asks
myconfig keys list --json
myconfig schema --json     # commands and options
```

## Exit codes

| Code | Meaning | What to do |
|---|---|---|
| 0 | OK | |
| 1 | Failed (unknown key or library item, file error, `check` found candidates, `lint` found errors, unsaved changes in the app, the library cannot change that way) | Read the message on stderr |
| 2 | Usage error, or several library items match | Fix the command; use the item ID |
| 3 | Approval or key needed (app not running, vault locked, denied, timed out, app too old, refused inside an AI agent) | Ask the user to open the MyConfig app, approve, update the app, or run the command themselves with `!` |
| other | `run` returns the child command's exit code | |

## When the app is not reachable

`myconfig status` shows the connection. Ask the user to open MyConfig and turn on
Settings > Security > "Accept command-line requests". Password tokens (`mycfg:p:`) can be
opened without the app by the user running the command with `--password-stdin`.

`lib`, `gen`, `ask`, `rekey`, `lock` and `unlock` need MyConfig app 0.8.0 or later. `lib mkdir`,
`lib mv`, `lib rename` and several items in one `lib rm` need the app and CLI 0.9.0 or later
(`myconfig status` shows both versions). If a command says the app "could not handle this request",
or the CLI does not know a `lib` command, ask the user to update MyConfig
(`brew upgrade --cask myconfig`; the app comes with the `myconfig` command line tool).
