---
name: openchamber-v2-self-edit
description: "Guides requested edits to OpenChamber v2 desktop preferences, host settings, and its OpenCode integration on Windows, with synchronized preferences, backups, and targeted verification."
---

# Edit OpenChamber v2 settings

Use this skill for a requested OpenChamber desktop preference, host/runtime
setting, or integration change. For the assistant's `build.md`, agents, skills,
providers, permissions, or MCP definitions, use
[opencode-v2-self-edit](../opencode-v2-self-edit/SKILL.md). Do not edit generated
integration files or restart a server just because a preference changed.

## 1. Identify the owner and the intended effect

Confirm the setting, desired value, affected host, and whether the user means
the desktop interface, future sessions, the current session, or OpenCode's
server-side behavior. "Your settings" and "OpenChamber settings" refer to
OpenChamber's `settings.json` and `preferences.json`; "your system prompt" means
the canonical OpenCode `agents\build.md`, not a desktop preference.

| Requested behavior | Correct owner |
| --- | --- |
| Desktop appearance, notifications, input, sidebar, model favorites, and composer defaults | OpenChamber settings and, for synchronized fields, the preferences store |
| Binary choice | An instance setting; changes can affect the managed server lifecycle |
| Desktop host selection, local port, and window state | Electron-owned instance settings; use the desktop interface, not a generic preference write |
| Browser/install-only state | Device scope; use its supported local interface, not the profile API |
| Agent instructions, provider/MCP definitions, tool permissions, skills, and executable plugins | OpenCode configuration; use the other self-edit skill |
| Terminal themes and keybinds | OpenCode's global `cli.json`, not OpenChamber preferences |
| Existing session model or agent | Session selection; changing a default does not switch every existing session |
| Project actions and worktree setup | The selected repo's `.openchamber\project.json`, where present |
| Scheduled task or dispatched session | The live `openchamber` tool, subject to approval; do not hand-edit its persistent records |

