# Changelog

## 0.3.1 - 2026-06-29

Added:

- `action drag` for target-bound pointer drag/drop using refs, selectors, or
  viewport CSS pixel coordinates, with timing controls for steps, duration,
  hold, and mouse button.

Changed:

- Documented drag/drop workflows in the skill guide, command reference, and
  workflow reference.
- The runtime and Claude plugin versions are now `0.3.1`.

Validated:

- `node --check skills/realbrowser/scripts/realbrowser.mjs`
- `skills/realbrowser/scripts/realbrowser self-test`
- `git diff --check`
- Anonymous headless drag/drop smoke test against a temporary local `data:` page.

## 0.3.0 - 2026-05-14

This release moves Realbrowser from `0.2.1` to `0.3.0`. The release goal is
Claude Code support, faster and lower-token browsing, and a more stable plugin
runtime across macOS, Linux, and Windows.

Added:

- Claude Code plugin support via `.claude-plugin/plugin.json`.
- Claude Code skill layout under `skills/realbrowser/`, with the CLI, wrappers,
  skill instructions, and references colocated for plugin discovery.
- Portable launch wrappers for macOS/Linux shells, Windows `cmd.exe`, and
  PowerShell.
- `read tree`: compact ARIA accessibility tree via CDP with refs for actions.
  Use `--interactive`/`-i`, `--compact`/`-c`, `--depth`/`-d`, `--diff`/`-D`,
  and `--selector` to keep reads bounded.
- Task and session tab registries so agents reuse one same-origin tab instead
  of creating duplicate tabs after a failed action or across nested sessions.
- `tab done` and `tab close --mine` cleanup paths that only touch agent-owned
  tabs.
- `action scroll`, `read autocomplete`, and `read overlay` for generic browsing
  flows that need scroll containers, transient UI, or overlay diagnostics.
- Cross-platform focus capture/restore for explicit stopped-profile launches:
  macOS via AppleScript, Linux via `xdotool` when present, and Windows via
  PowerShell.
- Windows process-tree cleanup for anonymous browser sessions.

Changed:

- The runtime version is now `0.3.0` everywhere.
- Browsing guidance is target-first and site-agnostic: acquire one target, read
  compact state, act once, and verify with a diff or page wait.
- `SKILL.md` stays concise and delegates command/reference detail to files under
  `references/`.
- `read tree -i -c` is the default interaction reader; broad HTML, full-page
  screenshots, and large body dumps are fallback/debug tools.
- Large reads, network bodies, HAR files, screenshots, and traces are directed
  to `--out` so local tools can inspect them without spending model context.
- `tab ensure --background` uses only background-safe CDP/browser paths. It now
  probes live browserContextIds with `chrome://version`, closes mismatches, and
  refuses OS Chrome launch for already-running browser-scoped profiles.
- Signed-in profile handling prefers direct `DevToolsActivePort` WebSocket
  reuse to avoid repeated Chrome approval prompts.
- Profile ownership is monotonic once proven, preventing later label/cache
  writes from downgrading a verified tab.
- Profile routing uses `chrome://version` Profile Path readback when profile
  ownership is not otherwise proven. Wrong-profile or unverifiable probe tabs
  are closed before the requested URL is loaded.
- Session-registry hits are reusable but not automatically considered `mine`,
  so cleanup commands do not close sibling task tabs.
- Health checks report stale/dead CDP sockets accurately so callers can restart
  the daemon instead of timing out repeatedly.
- Help and workflow examples use neutral hosts and labels such as
  `example.com`, `app.example`, `app`, and `project-a`.

Fixed:

- `wait ready --screenshot <path>` no longer treats the output path as a CSS
  selector.
- `action state --root <ref>` resolves ref-based and CSS roots instead of always
  falling back to the active root.
- ARIA refs are assigned from finalized nodes, preventing ref mismatch when CDP
  backend-id pushes partially fail.
- ARIA role matching no longer misclassifies roles such as `tabpanel` and
  `tablist`.
- `action submit`, `action click`, `action hover`, `action fill`, and
  `action select` argument validation accepts explicit `--ref` where supported.
