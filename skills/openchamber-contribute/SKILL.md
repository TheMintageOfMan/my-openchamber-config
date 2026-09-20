---
name: openchamber-contribute
description: Use when improving OpenChamber source code in openchamber/openchamber - bug fixes, features, Electron desktop, web server, UI, VS Code extension, docs MDX, bun type-check lint test build, gh PR workflow. Use ONLY when the task touches an OpenChamber repo checkout, CONTRIBUTING.md, AGENTS.md, packages/electron, packages/web, packages/ui, packages/docs, or upstream contribution back to GitHub.
---

# OpenChamber Contribute

Improve OpenChamber at the source level, test everything locally, then
contribute back to `openchamber/openchamber` with the user's GitHub account
through `gh`. Follow good programming practices and the repo's own rules.

## 1. When to use this skill

Use this skill when the user asks to:

- fix a bug, add a feature, refactor, or clean up in the OpenChamber repo
- work on Desktop (Electron), Web server/CLI, shared UI, VS Code extension,
  mobile shell, SDK, extensions, or product docs
- validate with `bun run type-check`, `lint`, `test`, `build`, `docs:validate`
- create a branch, commit, push, and open a pull request with `gh`
- answer "how do we contribute this back upstream?"

Do NOT use this skill for:

- OpenCode itself (`../opencode` is a separate repo, read-only here)
- local OpenChamber assistant config (`~/.config/opencode/`,
  `~/.config/openchamber/`) unless the change is also an upstream code change
- generic git/GitHub work outside `openchamber/openchamber`

## 2. Sources of truth - read live, never guess

Repo rules change. Before editing, read the live files in the checkout:

1. `AGENTS.md` at repo root - always-on repo rules, runtime boundaries,
   instruction order, validation, PR handoff.
2. `CONTRIBUTING.md` at repo root - setup, dev scripts, build commands,
   code style, PR contract, review enforcement.
3. Nearest `README.md` and `DOCUMENTATION.md`:
   - `packages/electron/README.md` for anything native/desktop
   - `packages/docs/README.md` + `packages/docs/CONTRIBUTING.md` for docs
   - dynamic discovery: search `packages/**/DOCUMENTATION.md`
   - high-value anchors: `packages/ui/src/sync/DOCUMENTATION.md`,
     `packages/ui/src/stores/DOCUMENTATION.md`,
     `packages/web/bin/lib/DOCUMENTATION.md`,
     `scripts/perf/DOCUMENTATION.md`,
     `packages/vscode/src/DOCUMENTATION.md`,
     `packages/sdk/DOCUMENTATION.md`
4. Every matching project skill under `.agents/skills/*/SKILL.md`.
   Multiple skills can apply. Read every task-required reference they name.
   Key triggers:
   - source/dependency/export/contract change -> `openchamber-change-discipline`
   - CLI/prompts/non-TTY/`--json` -> `clack-cli-patterns`
   - shared UI data, OpenCode SDK, `RuntimeAPIs`, bridges -> `ui-api-decoupling`
   - Electron main/preload/IPC/updater/SSH/packaging -> `desktop-shell`
   - sync/reducers/polling/optimistic state -> `sync-state-invariants`
   - hot paths, lists, caches, lag/CPU/memory -> `performance-engineering`
   - WebSocket/SSE/relay -> `relay-transport`
   - components/styling/colors/icons -> `theme-system`
   - user-facing text/labels/aria/toasts -> `locale-ui-patterns`
   - settings UI/search -> `settings-ui-patterns`
   - drag/reorder, `@dnd-kit` -> `drag-to-reorder`
   - iOS Simulator, `serve-sim` -> `serve-sim`
   - changelog only when maintainer asks -> `update-changelog`
   - editing skills/`AGENTS.md`/agent docs -> `writing-for-agents`
   - reviewing one PR -> `pr-review`; queue triage -> `triage-prs`/`triage-issues`
   - user-facing copy -> also `communication-style`
