---
name: opencode-v2-self-edit
description: "Guides requested edits to this Windows user's OpenCode v2 configuration, build prompt, agents, commands, skills, and plugins, with backups and live verification."
---

# Edit OpenCode v2 configuration

Use this skill when the user asks to change OpenCode's own behavior or files. For
OpenChamber's desktop preferences, host selection, or generated integration, use
[openchamber-v2-self-edit](../openchamber-v2-self-edit/SKILL.md). This workflow does
not authorize installations, deletions, process restarts, or unrelated changes.

## 1. Establish the requested change and running instance

Identify the setting, desired value, and scope: personal, project, agent, or
terminal client. Ask when these are ambiguous. Inspect before editing; do not
convert unrelated configuration or change provider credentials.

Use PowerShell and absolute paths. Set the shell tool's `workdir` rather than
changing directories inside a command. Discover the executable before using it:

```powershell
Get-Command opencode -ErrorAction Stop | Select-Object Name, Source
opencode --version
opencode debug paths
$configDir = (opencode debug paths config).Trim()
Get-ChildItem -LiteralPath $configDir -Force
```

`debug paths` is local inspection; it does not start a server or open the database.
Check `$env:OPENCODE_CONFIG`, `$env:OPENCODE_CONFIG_DIR`, `$env:XDG_CONFIG_HOME`,
and the relevant data/cache/state overrides. Inspect only the needed variables;
never dump the environment or print `OPENCODE_CONFIG_CONTENT`, passwords, keys,
or authorization headers. Check whether an inline configuration is present with
`[bool]$env:OPENCODE_CONFIG_CONTENT` rather than printing its contents.

### Verified layout on this PC

The following paths were verified on 2026-09-24 with OpenCode 2.0.15. Re-run
discovery instead of treating a version, process ID, or port as permanent.
`%USERPROFILE%` is `C:\Users\xrahman` on this PC.

| Purpose | Path or ownership |
| --- | --- |
| Global config directory | `%USERPROFILE%\.config\opencode` |
| Personal server configuration | `<config>\opencode.json`; also check for `opencode.jsonc` |
| Canonical self-prompt | `<config>\agents\build.md` |
| Personal agents | `<config>\agents\<id>.md` |
| Personal slash commands | `<config>\commands\<id>.md` |
| Personal skills | `<config>\skills\<id>\SKILL.md` |
| User-authored plugin location | `<config>\plugins\<id>\`; discover or create only as needed |
| Existing optional addon | `<config>\plugin\`; contains its own skills and resources; do not reorganize it as a side effect |
| Ambient global instructions | `<config>\AGENTS.md`; present and empty when inspected; not the canonical self-prompt |
| Terminal-only preferences | `<config>\cli.json`; absent when inspected; create only for a requested terminal setting |
| Desktop-generated overlay | `%USERPROFILE%\.config\openchamber\opencode.managed.json`; selected by `OPENCODE_CONFIG` |
| Data and database | `%USERPROFILE%\.local\share\opencode`; `debug paths db` resolves the actual database |
| State | `%USERPROFILE%\.local\state\opencode` |
| Cache | `%USERPROFILE%\.cache\opencode` |
| Logs | `%USERPROFILE%\.local\share\opencode\log` |
| Approved temporary work | `%LOCALAPPDATA%\Temp\opencode` |

Use direct reads of these resolved directories; a recursive glob that skips
dot-directories is not evidence that configuration is absent. Do not inspect or
edit databases, credential stores, service registrations, caches, or installed
dependencies to implement a normal preference change.

## 2. Select the owning source, not just a matching filename

- **Personal server settings:** edit the existing global `opencode.json(c)`.
  Preserve its `$schema` and unrelated values. Do not create a second competing
  config file just to avoid understanding the first one.
- **Project server settings:** use the selected project's `opencode.json(c)` or
  `.opencode\opencode.json(c)`. Project edits require the normal approval gate.
- **Self-prompt:** "build.md", "build agent", and "your system prompt" mean
  `<config>\agents\build.md`. Do not create a home `build.md` or copy these rules
  into `AGENTS.md`.
- **Terminal preferences:** use the single global `cli.json`. Its schema is
  `https://opencode.ai/v2/cli.json`. There is no project-local CLI config.
  `OPENCODE_CLI_CONFIG_CONTENT` overlays this file. OpenChamber appearance is
  configured separately, not in `cli.json`.
