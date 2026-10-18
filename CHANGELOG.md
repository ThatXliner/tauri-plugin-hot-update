# Changelog

All notable changes to `tauri-plugin-hot-update` (crate) and
`tauri-plugin-hot-update-api` (npm) are documented here. Versions of the two
packages are kept in lockstep. Format follows [Keep a Changelog](https://keepachangelog.com/),
and the project adheres to [Semantic Versioning](https://semver.org/).

## [0.2.0] - 2026-10-18

### Added

- **`apply` command / `HotUpdate::apply_staged()`: switch to a staged bundle
  without a cold launch.** The served source is no longer fixed for the
  process lifetime; `apply` promotes `staged → booting` (persisted before the
  provider switches, same ordering as boot arming) and serves the new bundle
  from the next request, so the app only has to reload its webview. On iOS and
  Android, where an app cannot relaunch itself, this removes the "force-quit
  and reopen" step. Outcomes: `applied`, `nothingStaged`, `trialPending`
  (refused while the running bundle is unacked), `notApplicable` (staged
  bundle failed the arming gates and was discarded).
- Permission `allow-apply` / `deny-apply`, included in `hot-update:default`.

### Changed

- The new bundle's trial is the ordinary one: no `notifyAppReady()` from the
  reloaded page means the same two-strike rollback on the next cold launches.
- Shells older than 0.2.0 reject `apply`; apps should fall back to
  apply-on-next-launch.

## [0.1.1] - 2026-07-16

### Changed

- **Rollback hardened from 1-strike to 2-strike.** A trial bundle must now fail
  to acknowledge (`notifyAppReady`) on **two** consecutive cold launches before
  its archive SHA-256 is blacklisted and it is rolled back. This prevents a
  good bundle from being falsely rolled back when the user quits before the ack
  fires — common on desktop, where sessions are long and launches rare. A
  genuinely broken bundle still rolls back deterministically (arm → re-arm →
  blacklist), the failure signal stays timer-free (absence of the ack across
  launches), and the ack remains independent of network and auth.
- A native/embedded frontend update that lands mid-trial (e.g. a Sparkle desktop
  update) still supersedes and discards the re-armed bundle rather than
  re-serving it, preserving the "never serve a stale frontend" invariant.

### Added

- Persisted `bootingStrikes` field in `state.json` (the unacked-launch counter).
  Backward compatible: a pre-0.1.1 state file without the field loads as `0`
  (`#[serde(default)]`).
- `resolve_boot` test coverage for prerelease-vs-release supersession
  (`<version>-ota.N` vs `<version>`) — the exact comparison a full-app update
  makes over an outstanding OTA bundle. Locks in correct SemVer precedence.

### Notes

- No public API change: the five IPC commands (`check`, `download`,
  `notifyAppReady`, `currentBundle`, `reset`) and the TypeScript wrappers are
  unchanged. The `-api` npm package is bumped only to stay in lockstep with the
  crate; it has no functional changes this release.

## [0.1.0] - 2026-07-11

### Added

- Initial release. Serves a Tauri v2 app frontend from either the embedded build
  assets or a downloaded, minisign-signed bundle, applied on the next cold
  launch, with an anti-brick rollback state machine (three-state
  staged → booting → committed pointer, monotonic version watermark, archive-hash
  blacklist).
- Signed-manifest acquisition pipeline: manifest signature verification,
  monotonic downgrade-replay protection, gated download, hardened extraction.
- Five-command IPC surface plus the `tauri-plugin-hot-update-api` TypeScript
  package.
- `hot-update-sign` CLI (behind the `cli` feature) for producing signed release
  bundles.
- iOS, Android, and desktop support (the asset-serving mechanism is
  platform-uniform).

[0.1.1]: https://github.com/yanqianglu/tauri-plugin-hot-update/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/yanqianglu/tauri-plugin-hot-update/releases/tag/v0.1.0