5. `.github/PULL_REQUEST_TEMPLATE.md` - exact PR sections required.
6. `package.json` scripts - the command source of truth. Never invent
   script names.

If these sources conflict, stop and resolve the conflict with the user.
Do not start editing when a matching skill or required reference is unread.
That is a process violation upstream.

Canonical upstream URLs (fetch raw when no checkout is present):

- https://github.com/openchamber/openchamber
- https://raw.githubusercontent.com/openchamber/openchamber/main/AGENTS.md
- https://raw.githubusercontent.com/openchamber/openchamber/main/CONTRIBUTING.md
- https://raw.githubusercontent.com/openchamber/openchamber/main/packages/electron/README.md
- https://raw.githubusercontent.com/openchamber/openchamber/main/packages/docs/README.md
- https://raw.githubusercontent.com/openchamber/openchamber/main/.github/PULL_REQUEST_TEMPLATE.md

## 3. Prerequisites - Fedora Linux host

Check these before starting. Stop and fix or ask if any fail:

- Node.js 22+ required for CLI/Web/VS Code paths. Check: `node --version`.
  Verified on this host: node v24.20.0.
- Bun required for install/dev/build/test. Check: `bun --version`.
  On this host Bun was missing at first use and is now installed
  (verified bun 1.4.2). On a fresh host, install per
  https://bun.sh (user approval required, it changes system state),
  then `bun install` from repo root.
  `bun install` expected behavior: postinstall runs
  `node ./fix-deprecation.js && bun run --cwd packages/sdk build &&
  bun run extensions:build &&
  node ./packages/electron/scripts/ensure-electron.mjs --best-effort`.
  A fresh Electron install often prints
  `[electron:ensure] ... is incomplete (...); repairing...` followed by
  `repaired` - this is normal self-healing, not an error. Rough time:
  ~100 s for 1500+ packages.
- `gh` authenticated as the user's account. Check read-only:
  `gh auth status`. Never print tokens. Expected: https protocol,
  `repo` + `workflow` scopes for PRs and CI.
  Verified on this host: account TheMintageOfMan, scopes `gist`,
  `read:org`, `repo`, `workflow`. If not logged in, stop and ask
  the user to run `gh auth login`.
- Git identity set for commits (`git config user.name`, `user.email`).
  If missing, ask the user; do not invent it.
- ripgrep is NOT installed on this host: `rg` fails with
  "command not found". Use the Grep tool (not shell rg) or plain
  `grep` in scripts.
- Linux desktop packaging is native-only: x64 AppImage on x64 host,
  arm64 on arm64 host. Set `OPENCHAMBER_TARGET_ARCH` to match.
  Running AppImages needs FUSE (`libfuse.so.2`, e.g. `libfuse2`/`libfuse2t64`
  on Debian/Ubuntu) or `APPIMAGE_EXTRACT_AND_RUN=1`. Keep the AppImage on a
  writable path so in-app updates can replace it.
- Do not pin versions in commit messages or docs; verify at run time with
  `opencode --version` and `ls ~/Downloads/OpenChamber*.AppImage`.

## 4. Repo map

```
packages/
  ui/         Shared React components, hooks, stores, theme system.
              Source library only, no standalone server.
  web/        Web server (Express) + frontend (Vite) + CLI (`openchamber`).
  electron/   Electron desktop shell, native boundary only.
  vscode/     VS Code extension (extension host + webview).
  mobile/     Capacitor iOS/Android shell, connects to existing server.
  docs/       Product docs source, NOT a Bun workspace.
  sdk/        Guest contract for third-party panels (manifest, iframe
              envelope, connectHost). Import from here, do not copy types.
  extensions/ App-owned SDK extensions + build registry, NOT a Bun workspace.
```

Other important paths:

- `AGENTS.md`, `CONTRIBUTING.md`, `README.md`, `SECURITY.md`
- `.agents/skills/*/SKILL.md` - project skills (canonical workflows)
- `.github/PULL_REQUEST_TEMPLATE.md`, `.github/workflows/docs-source.yml`
- `packages/docs/content/docs/*.mdx` - English docs source of truth
- `packages/docs/content/docs/<locale>/*.mdx` - translations mirroring
  English filenames; `sidebar.config.json` - nav; `DEPLOYMENT.md` - packaging
