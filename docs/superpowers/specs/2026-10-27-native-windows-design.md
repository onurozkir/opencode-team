# Native Windows support (issue #42)

## Intent and decisions

A Windows user with OpenCode 1.x must be able to install and use OpenCode Team from PowerShell. The user is blocked at the documented `./install.sh` step, not by a demonstrated OpenCode runtime failure. Their installed `opencode --version` was independently confirmed as `1.19.4`.

The user accepts enabling Windows Developer Mode or running an elevated PowerShell to create symbolic links. Use real symbolic links, matching the existing Bash installer, so repository edits take effect without reinstalling. Do not add a copy, junction, or hard-link fallback.

The following are proposed implementation choices, not additional user requirements: support Windows PowerShell 5.1 and PowerShell 7; add a PowerShell memory compactor because the orchestrator currently invokes a Bash script at every gate; use npm only for the explicitly requested optional OpenCode installation. These choices are subject to approval of this design.

## Installer behavior

Add `install.ps1` at the repository root with mutually exclusive `-Check` and `-Uninstall` modes, default installation, and an optional `-InstallOpenCode` switch for installation mode. Document commands that work from a freshly cloned repository, including a process-scoped execution-policy workaround when needed; never change machine execution policy or enable Developer Mode automatically.

Use `OPENCODE_CONFIG_DIR` when set, otherwise `%USERPROFILE%\.config\opencode`. Resolve paths independently of the caller's working directory and handle spaces, Unicode and Windows path separators. Link `agents`, `commands`, `team.yaml`, `team/AGENTS.md`, and each repository-local `skills/team-*` directory to their corresponding repository targets. Keep the containing `skills` and `team` directories as real directories.

Require a successful, parseable OpenCode 1.x version for install and check. An existing valid 1.x installation is reused. If installation is requested and OpenCode is missing or incompatible, run the officially documented npm distribution with a major-version constraint (`npm install --global opencode-ai@1`), check its exit status, and verify the resulting executable/version. Without that switch, never install or replace OpenCode. Explain missing npm, unsupported versions, and missing symlink privileges with actionable errors. Uninstall does not require OpenCode.

Never modify `opencode.json` (including JSONC). Print the recommended `default_agent` and `instructions` snippet when needed, using paths for the actual config directory. The user merges that snippet into their own configuration; this work does not change provider or model settings.

## Existing files and ownership

Repeated installation succeeds when expected links already exist. A link pointing elsewhere inside this repository may be corrected. An ordinary existing core file/directory or a core link outside the repository causes a clear failure and is preserved. Existing local or externally managed `skills/team-*` entries are skipped and preserved, matching the Bash installer.

Check validates that core links resolve to the exact expected targets, required targets exist, and required local skills are available; local/external skill collisions remain informational. A missing recommended JSON setting is informational, not a failed check.

Uninstall removes only links managed by this repository, never their targets, external links, ordinary files, or user configuration. Determine ownership with normalized absolute paths and directory boundaries, including relative link targets and case-insensitive Windows comparisons. Reject unexpected linked parent containers before writing through them. Preflight predictable collisions and link-creation capability before changing installation entries, and clean up any preflight probe. No recursive deletion of installed directory links.

## Native memory operation

Add `skills/team-memory/memory-compact.ps1` with the existing script's contract: optional role; `MEMORY_DIR` override or current project's `.team/memory`; logs above 40 entries archive all but the newest 10 into monthly role archives; append without repeating archive headers. Remove exact duplicate index lines only when the index exceeds 60 lines. If it still exceeds 60 unique lines, fail without changing the index. Preserve Unicode and original entry order. Validate role/path-derived names before writes and keep writes within the selected memory directory.

Update the orchestrator and memory skill to select the PowerShell helper on native Windows and the existing Bash helper on macOS/Linux. Respect the configured global config directory and the existing project memory-directory setting when describing invocation. Preserve the pipeline, roles, memory format, and human approval gates.

## Tests and documentation

Add dependency-free PowerShell smoke tests using disposable config directories and fake OpenCode/npm commands, with no network or real user configuration writes. Cover installation, repeat installation, check, uninstall, missing/bad versions, optional installation, unchanged JSON, custom config paths, spaces, Unicode, local skill collisions, in-repository relinking, external links and parent-container protection. Include memory tests for empty logs, thresholds, repeated/monthly archives, Unicode, current-project paths, custom memory directories, exact duplicate removal, and an unchanged oversized unique index.

Add a `windows-latest` smoke job to `.github/workflows/checks.yml`; exercise the PowerShell installer and memory helper under the supported PowerShell versions. Keep Ubuntu/macOS jobs. Update both READMEs, install guide, FAQ, contribution instructions, and affected memory/runtime instructions. Regenerate reference pages if source manifests change. Describe symlink privileges, config paths, execution policy, and manual JSON configuration accurately.

## Verification and completion

Run the Windows smoke and memory tests locally under the available PowerShell versions. Use the actual OpenCode 1.19.4 binary with an isolated configuration to verify agent/config discovery without paid model requests. Run `ruby scripts/validate.rb`, Bash syntax checks and existing shell/memory smoke tests where their required runtimes are available, and build the documentation. Report any unavailable runtime/check explicitly instead of claiming it passed. A configured Windows CI job is not evidence of a completed remote CI run.

After isolated tests, install into the user's actual configuration within the authorized scope and run the installer check, preserving existing entries and JSON. If local collisions or permissions prevent installation, retain all existing data and report the precise remaining step. Provide the concrete JSON snippet if the orchestrator default/instructions need manual configuration.

## Scope and builder discretion

No OpenCode 2 migration, WSL requirement, new test framework, provider authentication changes, paid agent executions, automatic elevation, machine setting changes, publishing, issue comments, or PR merging. Existing Bash behavior stays compatible. This is a Windows support change, not a rewrite of the pipeline or documentation toolchain.

The builder chooses function organization, test fixture structure, safe .NET/PowerShell filesystem APIs, and the exact failure messages. Do not broaden scope solely to install contributor tools; use available or portable runtimes and document genuine verification limits.