- **Desktop integration:** `OPENCODE_CONFIG` currently adds OpenChamber's generated
  overlay; it does not replace the personal global config. Leave that overlay
  and its generated plugin package alone for normal OpenCode edits.

During filesystem config discovery, global configuration has lower precedence
than project documents. OpenCode searches ancestors through the filesystem root,
merging direct `opencode.json(c)` files farthest-to-nearest, then `.opencode`
config files farthest-to-nearest. Every discovered `.opencode` document therefore
overrides every discovered direct document. Additional environment-selected
sources and plugins can affect the result. Inspect the active location's
configuration-source list when precedence matters; do not infer the winning
value from one file. Config arrays have field-specific behavior: permission
rules append, and skill/plugin sources accumulate.

## 3. Use native v2 fields and current topic documentation

Read the [v2 configuration guide](https://opencode.ai/v2/docs/config) and the
relevant topic below before choosing a field. Retain
`"$schema": "https://opencode.ai/config.json"` for editor integration. Verify
that schema guidance matches the running v2 release; resolve disagreements with
the v2 topic reference and the running server's OpenAPI schemas. A JSON parse
alone does not validate configuration semantics.

| Requested change | Native v2 location and important constraints |
| --- | --- |
| Default model | Root `model` uses `provider/model`; confirm a real available ID. The root setting does not retain a `#variant`. |
| Default agent | `default_agent`; changing it does not replace the agent already selected in an existing session. |
| Agent behavior | `agents.<id>` with `system`, `mode`, `description`, `disabled`, `steps`, and `permissions`; agent `model` can use `provider/model#variant`. |
| Tool access | Ordered `permissions` rules with string `action`, `resource`, and `effect`; effects are `allow`, `ask`, or `deny`; last match wins. Agent-specific rules append. |
| Permission actions | Use `shell`, `edit`, `subagent`, `skill`, and the other documented actions. `edit` covers file writes and patches. Code Mode does not bypass nested tool permissions. |
| Providers | `providers.<id>`; custom runtime package in `package`, connection settings in `settings`, request overlays in `headers` and `body`. Consult provider/model documentation before adding options. |
| Model definitions | `providers.<id>.models.<id>`; fields include `modelID`, `capabilities`, `limit`, and `settings`; variants are an array of entries with `id` and `settings`. |
| MCP servers | `mcp.servers.<id>`; local entries require `type: "local"` and a command array; remote entries require `type: "remote"` and `url`. Use `disabled` and `timeout.startup/catalog/execution`. |
| MCP tool presentation | A server's `codemode` defaults to true; false exposes its tools directly. Do not invent tool names; use the live catalog. |
| Extra skills | `skills` is an ordered array of directories or catalog URLs. Relative skill paths resolve from the active working directory, not the config file. |
| Slash commands | `commands.<id>` with `template`, `description`, `agent`, optional model reference, and `subagent` for background delegation. |
| Plugins | `plugins` entries are package/path strings or objects with `package` and `options`. Relative plugin paths resolve from the containing config file. |
| Context retention | `compaction.auto`, `compaction.keep.tokens`, and `compaction.buffer`; use documented budgets, not guessed model limits. |
| Other common settings | `snapshots`, `media.image`, `references`, `websearch`, `worktree`, `formatter`, and global `update`; fetch the relevant guide before editing. |

For MCP, a higher-precedence definition replaces the entire server entry with
the same ID. Preserve all required connection fields in an override. Prefer
user-controlled OAuth sign-in through `/mcps` or the app. Use `{env:NAME}` for
approved secret substitutions; never place a literal credential in the file or
start an interactive authentication flow inside a hidden tool process.

The current agent `request` fields are retained but not sent by the session
runner; configure active request options on the provider, model, or variant.
The `instructions` array is accepted but does not load instruction files; use
`AGENTS.md`. OpenCode v2 does not run language servers or produce LSP diagnostics;
use the project's actual lint, typecheck, or compiler commands when verification
requires them.

Topic references:
[agents](https://opencode.ai/v2/docs/agents),
[permissions](https://opencode.ai/v2/docs/permissions),
[providers](https://opencode.ai/v2/docs/providers),
[models](https://opencode.ai/v2/docs/models),
[MCP](https://opencode.ai/v2/docs/mcp-servers),
[commands](https://opencode.ai/v2/docs/commands),
[CLI settings](https://opencode.ai/v2/docs/cli/config), and
[CLI keybinds](https://opencode.ai/v2/docs/cli/keybinds).

## 4. Edit Markdown definitions and plugins at their source

### Build prompt and agents

Read the entire selected agent file. Preserve supported frontmatter and put
system instructions in the Markdown body, not in a duplicate `system` field.
Keep `build` primary. Preserve the user's approval, backup, confidentiality,
and output rules unless the user specifically requests changing them. A later
agent definition or plugin can affect the effective agent; verify the live
`build` definition rather than assuming a successful file save proves activation.

Project instructions belong in `AGENTS.md`. The global file and upward-discovered
files are combined with the selected agent prompt, not used as replacements for
it. `OPENCODE_DISABLE_PROJECT_CONFIG=1` skips project discovery, not global
instructions. Ambient global/upward instruction edits are detected before the
next model request. Nested instructions loaded later while exploring are held
in session history; use a new session when an edit to one must apply immediately.
See [instructions](https://opencode.ai/v2/docs/instructions).

### Skills and commands

Create personal skills under `<config>\skills\<id>\SKILL.md`. Use a unique
lowercase kebab-case folder name and matching `name`, plus a quoted, task-specific
`description`. Keep an ordered workflow, prerequisites, approval boundaries,
validation, and failure handling in the entrypoint. Put supporting resources
beside it and use relative links. Do not add permission grants or automatic jobs.

The path determines the skill ID; `name` is a display label. Project skills,
explicit sources, or plugins can supply a duplicate ID. Check the live registry's
path, description, and content. Skills need a description to be advertised and
must pass the agent's `skill` permissions. Load a skill using its exact ID to test
it; do not use a filename as the tool ID. See
[skills](https://opencode.ai/v2/docs/skills).

Create personal command files under `<config>\commands\<id>.md`. Their Markdown
body is the template; frontmatter supplies fields such as `description`, `agent`,
`model`, and `subagent`. Use `.opencode\agents`, `.opencode\commands`, and
`.opencode\skills` for explicitly requested project-scoped equivalents.

### Plugins and integrations

Read [plugin configuration](https://opencode.ai/v2/docs/plugins) and the
[plugin API](https://opencode.ai/v2/docs/build/plugins) before editing executable
extensions. Native plugins use `@opencode/plugin`, `Plugin.define`, `setup(ctx)`,
domain transforms, hooks, and cleanup. HTTP integrations use `@opencode/client`
and the v2 `/api/...` contract. Do not infer hooks, exports, endpoints, or tool
signatures from a package name. Inspect the running `/openapi.json` for exact
schemas. Package installation/update and execution of new code are separate
state-changing actions; explain side effects and obtain required approval.

## 5. Back up, make the smallest edit, and validate

1. Outline the exact files and intended change. A specific requested edit under
   the OpenCode config or OpenChamber data directory is pre-approved by the
   user's standing rules; deletion is not. For other mutations, show the exact
   command and proposed diff, then wait for explicit approval for that action.
2. Read the complete target and check for concurrent changes. Prepare valid
   content before touching a watched file; preserve unrelated fields, comments,
   line endings, and encoding. Do not rewrite JSONC through a strict JSON parser
   or strip comments with a regular expression.
3. Immediately before each existing-file edit, create a timestamped sibling
   backup in the same action block. For example, after resolving `$target`:

   ```powershell
   $backup = '{0}.{1}.bak' -f $target, (Get-Date -Format 'yyyyMMdd-HHmm')
   if (Test-Path -LiteralPath $backup) { throw 'Backup already exists; do not overwrite it.' }
   [System.IO.File]::Copy($target, $backup, $false)
   ```

   Then apply the exact patch without intervening unrelated work. New files need
   no backup. Never delete or overwrite an earlier backup. Check that the source
   has not changed since it was read; stop and re-read if it has.
4. Prefer the patch tool for a narrow, reviewable edit. Verify any newly created
   directories with `Get-ChildItem`. Do not use a package manager or broad
   config-writing command merely to change a Markdown file.
5. Re-read every changed file. Parse strict JSON in memory without echoing
   secrets; use an already available JSONC/YAML-capable parser when applicable.
   Validate fields against the matching v2 schema/docs. Check Markdown
   frontmatter, IDs, every local resource link, and preservation of unrelated
   settings. Do not install a validator without approval.
6. Verify the active source and runtime result as described below. If validation
   fails, stop and propose a precise correction or restoration. Back up the
   current file before any restoration; retain all backups.

## 6. Verify hot reload against the correct server

OpenCode v2 watches configuration, agents, commands, skills, ambient instructions,
MCP definitions, and plugins in watched locations. Changes apply to subsequent
model steps; only changed MCP connections are reconciled. CLI preferences also
reload in the terminal client. Do not prescribe a routine restart after a save.
Unwatched plugin dependencies, process environment changes, binary replacement,
or an unhealthy process may require a targeted restart with approval.

This desktop's server is owned by OpenChamber, not necessarily by the CLI's
shared-service manager. `opencode service status` reporting `stopped` does not
prove that OpenChamber is disconnected. Bare API/debug commands can start or
query a different shared service. Resolve the live desktop endpoint and use its
existing authentication context; see the
[managed-server procedure](../openchamber-v2-self-edit/SKILL.md#6-verify-without-restarting-the-wrong-server).

Use these read-only routes with the verified executable and explicit server
origin. Capture stdout and stderr in memory; do not print complete API responses.

| Route | Safe verification output |
| --- | --- |
| `/api/info` | Version and process ID, matched to the managed record |
| `/api/config` | Source type, path, and property names, not configuration values |
| `/api/agent/build` | Agent ID, mode, and a content-comparison pass/fail |
| `/api/skill` | Requested skill IDs, paths, and content-comparison pass/fail |
| `/api/plugin` | Requested plugin IDs and state; inspect errors only with redaction |

For example, after resolving `$binary` and `$server`:

```powershell
$raw = @(& $binary api --server $server get /api/agent/build 2>&1)
if ($LASTEXITCODE -ne 0) { throw 'API read failed; raw output withheld.' }
$agent = ($raw -join "`n") | ConvertFrom-Json
$agent.data | Select-Object id, mode
```

`/api/config` lists source documents and discovery directories in priority order;
it is not one merged configuration object and may contain secrets. The agent,
skill, and plugin responses contain a `location` and `data`. Confirm that the
location matches the intended workspace. For another location, use the exact
`location` query schema in the running [API](https://opencode.ai/v2/docs/api),
not an assumed header. Compare the requested agent/skill body with the saved file
and check plugin/MCP runtime state when those were changed. Registration alone
does not prove a plugin or MCP connection is healthy.

Report changed absolute paths, backup paths, assessed/total files, validation
results, and any unverified behavior. Distinguish a file being saved from the
runtime actually using it. Never restart, delete service files, or touch the
database as an automatic fallback. Use
[troubleshooting](https://opencode.ai/v2/docs/troubleshooting) for a scoped failure.