- `changelog/unreleased.md` - READ-ONLY until maintainer asks to update
  changelog. `packages/vscode/CHANGELOG.md`, `changelog/index.json`,
  `CHANGELOG.md` are generated/legacy: never edit or regenerate them.

Runtime boundaries (from `AGENTS.md`):

- Shared UI calls official OpenCode APIs via `@opencode-ai/sdk/v2`.
  OpenChamber-owned capabilities use `RuntimeAPIs`, `runtimeFetch`, shared
  browser/realtime helpers.
- Electron starts the backend in-process (`main.mjs` imports
  `@openchamber/web/server/index.js` and calls `startWebUiServer()`).
  Never a sidecar. Packaged builds load staged assets via
  `openchamber-ui://`, loopback server stays the API backend.
- Keep domain backends in web/runtime modules unless behavior is inherently
  native. Keep entrypoints and bridges thin.
- Shared contracts must define behavior for every applicable runtime:
  web, desktop, VS Code, hosted mobile, Capacitor mobile.

## 5. Safe start - plan, branch, minimal diff

1. Outline the plan before implementation: intent, non-goals, affected
   surfaces per runtime, which skills/docs apply, validation you will run.
2. Work in a checkout of `openchamber/openchamber`, never in
   `~/.config/openchamber/` or `~/.config/OpenChamber/`.
   Checkout convention: clone under
   `~/Downloads/open_chamber_working/openchamber` (matches the
   deliverable rule in the assistant base config).
   Standard start (show exact commands, wait for explicit approval -
   git/network/file writes are state-changing):
   `git clone https://github.com/openchamber/openchamber.git`,
   `cd openchamber`, `bun install`,
   `git checkout -b <short-topic-branch>`.
   Prefer `gh repo fork` + feature branch when the user has no push access;
   never push directly to `main`.
3. Keep the change focused. Separate unrelated cleanup or refactors into
   another branch/PR.
4. Preserve unrelated worktree changes. Before overwriting any tracked file,
   create a timestamped sibling backup `<name>.YYYYMMDD-HHMM.bak` when working
   outside git, and rely on git status/diff inside the checkout.
   Never delete files without explicit approval.
5. Approval rhythm proven with this user: present commands one at a time as
   numbered "Command 1/2/3", wait for explicit approval per command
   ("Command 1 approved", "Yes", "Continue" mean go for the proposed step),
   then report output before proposing the next. Do not chain commands the
   user has not seen. Do not run git or GitHub commands for upstream work
   unless the user explicitly asked for it; this skill applies after they did.
6. Never add secrets, bearer tokens, pairing credentials, or user data to
   code, logs, screenshots, or PR text. Never add dependencies unless
   explicitly requested.

## 6. Dev loops - run from repo root unless noted

### Verified workspace script map (HEAD 449d9b42a, Sep 2026)

These are the actual root scripts observed in this checkout. They can change;
re-check `package.json` before relying on them.

- `bun run type-check` = `bun run --filter '*' type-check`. Six workspaces:
  mobile, sdk, ui, web, electron, root. Each must exit 0.
- `bun run lint` = `bun run --filter '*' lint`. Same six workspaces.
- `bun run test` = isolated test runner per package:
  `node scripts/run-isolated-tests.mjs scripts` (7 files),
  `packages/sdk test` (14), `packages/ui test` (539), `packages/vscode test`
  (46), `packages/electron test` (26), then `packages/web test` under
  vitest (235 files / ~2800 tests, ~66 s). Counts drift; the structure is
  stable.
