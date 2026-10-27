# Native Windows Support Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Install and run OpenCode Team natively on Windows with OpenCode 1.x, preserving user configuration and existing Unix behavior.

**Architecture:** A standalone PowerShell installer manages symbolic links and version checks. A separate PowerShell memory helper mirrors the Bash helper. Dependency-free subprocess smoke tests exercise each public script; Windows CI runs them under Windows PowerShell 5.1 and PowerShell 7.

**Tech Stack:** PowerShell 5.1/7, .NET filesystem APIs, GitHub Actions, existing Bash/Ruby/VitePress checks.

**Spec:** `docs/superpowers/specs/2026-10-27-native-windows-design.md` (approved by the user).

## Global Constraints

- Support Windows PowerShell 5.1 and PowerShell 7.
- Require a successful, parseable OpenCode 1.x version for install and check.
- Never modify `opencode.json` (including JSONC).
- No new test framework, automatic elevation, machine setting changes, or paid agent executions.
- Use `OPENCODE_CONFIG_DIR` when set, otherwise `%USERPROFILE%\.config\opencode`.
- Use real symbolic links; do not add a copy, junction, or hard-link fallback.
- Preserve Bash behavior, user entries, memory entry order, and Unicode.

## Review Focus

1. Linked parent containers must not redirect writes/deletes into unrelated user directories (installer negative fixtures).
2. Relative link targets, case differences, and a repository-prefix sibling must not confuse link ownership (installer ownership fixtures).
3. Native-command failures must propagate even on Windows PowerShell 5.1 (version/npm failure fixtures).
4. Oversized unique indexes and malformed archive dates must preserve existing memory (memory failure fixtures).
5. Paths with spaces/Unicode and an alternate caller working directory must work without changing the target project (both script fixtures).

## File Responsibilities

- `install.ps1`: public install/check/uninstall/version/bootstrap interface and safe link management.
- `scripts/smoke-install.ps1`: disposable fixtures, fake commands, subprocess execution, installer assertions.
- `skills/team-memory/memory-compact.ps1`: native memory maintenance.
- `scripts/test-memory-compact.ps1`: native memory behavior and data-preservation assertions.
- `.github/workflows/checks.yml`: Windows PowerShell matrix alongside existing Unix checks.
- READMEs, install/FAQ/contribution/memory guides, `team/AGENTS.md`, `agents/orchestrator.md`, `skills/team-memory/SKILL.md`: supported commands, constraints, and runtime helper selection.
- `docs/reference/*`: regenerated from source after instruction changes.

### Task 1: PowerShell installer and lifecycle tests

**Interfaces:** `install.ps1 [-Check | -Uninstall] [-InstallOpenCode]` exits zero on success; nonzero on errors. Environment: `OPENCODE_CONFIG_DIR`, normal command lookup for `opencode` and `npm`. `-InstallOpenCode` is valid only for install. Test runner uses the current PowerShell executable for child processes.

- [ ] Write `scripts/smoke-install.ps1` with fake `opencode.cmd` returning `1.19.4`, disposable configs, and assertions that install creates the exact links, reinstall succeeds, check succeeds, uninstall preserves repository targets, and check then fails. Run before `install.ps1` exists and confirm the expected missing-script failure.
- [ ] Implement `install.ps1` using `[CmdletBinding(DefaultParameterSetName='Install')]`, switch parameters, strict error handling, normalized paths, and explicit native exit-code handling. Keep filesystem helpers private to the script; use .NET `Directory.Delete(path)` / `File.Delete(path)` for link-only deletion.
- [ ] Extend the smoke tests for unchanged JSON/JSONC, missing settings as a hint, spaces/Unicode, invocation from another working directory, and a custom config directory.
- [ ] Add collision fixtures: local skills survive and check succeeds; core local entries fail unchanged; internal wrong links are repaired; relative external/dangling links and prefix-sibling links survive; linked config child containers cannot redirect writes or uninstall.
- [ ] Add version/bootstrap fixtures: missing/unparseable/2.x/failing executable fail install/check; uninstall works without OpenCode; existing 1.x skips npm; explicit bootstrap calls `npm install --global opencode-ai@1`; missing/failing npm and failed post-install verification return nonzero. Invalid mode combinations fail.
- [ ] Implement link preflight, cleanup probes, readable actionable errors, accurate config snippet paths, and the explicit npm bootstrap needed by those fixtures. Run all installer tests under both shells; verify fixture cleanup never traverses link targets.
- [ ] Commit installer and tests: `feat: add native Windows installer`.