Read the current [OpenChamber documentation](https://docs.openchamber.dev/) and
the [v2 release guidance](https://openchamber.dev/blog/opencode-v2/). For a field
not documented there, inspect the source matching the installed release at
[openchamber/openchamber](https://github.com/openchamber/openchamber), or the
installed application code, before assuming its type or behavior. Existing disk
keys alone do not prove that an option is active or exposed in the current UI.

## 2. Resolve the actual Windows layout

Use the session's working directory in the shell tool's `workdir`; use absolute
paths in PowerShell. Inspect safe environment fields only:

```powershell
$dataDir = if ($env:OPENCHAMBER_DATA_DIR) {
    $env:OPENCHAMBER_DATA_DIR
} else {
    Join-Path $env:USERPROFILE '.config\openchamber'
}
Get-ChildItem -LiteralPath $dataDir -Force
Get-Command opencode -ErrorAction Stop | Select-Object Name, Source
opencode --version
opencode debug paths
Get-Process -Name OpenChamber -ErrorAction SilentlyContinue |
    Select-Object Id, Path
```

Read the installed executable's `VersionInfo` after discovering its path. Do
not infer the installed version from a Downloads filename, assume `openchamber`
is a CLI command on PATH, or scan credential folders for configuration.

### Verified layout on this PC

Inspected on 2026-09-24: OpenChamber 2.0.0, bundled OpenCode 2.0.15, Windows 11
x64, and `OPENCHAMBER_RUNTIME=desktop`. The user profile is `C:\Users\xrahman`.
Re-discover paths and versions when applying a later change.

| Location | Purpose and editing boundary |
| --- | --- |
| `%USERPROFILE%\.config\openchamber` | OpenChamber data root; `OPENCHAMBER_DATA_DIR` can override it |
| `<data>\settings.json` | Persisted app settings, including host/device state and preference values; contains sensitive fields |
| `<data>\preferences.json` | Versioned synchronized preference fields and their update timestamps |
| `<data>\projects\*.json` | App-managed per-project records; the inspected home-project record has `version` and `scheduledTasks` |
| `%USERPROFILE%\.config\openchamber\managed-opencode\<pid>.json` | Managed server metadata; `OPENCHAMBER_MANAGED_PROCESS_REGISTRY` overrides this independent registry root |
| `<data>\opencode.managed.json` | Generated OpenCode configuration overlay; not the personal OpenCode settings file |
| `<data>\agent-tool\openchamber-agent-tool\` | Generated v2 agent-tool package, with `package.json` exporting `index.js`; do not patch it for normal settings |
| `<data>\message-queue.json` and catalog/cache files | App-managed state; do not hand-edit as a preference change |
| `<data>\chats` | Projectless-chat working directories, not transcript storage; absent when inspected; `OPENCHAMBER_CHATS_DIR` overrides new chat locations without moving existing chats |
| `%USERPROFILE%\.config\opencode` | Personal OpenCode configuration, including the canonical `agents\build.md` |
| `%APPDATA%\OpenChamber` | Electron private user data, including browser/renderer persistence; do not hand-edit |
| `%LOCALAPPDATA%\Programs\@openchamberelectron\OpenChamber.exe` | Installed desktop executable discovered on this PC |
| `%LOCALAPPDATA%\Programs\@openchamberelectron\resources\opencode-cli\opencode.exe` | Bundled OpenCode executable discovered on this PC |
| `<repo>\.openchamber\project.json` and `<repo>\.openchamber\plans` | Repo-shared files, distinct from app-managed project records |

The OpenCode data/cache/state directories come from `opencode debug paths`, not
from the OpenChamber data root. Do not assume Windows puts either application's
editable configuration under AppData. The managed-process registry has its own
override and does not move merely because `OPENCHAMBER_DATA_DIR` changes.
Repo plans can use an in-repository `plansDir` configured in `project.json`;
personal project context/plans can also live under `<data>\projects\<stem>\`.
Inspect the selected project's configuration rather than guessing the stem.

## 3. Understand the desktop integration before changing it

On this PC, `OPENCODE_CONFIG` points to `<data>\opencode.managed.json`. Its
native `plugins` entry loads `<data>\agent-tool\openchamber-agent-tool`. Personal
OpenCode settings still come from `<OpenCode config>\opencode.json(c)`; the
managed overlay is an additional source. Check the active `/api/config` source
list and `/api/plugin` runtime state rather than inferring activation from files.

There is a startup exception: if OpenChamber's parent environment already owns
an `OPENCODE_CONFIG` override, the app preserves it and adds managed plugins via
`OPENCODE_CONFIG_CONTENT` instead. Managed-tool toggles in that environment-based
mode require an approved restart. A managed child showing `OPENCODE_CONFIG` set
to `opencode.managed.json` is normal and does not itself establish this exception.
Do not print inline config while investigating which mode is active.

OpenChamber regenerates its integration files. Do not edit the overlay, bridge
package, application bundle, or a managed-server record to customize a prompt or
preference. Do not add a second bridge package or place personal instructions in
generated files. Use the owning settings interface or a user-authored native
OpenCode plugin when an explicitly requested feature actually needs code.

`desktopLocalPort` is an OpenChamber setting, not proof of the OpenCode server's
listening port. Discover the latter from the live managed record. A configured
`opencodeBinary` can select a different executable; an empty value on this PC
uses the app's normal selection. Verify the process that actually runs.

Environment variables are process-start inputs. Check the selected host and the
current runtime documentation before changing an endpoint, port, data directory,
or startup registration. Do not guess a startup command, silently change a
machine-wide environment variable, install a second server, or restart the wrong
process. Remote hosts have their own configuration and data; a local file edit
does not configure an unrelated remote host.

## 4. Keep settings and synchronized preferences consistent

Read the needed fields from both files. Never print either entire document:
`settings.json` contains relay/security material and can contain host details.
Use an allowlist of the requested safe keys; redact secrets, URLs with embedded
credentials, sensitive headers, and diagnostics before displaying them.

### Ownership, shape, and precedence

The v2 settings registry assigns each key a scope. `instance` keys belong to the
server machine and `settings.json`. `profile` keys belong to `preferences.json`
and are shared by clients of that instance. `device` keys stay in local stores
and do not go through normal profile writes. Electron-owned instance keys such
as `desktopHosts` and `desktopLocalPort` are written by the desktop shell.

The preference-file format is `version: 1`; preserve that value. `fields` maps
each preference name to an entry that can contain:

- Base `value` and `updatedAt` in Unix milliseconds.
- Optional `surfaces.desktop`, `surfaces.web`, `surfaces.vscode`, or
  `surfaces.mobile`, each with its own `value` and `updatedAt`.
- A surface-only entry without a base value. Do not invent a base for it.

For desktop reads, `fields[key].surfaces.desktop.value` wins when present;
otherwise the base `fields[key].value` applies. These preferences overlay
matching keys in `settings.json`. File modification time or the largest stamp
does not select the winning copy. Missing values leave a client's existing
store value alone; omission is not a request to reset to defaults.

The registry marks `darkThemeId` and `lightThemeId` as `perSurface`; a desktop
theme update must preserve the base and other surfaces. `showReasoningTraces`,
`notifyOnCompletion`, `defaultAgent`, `defaultModel`, and `favoriteModels` are
base profile fields in this release. `settings.json` also contains a mirror of
base profile values: keep that mirror consistent when editing a base preference,
but never overwrite it with a desktop-only value.

`updatedAt` records an accepted write, not a documented cross-host conflict
resolution protocol. An unchanged target entry keeps its stamp; a changed or
new target entry receives the current write time. Preserve unrelated entries.
Check the current registry before adding a key or assuming its scope, type,
array/object shape, or `perSurface` behavior. Unknown keys do not persist.

### Preferred live editing path

Prefer Settings or the supported merged settings interface on the selected
**OpenChamber backend**, not on the OpenCode server:

| Operation | v2 interface |
| --- | --- |
| Read effective desktop settings | `GET /api/config/settings?surface=desktop` |
| Update requested desktop settings | `PUT /api/config/settings?surface=desktop` |

The PUT body is a flat partial object containing only the changed, registered
keys and their requested values. Do not submit the `preferences.json` envelope,
a complete stale settings snapshot, or device/desktop-shell fields. Use the
appropriate surface for another client. The server validates and serializes
writes, merges against current state, and updates the correct file and surface
entries. It leaves unrelated surfaces and unchanged entries intact.

Use the selected host's authenticated connection. Show the exact origin, method,
redacted payload, and proposed change for approval before an API or UI mutation.
Being allowed to edit local config does not grant arbitrary network/UI writes.
The `openchamber` session tool does not expose arbitrary settings methods; do not
invent one. OpenCode's `/api/config` is a different contract.

The settings reader can seed a missing preferences file or normalize settings.
A GET against an uninitialized profile is therefore not guaranteed to be
filesystem-side-effect-free; inspect the files and account for those effects
before using it as a diagnostic probe. An unreadable preferences document must
not be replaced with an empty profile to force an update through.

The client debounces writes and caches GET responses; its two-second cache is
not a polling guarantee. No settings/preferences file watcher was established
in the reviewed v2 settings implementation. A direct disk edit bypasses both
client store synchronization and persistence side effects, including managed
plugin refresh after a tool toggle. Do not promise every open window will
automatically adopt a disk edit, or claim an unverified focus-refresh interval.

### Direct-file editing procedure

1. Confirm the field's scope and selected surface. Prefer the supported update
   path for live changes or settings with side effects. Avoid simultaneous edits
   in the UI or another connected client; disk saves do not update UI stores.
2. Read both files in memory, record their modification times or hashes, and
   select only the requested leaf values. Re-read if either changes before the
   patch; do not overwrite a newer client update with a stale snapshot.
3. For a base profile preference, change `fields[key].value` and its matching
   base-value mirror in `settings.json`. For a desktop-specific preference,
   change only `fields[key].surfaces.desktop`; preserve the base, other surfaces,
   and the base mirror in `settings.json`. Do not flatten the whole document.
4. Keep an unchanged entry's timestamp. Stamp only a changed/new target entry
   with `[DateTimeOffset]::UtcNow.ToUnixTimeMilliseconds()`. Do not reset every
   timestamp or fabricate a future revision. For an instance key, change its
   owning settings field without adding a profile entry. Device/shell changes
   belong to their supported interface, not a fabricated synchronized field.
5. Back up each existing file immediately before the edit, then apply narrow
   patches in the same action block. A two-file disk edit is not a transaction
   with every connected renderer; stop and use the supported update mechanism
   if consistency cannot be verified.
6. Validate JSON and the precise field type. Check the requested entry, the base
   mirror when applicable, and preservation of every unrelated surface. Verify
   the effective value through the same-surface settings read and the intended
   UI. If the UI stays stale, inspect synchronization rather than toggling the
   setting repeatedly or restarting OpenCode. Account for read-side effects.

Do not hand-edit Electron local storage, Chromium databases, cookies, or private
preference files to force a value into the UI. If a UI refresh or app restart is
truly needed, explain why and obtain approval for that exact action.

## 5. Apply the approval and backup boundary

The user's specific request pre-approves relevant file edits under the OpenChamber
data directory and OpenCode config directory. It does not authorize deletion,
installation, authentication changes, or process lifecycle actions. For other
mutations, show the exact command or tool payload and proposed diff, then wait for
explicit approval. Approval does not carry forward to the next action.

For every existing target, create `<name>.YYYYMMDD-HHMM.bak` beside it before
editing. The backup and edit belong in the same action block. Never overwrite a
backup with the same minute's name:

```powershell
$backup = '{0}.{1}.bak' -f $target, (Get-Date -Format 'yyyyMMdd-HHmm')
if (Test-Path -LiteralPath $backup) { throw 'Backup already exists; do not overwrite it.' }
[System.IO.File]::Copy($target, $backup, $false)
```

Backups of settings contain the same secrets as the originals; leave them local
and do not display or upload them. New files do not need backups. Prefer the
patch tool, preserve encoding and unrelated fields, and verify new directories
with `Get-ChildItem`. Never delete a backup, temporary file, or generated record
without explicit approval.

For repo-shared settings, obtain the normal project edit approval. Read every
action or worktree setup command before its first approved run; a pull can
change the command. Do not execute repo configuration merely to inspect it.

## 6. Verify without restarting the wrong server

OpenCode v2 hot-reloads its config, agents, commands, skills, watched plugins, and
ambient instructions. MCP edits reconcile the affected connections independently.
OpenChamber does not need to restart the managed server for ordinary OpenCode
settings saves. Desktop preference propagation is a separate mechanism; verify
it instead of assuming that the OpenCode watcher handles it.

A process environment change, binary switch/update, unwatched plugin dependency,
or failed process may require a targeted restart. Explain the affected component
and obtain approval first. Do not use `opencode service restart` as a generic
desktop refresh. OpenChamber's `POST /api/config/reload` is a lifecycle operation,
not harmless preference validation; OpenCode's `POST /api/location/reload` is a
different API. Do not call either just to inspect a saved file.

### Resolve and verify the live managed endpoint

Resolve the registry from `OPENCHAMBER_MANAGED_PROCESS_REGISTRY`, or otherwise
`%USERPROFILE%\.config\openchamber\managed-opencode`, and read its `*.json`
records. Select a record whose process is running and whose executable matches
its `binary` field. The inspected record
shape contains `pid`, `ownerPid`, `port`, `binary`, `runtime`, and `startedAt`.
Validate the owner and runtime when multiple desktop instances are present.
Do not select a record just because it is newest; stop if the target is ambiguous.

The following performs read-only discovery without assuming the registry follows
an overridden data directory:

```powershell
$registryDir = if ($env:OPENCHAMBER_MANAGED_PROCESS_REGISTRY) {
    $env:OPENCHAMBER_MANAGED_PROCESS_REGISTRY.Trim()
} else {
    Join-Path $env:USERPROFILE '.config\openchamber\managed-opencode'
}
$records = @(Get-ChildItem -LiteralPath $registryDir -Filter '*.json' |
    ForEach-Object { Get-Content -LiteralPath $_.FullName -Raw | ConvertFrom-Json })
$live = @($records | Where-Object {
    $process = Get-Process -Id $_.pid -ErrorAction SilentlyContinue
    $process -and $process.Path -eq $_.binary
})
if ($live.Count -ne 1) { throw 'Select the intended managed server before continuing.' }
$record = $live[0]
$binary = $record.binary
Get-Command $binary -ErrorAction Stop | Select-Object Name, Source
$server = 'http://127.0.0.1:' + $record.port
$raw = @(& $binary api --server $server get /api/info 2>&1)
if ($LASTEXITCODE -ne 0) { throw 'Managed API read failed; raw output withheld.' }
try { $info = ($raw -join "`n") | ConvertFrom-Json }
catch { throw 'Managed API returned non-JSON; raw output withheld.' }
if ($info.pid -ne $record.pid) { throw 'Server changed; re-read its record before continuing.' }
$info | Select-Object version, pid
```

Use the existing authenticated client/environment context. Never put a password
in a command argument, display `OPENCODE_SERVER_PASSWORD`, read credential stores,
or disable authentication to make a probe work. If the selected server requires
a sign-in not available to the client, stop and ask the user to reconnect through
the supported interface. For an explicitly selected remote host, use its
established endpoint and authentication instead of the local-record procedure.

`opencode service status` describes the CLI shared service, which can be stopped
while this desktop-managed server is healthy. Bare API/debug calls can discover
or start a different service. Always use the explicit verified `--server` for
desktop diagnosis and consult the running `/openapi.json` before choosing other
v2 routes, request bodies, or location parameters.

### Check the effective result

- **OpenCode files:** read the relevant native v2 registry and compare the
  requested agent/skill/config source with the saved file. Use the other skill's
  [verification procedure](../opencode-v2-self-edit/SKILL.md#6-verify-hot-reload-against-the-correct-server).
- **Desktop preferences:** verify the same-surface merged settings read, the
  relevant persisted entries/mirrors, and the actual UI. Confirm the intended
  desktop host; do not use another host's readback as evidence.
- **Plugins/MCP:** check actual load/connection status, not only config presence.
  Discover Code Mode tools with synchronous `search(...)` inside `execute`, await
  their calls, and use the returned exact tool paths. Skills do not install tools.
- **Failures:** preserve the files and collect only scoped, redacted diagnostics.
  Propose a minimal correction or restoration; create another sibling backup
  before restoring an existing file. Do not delete state or launch another server
  as a shortcut.

Report all changed paths and backups, assessed/total coverage, syntax checks,
persisted and live verification, and any remaining limitation. State clearly
when a file is saved but UI propagation has not been verified.

## Source references

These v2.0.0 sources were checked alongside the local installation. Re-check the
matching release when editing a newer build; do not replace published docs with
an assumed API contract.

- [Settings ownership and invariants](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/ui/src/lib/settings/DOCUMENTATION.md)
- [Registered field scopes](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/web/server/lib/opencode/settings-registry.json)
- [Preference shape, surfaces, and merge rules](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/web/server/lib/opencode/settings-files.js)
- [Settings routes](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/web/server/lib/opencode/routes.js#L248-L257)
- [Persistence and side effects](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/web/server/lib/opencode/settings-runtime.js)
- [Client caching and synchronization](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/ui/src/lib/persistence.ts)
- [Managed config and environment modes](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/web/server/lib/opencode/managed-config-file.js)
- [Independent managed-process registry](https://github.com/openchamber/openchamber/blob/v2.0.0/packages/web/server/lib/opencode/managed-process-registry.js)
- [Environment settings](https://docs.openchamber.dev/environment/) and [repository configuration](https://docs.openchamber.dev/repository-config/)