- `bun run build` = `bun run --sequential --filter '!@openchamber/mobile'
  build` then `bun run --cwd packages/mobile build:assets`. Per workspace:
  - `packages/ui` build = `tsc --noEmit` - no `dist/` is produced.
  - `packages/sdk` build = `tsc -p tsconfig.build.json`.
  - `packages/web` build = `bun ../../scripts/build-builtin-extensions.mjs
    && vite build` (~80 s).
  - `packages/vscode` build = `build:extension` + `build:webview`.
  - `packages/electron` build = a `bun -e "process.exit(0)"` stub - the root
    build does NOT package the desktop app. Real packaging is
    `bun run electron:build`.
  - `packages/mobile` build:assets = `node scripts/prepare-web-assets.mjs`
    (stages web assets; last line of a successful root build).
- Full gate runtime on this host: install ~100 s, type-check ~15 s,
  lint ~30 s, test ~4-5 min, build ~2-3 min.

### Truncated output handling

Long commands (install, test, build) may exceed the tool output cap. The
full output is written to a file by OpenCode; do not re-run blindly. Verify
instead:

1. Exit code of the command (non-zero would be reported).
2. Grep the saved output for `error|failed|✗|ELIFECYCLE` - expect no match.
3. Check expected artifacts exist (e.g. `packages/web/dist/index.html`,
   `packages/mobile/dist`, `packages/vscode/dist/webview/assets`).
4. `git status --short --branch` must stay clean - builds never dirty
   tracked files.

Web:

- `bun run dev` - default web HMR flow, auto-selected ports.
- `bun run dev:web:full` - build watcher + Express, port 3001, manual refresh.
- `bun run dev:web:hmr` - Vite + Express API. Open the Vite URL for HMR.
  Ports 5180 (Vite), 3902 (API).
- `bun run start:web` - packaged web server, port 3000 default.
- Env overrides: `OPENCHAMBER_PORT`, `OPENCHAMBER_HMR_UI_PORT`,
  `OPENCHAMBER_HMR_API_PORT`.

Desktop (Electron):

- `bun run electron:dev` - HMR web UI + Electron shell (`main.mjs`).
- `bun run electron:dev:bundled` - Electron with built web assets.
  Use when testing closer to packaged app, startup/preload/routing/assets.
- `bun run electron:build` - full packaged app for current OS.
  Output: `packages/electron/dist`. Order: `build:web-assets`,
  `prepare:opencode-cli`, `bundle:main`, `rebuild:native`, `package.mjs`.
- Focused: `bun run --cwd packages/electron ensure:electron`,
  `bun run type-check:electron`, `bun run lint:electron`.
- If Electron binary is incomplete (missing binary, stale `dist/version`/
  `path.txt`, wrong arch), `ensure-electron.mjs --best-effort` repairs via
  postinstall under Bun. Dev launcher self-heals fail-fast.

VS Code extension:

- `bun run vscode:dev` - watch + Extension Development Host (auto-opens).
  Override with `OPENCHAMBER_VSCODE_BIN` and
  `OPENCHAMBER_VSCODE_DEV_WORKSPACE`.
- `bun run vscode:build`, `bun run vscode:package` (local `.vsix`).

Shared UI (`packages/ui`):

- No server. Source library for Web/Desktop/VS Code.
- `bun run build:ui`, `bun run type-check:ui`, `bun run lint:ui`.

Build/package:

- `bun run build` (all workspaces, BUT NOT the desktop package - see the
  verified map above; `packages/electron` build is a noop stub),
  `bun run build:web`, `bun run build:ui`,
  `bun run build:electron`, `bun run electron:build` (real packaging),
  `bun run vscode:build`, `bun run vscode:package`, `bun run pack:web`.

Docs:

- Edit `content/docs/*.mdx` (English source), mirror filenames for locales.
- Validate: `bun run docs:validate` (frontmatter `title`+`description`,
  sidebar routes resolve). Authoring: `packages/docs/CONTRIBUTING.md`.
- Deployment renders in `openchamber-website` (`apps/docs`); this repo owns
  content. Packaging via `.github/workflows/docs-source.yml`.

## 7. Good programming practices + repo constraints

- Functional React components only. TypeScript strict, no `any` without
  written justification.