- Screenshot sizing env parsing no longer parses the same env var twice.
- `readTargetProfileDir` handles Chrome's newline-rendered `Profile Path`
  output and paths with spaces.
- CDP-created background tabs no longer fall through to OS profile launch after
  a strict profile verification failure.
- Dead code and duplicated in-page helper code were removed from the CLI.

Compatibility:

- `agents/openai.yaml` remains for Codex compatibility.
- The CLI remains a zero-dependency standalone Node script.

Validated:

- `node --check skills/realbrowser/scripts/realbrowser.mjs`
- `skills/realbrowser/scripts/realbrowser --version`
- `skills/realbrowser/scripts/realbrowser self-test`
- `git diff --check`
- Anonymous background browse smoke against `https://example.com`

## 0.2.1 - 2026-05-07

This release hardens `0.2.0` for parallel sessions and lower-approval
real-browser workflows.

Added:

- Owner-scoped labels, default context, and element refs via `--owner` /
  `REALBROWSER_OWNER`.
- Target leases for mutating commands so parallel sessions do not accidentally
  control each other's tabs.
- Owner-separated anonymous daemons while keeping real signed-in browser
  endpoints shared per physical browser endpoint.
- Local JSON state locking for labels, target metadata, leases, and owner
  default contexts.
- Daemon runtime schema checks so old same-version daemons are restarted when
  the state/lease model changes.

Changed:

- `tab ensure` skips another owner's URL match and creates a fresh tab instead
  of reusing a leased target.
- HTTP and direct WebSocket forms of the same CDP endpoint are treated as the
  same browser endpoint for daemon reuse.
- Chrome browser-wide permission grant/reset now requires `--force`, even after
  the target lease check passes.
- Full, responsive/device, and annotated screenshots/checkpoints are
  lease-guarded because they temporarily scroll, emulate, or draw page overlays.
- Expanded docs for multi-session operation, Chrome approval prompt limits, and
  release validation.

Fixed:

- Owner-scoped refs prevent one session's `b1`/`e1` reads from overwriting
  another session's pending action refs.
- Cross-owner lease detection considers duplicate/stale lease records and keeps
  the newest relevant lease.
- Broadened mutating raw CDP detection for `devtools raw`.
- Preserved the `realbrowser` name consistently across code and docs.
- Set the script version to `0.2.1`.

## 0.2.0 - 2026-05-07

This is a major CLI and workflow refactor from `0.1.0`.

Breaking changes:

- Replaced the old selected-tab/top-level command model with a target-first
  grouped CLI:
  `realbrowser [global flags] <group> <command> [args] [flags]`.
- Reads, actions, screenshots, console, network, state, dialogs, performance,
  downloads, and PDF export now require an explicit `-t <label>` target or
  `--handle`.
- Replaced old flows such as `open`, `tabs`, `observe`, `snapshot`, `console`,
  and `screenshot` with grouped commands such as `tab ensure`, `tab list`,
  `read observe`, `read snapshot`, `console list`, and `screenshot capture`.
- Removed the old split reference docs in favor of the new grouped command
  reference and workflow reference.

Added:

- Anonymous session lifecycle through `--anonymous --session <name>`.
- Signed-in profile attach and reuse through `--profile <profile>`.
- Default context via `session use`, `session list`, and `session clear`.
- Stable target handles via `handle create`, `handle list`, and
  `handle release`.
- Guarded action surface: `action state`, `action root`, `action click`,
  `action fill`, `action type`, `action press`, `action upload`,
  `action submit`, `action hover`, and `action select`.
- Console, network, dialog, performance, download, screenshot, PDF, and raw CDP
  command groups.
- Self-test coverage for parser and key runtime helpers.

Changed:

- The skill now defaults to compact reads and explicit action targets.
- Large outputs prefer `--out` files instead of stdout.
- Profile operations are more explicit about signed-in state, approval prompts,
  and foreground handoff.

Fixed:

- Reduced accidental tab mutation by requiring explicit targets.
- Reduced repeated browser launches by reusing daemon context per selected
  browser endpoint.