### Task 2: Native memory helper and regression tests

**Interfaces:** `memory-compact.ps1 [[-Role] <string>]` exits zero for successful/no-op compaction, nonzero for unsafe input or unique indexes above 60 lines. Environment: `MEMORY_DIR` or `$PWD/.team/memory`. With no role, process the same 14 roles as the Bash helper, then index; with a role, process only it, then index.

- [ ] Write `scripts/test-memory-compact.ps1` using the current shell in child processes. Assert an empty log succeeds, 40 entries remain unchanged, and 45 entries yield exactly 35 archived + 10 retained. Run before the helper exists and confirm the expected failure.
- [ ] Implement `skills/team-memory/memory-compact.ps1`: UTF-8 reads/writes, stable exact comparisons, monthly append archives, unique headers, validation before writes, and safe temporary replacements. Index threshold is 60 lines; exact duplicate removal preserves first occurrence; unique overflow fails with unchanged index.
- [ ] Add tests for repeated compaction (70 archive entries and one header), multiple months, Unicode/case-sensitive facts, 60/61-line indexes, role filtering, default CWD, custom memory path, malformed dates, unsafe role paths, and linked paths that escape the memory directory. Add no-trailing-newline and CRLF inputs.
- [ ] Run native memory tests under both shells and existing Bash memory regression tests. Confirm retained/archive entries match the intended ordering and failure fixtures preserve data.
- [ ] Commit helper and tests: `feat: support memory compaction in PowerShell`.

### Task 3: Runtime instructions, docs, and Windows CI

**Interfaces:** Windows invocation uses `powershell.exe -NoProfile -ExecutionPolicy Bypass -File <helper.ps1>` from the target project root, passing `MEMORY_DIR` when configured. Unix invocation remains the existing `.sh`. Config-root precedence is `OPENCODE_CONFIG_DIR`, then the documented default.

- [ ] Update orchestrator and memory skill instructions with concrete native Windows helper invocation and custom config/memory behavior. Update `team/AGENTS.md` installation/config precedence without changing pipeline gates or role behavior.
- [ ] Add a `windows-latest` CI job with a shell matrix (`powershell`, `pwsh`) running `scripts/smoke-install.ps1` and `scripts/test-memory-compact.ps1`. Keep existing jobs and pinned checkout action.
- [ ] Update `README.md`, `README.tr.md`, `docs/install.md`, `docs/faq.md`, `CONTRIBUTING.md`, and `docs/memory.md` with native commands, supported shells, Developer Mode/elevation, execution-policy workaround, custom config, optional npm bootstrap, and user-merged JSON snippet. Do not suggest changing machine policy.
- [ ] Regenerate `docs/reference/*` using the existing Ruby generator. Validate manifests, Bash syntax, existing install/memory tests, reference freshness, and VitePress docs build. If Ruby is absent, locate an available/portable runtime without changing machine installation; report any genuine limits.
- [ ] Commit runtime instructions, docs and CI: `docs: enable and document native Windows support`.

### Task 4: Actual binary verification and final review

**Interfaces:** Use installed OpenCode `1.19.4`; discovery only (`opencode agent list`, `opencode debug ...`), with no provider calls. Do not print provider credentials or whole user configuration.

- [ ] Install into a temporary isolated config with the actual binary and verify the 14 team agents, 19 commands, and 13 team skills are discovered using bounded output/parsed JSON. Verify default-agent/instruction resolution with a disposable `opencode.json`.
- [ ] Inspect only paths/collisions in the actual user config, run the installer there when permitted, and run `-Check`. Preserve all existing entries; never edit user JSON. Report any necessary snippet or collision without overwriting it.
- [ ] Use `requesting-code-review` for a fresh whole-branch review. Correct actionable issues and rerun the tests affected by fixes.
- [ ] Use `verification-before-completion`, inspect the final diff/status, and record exactly which checks passed or could not run. Preserve implementation in the task branch and show the runnable Windows command, any remaining manual config step, and verification limits. Do not publish or merge without a request.