- Use existing theme colors/typography from
  `packages/ui/src/lib/theme/` and `packages/ui/src/lib/typography.ts`.
  Do not add new tokens. Tailwind v4. Components support light + dark.
- Prefer early returns and `if/else`/`switch` over nested ternaries.
- Place domain logic in focused owning modules. One canonical owner per
  cross-cutting rule; companions add only a pointer + local consequence.
- Correctness invariants: prefer authoritative state over heuristics;
  derive live activity from live channels, not persisted history; scope
  fallbacks narrowly and clear them; fetch failure is never empty-success;
  make partial/rollback/cleanup/stale explicit; one failed entity must not
  erase unrelated complete entities; runtime differences must be intentional
  and visible in code.
- Security/correctness in core/runtime logic, not only UI visibility.
- IPC pattern for new native capabilities, in order:
  1. `preload.mjs` bridge only if a new renderer shape is needed,
  2. real handling in `main.mjs` under `openchamber:invoke`,
  3. gate privileged commands in main (remote pages get no local
     filesystem/shell), 4. keep shared contracts in `packages/ui`,
     server APIs in `packages/web`. Never import Electron from shared UI.
- Desktop cautions: keep OpenCode backend logic out of Electron; hidden
  Windows launches (no console flash); keep `@openchamber/web`, `bun-pty`,
  `node-pty`, native modules external in `bundle-main.mjs`; rebuild native
  modules after dep/Electron changes; test HMR + bundled UI modes.
- Update owning `DOCUMENTATION.md`/`README.md` when ownership, contracts,
  or invariants change.
- Communication: trusted colleague tone. Conclusion first, short sentences.
  User-facing text (docs, UI copy, PR/issue comments) must follow
  `.agents/skills/communication-style/SKILL.md`. Code/comments/docs in
  English. Match the maintainer's language in replies.
- Never claim a runtime/platform/relay/performance/interaction is correct
  from types or lint alone.

## 8. Validation ladder - must pass before any PR

Run the narrowest relevant checks while iterating, then the full ladder:

```bash
bun run type-check   # must pass
bun run lint         # must pass
bun run test         # must pass (all suites via scripts/run-isolated-tests.mjs)
bun run build        # must succeed
```

Practical rhythm (proven with this user): run each gate as its own command
and ask for approval between them; a green single gate (e.g. type-check only)
covers one step, not the rest. Baseline check before any change: run the
full ladder on a clean checkout once so a green baseline exists - done on
this host (Sep 20 2026): clone HEAD 449d9b42a, install, type-check, lint,
test, build all green, git status clean throughout.

- Single-file iteration: `bun test <file>`.
- Docs-only: `bun run docs:validate` may be enough.
- When files are added/deleted/renamed or exports/entrypoints/import shape
  change: `bun run dead-code` and inspect its report (non-blocking).
- On created/rewritten TS/JS: `bunx oxlint <changed-paths>` (vendored
  `anti-slop`: no unjustified assertions, no `unknown`/`object`/
  `Record<string, unknown>` contracts, no ad hoc `typeof` narrowing, no
  module mocking). Fix findings in code you authored. Do not mass-fix
  backlog elsewhere; never silence rules or launder types.
- Server JS/CLI JS/Electron helpers/native behavior need runtime checks
  (focused tests, syntax checks, builds, live runs), not just types.
- Report exactly what was and was not validated. A command name without a
  result is not evidence.
- After any build/test run, confirm `git status --short --branch` is clean
  before continuing; a dirty tree means stray output or a test wrote files.

Live run (required when user-reachable behavior changes):

- Run the built or running app, exercise the changed path, and state:
  runtime (web/desktop/VS Code/hosted mobile/Capacitor mobile), OS
  (e.g. Fedora Linux x86_64), what you saw.
- Reading the diff, passing types, and green CI are NOT a live run.
- If you genuinely cannot run it, say so and why. Honest gaps are
  reviewable; hollow claims are not.

Visual evidence (user-visible changes):

