# Quillium integration pin

This fork carries the scoped cold-launch compatibility and recovery changes
proposed to upstream from `codex/scoped-cold-launch-recovery`. The fixes build on
upstream commit d102eb1a2808268f561a77db6ab293abdc70b69e and retain the original
MIT OR Apache-2.0 licenses. The plugin has no Quillium application dependency.

The `codex/quillium-ota` branch adds a verification lockfile and a disposable-key
example for integration checks. Those two files are outside the upstream PR.
Quillium pins a full commit SHA and resolves the signer and verification source
from its native Cargo dependency. This branch is not a published crate release.

Run `cargo test --locked --features cli` to verify recovery and signed acquisition.
The `fixture-key` example generates unencrypted test keys for disposable profiles;
use a dedicated encrypted key for production publication.