- Before + after screenshots for static states; short recording for motion,
  gestures, drag-drop, focus, multi-step flows. Must represent PR HEAD;
  refresh after implementation changes or explain why still valid.
- Responsive changes: narrow/mobile + desktop. Styling changes: light + dark.
  Include loading/empty/error/disabled/long-content/high-contrast where
  affected. Settings changes: narrow + wide pane states.
- Performance claims need before/after measurements.
- No visible change: explain concretely why the diff cannot affect rendering.
  Do not delete the section.

## 9. GitHub workflow via `gh` - test first, then upstream

Only after local validation passes and the user approves publishing:

1. Sync: `git status`, `git diff`, `git log --oneline -10`,
   `git fetch origin`, `git rebase origin/main` (or merge per user pref).
   Stage only intended files.
2. Commit focused changes. Concise message matching repo style. Never commit
   secrets. Example flow (show exact commands, one approval per command):
   `git status`, `git add <paths>`, `git commit -m "..."`, `git push -u origin <branch>`.
3. Open PR with `gh`:
   `gh pr create --title "..." --body "..."` filling every template section,
   or `gh pr create --fill` then `gh pr edit` to complete the contract.
   Never open an empty-body PR.
4. PR contract sections (see template):
   Intent, Non-goals, Affected surfaces (packages + contracts + states +
   one line per runtime: Web / Desktop / VS Code / Hosted mobile /
   Capacitor mobile, "Not applicable" allowed, blank not allowed),
   Repository guidance (table: Guidance | Why it applies | How complies -
   explain, do not just list filenames), Validation (table: Check | Result),
   Live run statement, Visual evidence, Risks and failure behavior.
5. Watch automation: `gh pr checks`, `gh pr view`, `gh pr comments`.
   Readiness labels are the signal: `review:pending`, `review:ready`
   (only ready state enters maintainer queue), `review:needs-evidence`,
   `review:blocked`, `review:human-required`, `review:automation-failed`.
   Verdicts: `PASS`, `NEEDS_EVIDENCE`, `BLOCKED`, `HUMAN_REVIEW_REQUIRED`.
   AI verdicts are advisory; address `BLOCKED` concretely, refresh evidence
   and live-run statement for `NEEDS_EVIDENCE`.
6. Iterate: push fixups, re-validate, update PR body for new HEAD, re-request
   review with `gh pr comment` or `gh pr ready` as appropriate.
7. Keep PRs active: stale after 28d no activity, closed 7d later. Comment,
   push, or add `pinned`/`security`/`help wanted` to exempt long-running PRs.
   Reopening is fine if relevant again.
8. Non-developer contributions also welcome: English issues (machine
   translation fine), device/browser/OS testing, feature ideas, Discord help.

Safety: show every `git`/`gh` command before running it. Never force-push,
skip hooks, or amend failed commits - fix and create a new commit. Never
print `gh` tokens or `relaySigningKey` material.

## 10. Quick reference

| Need | Command |
|---|---|
| Setup | `git clone https://github.com/openchamber/openchamber.git && cd openchamber && bun install` |
| Web HMR | `bun run dev` |
| Web full | `bun run dev:web:full` |
| Web Vite+API | `bun run dev:web:hmr` |
| Desktop HMR | `bun run electron:dev` |
| Desktop bundled | `bun run electron:dev:bundled` |
| Package desktop | `bun run electron:build` |
| Verify AppImage | `bun run --cwd packages/electron verify:linux-appimage` |
| VS Code dev | `bun run vscode:dev` |
| Gate | `bun run type-check && bun run lint && bun run test && bun run build` |
| Docs | `bun run docs:validate` |
| Dead code | `bun run dead-code` |
| Anti-slop | `bunx oxlint <changed-paths>` |
| GH status | `gh auth status`, `gh pr checks`, `gh pr view` |

After saving any opencode config/skill/agent change, tell the user to quit
and restart OpenCode (Apply and Restart managed server) - config loads once
at startup and is not hot-reloaded.
