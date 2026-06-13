# Build · Scripts · CI · Skills · Docs — File Index
> Scope: build/config, Scripts/, .github/workflows, .agents/skills, docs/, AppBundle/AppResources. 96 entries.

## Subsystem role
RepoPrompt CE is a SwiftPM macOS app (no `.xcodeproj`) that builds two executables — the `RepoPrompt` app and the `repoprompt-mcp` server CLI — packaged into a `.app` bundle by shell scripts rather than Xcode. The build/run/test/release surface is fronted by a coordinated developer daemon, the **conductor** (`./conductor` → `Scripts/conductor.py`), which serializes jobs across named lanes (build, debugArtifact, liveApp, release, style) so concurrent agents never build, launch, or test over each other; the plain `make`/`swift`/`Scripts` paths are the uncoordinated fallback. CI on `macos-26` runners gates every PR/push with guardrails, lint, conductor/release self-tests, and build+test, while `pull_request_target`/`issues` workflows enforce an allowlist-based contribution gate. Release tooling implements two lanes (secret-free ad-hoc candidate; maintainer Developer-ID-signed + notarized + Sparkle-EdDSA-signed publish/promote) with extensive fail-closed validation of universal architectures, packaged legal notices, and MCP socket ownership. Three repo-local skills (`rpce-contribution-check`, `rpce-merge-pr-batch`, `rpce-release`) codify the contributor preflight, PR-batch merge, and release maintainer workflows.

## Repo-root build & config , Scripts/ , .github/workflows/ , .agents/skills/ , docs/ , App bundle resources

### Repo-root build & config
- `Package.swift` — SwiftPM manifest (swift-tools 6.2, macOS 14+); declares `RepoPrompt` app + `repoprompt-mcp` products, pins all deps (Sparkle, KeyboardShortcuts, MCP swift-sdk, tree-sitter grammars, SwiftAnthropic/SwiftOpenAI, etc.), and local `Vendor/`/`Packages/` packages.
- `Package.resolved` — resolved dependency lockfile.
- `Makefile` — developer entrypoints; `doctor/setup/build/run/test/lint/guardrails`, conductor/release self-tests, and the coordinated `dev-*` targets that route through the conductor daemon.
- `conductor` — thin bash launcher that execs `python3 Scripts/conductor.py "$@"`.
- `version.env` — release identity (APP_NAME, DISPLAY_NAME "RepoPrompt CE", MARKETING_VERSION, BUILD_NUMBER, BUNDLE_ID, SIGNING_TEAM_ID); sourced by packaging/release scripts.
- `.swiftlint.yml` — SwiftLint policy mirroring the canonical first-party Swift path scope owned by `swift_style.sh`.
- `.swiftformat` — SwiftFormat policy (Swift 6.0, 4-space indent) with exclusions for vendored/C/generated and large workflow-prompt body files.
- `.gitignore` — ignores `.build`/`.swiftpm`/DerivedData, packaged `.app`/archives/symbols, and similar build artifacts.
- `.gitattributes` — disables whitespace handling for `Vendor/Sparkle/**` to preserve upstream distribution bytes.
- `.ignore` — ripgrep/search ignore list (build/derived/app/log artifacts).
- `.repo_ignore` — RepoPrompt-tooling ignore: keeps `Vendor/Sparkle/` out of agent context, re-includes `docs/`.
- `Launch RepoPrompt CE.command` — Finder double-click launcher; prefers coordinated `./conductor app relaunch`, falls back to `Scripts/run.sh` when `python3` is absent.
- `Install RepoPrompt CE Local Production.command` — Finder launcher to install a local self-signed production build to `/Applications` (requires python3).
- `AGENTS.md` — agent operating notes: prefer the coordinated daemon, run the contribution-check preflight before commit/push, signing/secure-storage rules.
- `CONTRIBUTING.md` — community contribution gate explainer (APPROVED_CONTRIBUTORS file, `issue`/`pr` capabilities, `lgtm`/`lgtmi` approval).
- `README.md` — product overview + CI/license badges and setup paths.
- `SECURITY.md` — private vulnerability reporting policy (GitHub Security tab flow).
- `LICENSE` — Apache License 2.0.
- `THIRD_PARTY_NOTICES.md` — attribution for bundled source and the resolved SwiftPM dependency graph (Sparkle 2.9.2 vendored, etc.).
- `minimalXcodeFreeSetup.md` — design note on the SwiftPM-app + shell-bundling approach and the Apple SDK/toolchain requirement (macOS 26 SDK for Liquid Glass).

### Scripts/ — build, run, doctor
- `Scripts/conductor.py` — the developer daemon: lane-serialized job queue with tickets/summaries, fake-sleep validation, and delegated `doctor/guardrails/format/lint/swift-build/build/test/run/app/smoke/diagnostics/release` operation families (protocol v3).
- `Scripts/package_app.sh` — packages the SwiftPM build into a `RepoPrompt.app` bundle (debug/release configs); applies Info.plist/entitlements templates and embeds resources.
- `Scripts/run.sh` — uncoordinated build+package+launch of the debug app with phase timing.
- `Scripts/run_without_github_tokens.sh` — exec wrapper that strips GH/GITHUB/SOURCE_GH tokens from the environment for hermetic SwiftPM resolves.
- `Scripts/doctor.sh` — environment diagnostics: Swift/Xcode CLT, SDK, signing, SwiftUI probe, debug-CLI and format-tools status.
- `Scripts/swift_style.sh` — canonical first-party Swift `format`/`format-check`/`lint` driver (SwiftFormat + strict SwiftLint).
- `Scripts/install_format_tools.sh` — install/status of pinned SwiftFormat/SwiftLint style tools.
- `Scripts/install_debug_cli.sh` — install/uninstall/status of the `rpce-cli-debug` symlink to the debug app's bundled `repoprompt-mcp`.

### Scripts/ — guardrails
- `Scripts/source_layout_guardrails.sh` — enforces the source-layout ownership map (forbidden cross-boundary references, banned patterns).
- `Scripts/contributor_allowlist_guardrails.sh` — validates `.github/APPROVED_CONTRIBUTORS` format/capabilities and sorting.
- `Scripts/swiftpm_notice_guardrails.sh` — verifies third-party SwiftPM/tree-sitter notices and SHA256SUMS inventories stay in sync.

### Scripts/ — release & signing
- `Scripts/release.sh` — release lane driver: `preflight`, ad-hoc `artifact`, `sync-cli-version`, packaging using `version.env` metadata.
- `Scripts/promote_release.sh` — maintainer promotion of a reviewed source draft to the stable public update channel (`repoprompt-ce-updates`).
- `Scripts/build_swiftpm_release_products.sh` — builds universal (arm64+x86_64) release products via per-arch SwiftPM scratch dirs, compares resources, and lipo-merges.
- `Scripts/load_release_metadata.sh` — sourced helper that parses `version.env` into shell vars via python3.
- `Scripts/sign_staged_release.sh` — codesigns a staged release app bundle with the Developer ID identity and entitlements template.
- `Scripts/validate_staged_release.sh` — validates a staged release bundle against the approved source root before signing.
- `Scripts/extract_staged_release.py` — safely extracts the untrusted release-stage ZIP without path traversal.
- `Scripts/sync_mcp_cli_version.sh` — syncs/checks the `repoprompt-mcp` CLI version string in `main.swift` against `MARKETING_VERSION`.
- `Scripts/install_local_production.sh` — builds and installs a local self-signed production app to `/Applications` (not notarized; not for distribution).
- `Scripts/publish_public_update_test.sh` — publishes a private smoke-test update to the public updates repo for appcast verification.
- `Scripts/derive_sparkle_public_key.swift` — derives the Sparkle Ed25519 public key from a 32-byte private seed (CryptoKit).
- `Scripts/verify_sparkle_signature.swift` — verifies a Sparkle EdDSA signature over a release archive.
- `Scripts/verify_sparkle_vendor.sh` — verifies the vendored Sparkle xcframework matches the trusted manifest/checksums.
- `Scripts/patch_keyboard_shortcuts_resource_lookup.sh` — applies the temporary KeyboardShortcuts 2.3.0 resource-lookup patch to the SwiftPM checkout for packaged layouts.
- `Scripts/verify_release_ref.sh` — asserts the release tag is canonical `vX.Y.Z` and reachable from protected `main`; emits the commit.
- `Scripts/verify_remote_release_commit.sh` — verifies a remote release tag resolves to the expected commit (HEAD-match enforcement).

### Scripts/ — packaging validation & smoke
- `Scripts/validate_app_architectures.sh` — asserts packaged Mach-O binaries contain the expected universal arch set (arm64,x86_64).
- `Scripts/validate_embedded_mcp_helper_layout.sh` — validates the embedded `repoprompt-mcp` helper layout inside the app bundle.
- `Scripts/validate_required_swiftpm_resource_bundles.sh` — asserts required SwiftPM resource bundles (e.g. KeyboardShortcuts) are embedded.
- `Scripts/validate_packaged_legal.sh` — checks the packaged app carries required legal/license notices.
- `Scripts/smoke_embedded_mcp_helper.sh` — smoke-tests the embedded MCP helper executable inside an app bundle.
- `Scripts/smoke_packaged_mcp_roundtrip.sh` — end-to-end packaged MCP socket roundtrip smoke with arch + socket-owner checks.
- `Scripts/compare_swiftpm_release_resources.py` — deterministically compares architecture-independent SwiftPM release resources across per-arch builds.
- `Scripts/write_app_artifact_manifest.py` — writes/verifies the deterministic external app artifact manifest (hashes, plist).
- `Scripts/verify_packaged_mcp_socket_owner.py` — fail-closed macOS ownership checks for packaged MCP release sockets.

### Scripts/ — signing identity inventory
- `Scripts/local_signing_identity.py` — inventories/resolves the local code-signing identity (reads `security`/`openssl`, or an offline JSON fixture in tests).
- `Scripts/inventory_local_signing_identities.py` — offline classifier for captured signing-identity JSON (cannot touch the Keychain).

### Scripts/ — self-tests (Python)
- `Scripts/test_conductor_lifecycle.py` — tests for conductor interactive app-lifecycle intent (app status/stop/relaunch).
- `Scripts/test_conductor_output.py` — tests for conductor concise output summaries.
- `Scripts/test_local_production_installer.py` — offline regression tests for the local production installer.
- `Scripts/test_release_promotion.py` — regression tests for reviewed release promotion.
- `Scripts/test_release_tooling.py` — regression tests for trusted release-control helpers (largest harness).
- `Scripts/test_repoprompt_mcp_parse_harness.py` — deterministic process harness for the DEBUG `repoprompt-mcp` parser hook.
- `Scripts/test_security_inventory.py` — offline tests for the Item 0 security inventory tooling.

### Scripts/ — fixtures & patches
- `Scripts/Fixtures/item0_identity_inventory_input.json` — fixture input for signing-identity inventory tests.
- `Scripts/Fixtures/item0_measurement_record.json` — fixture measurement record for security inventory tests.
- `Scripts/Fixtures/local_signing_identity_inventory.json` — fixture local signing-identity inventory (synthetic certs).
- `Scripts/patches/keyboardshortcuts-2.3.0-resource-lookup.patch` — unified diff patching KeyboardShortcuts `Utilities.swift` resource-bundle lookup for the packaged layout.

### .github/workflows/
- `.github/workflows/ci.yml` — main CI on PR/push to main (macos-26): guardrails, style lint, conductor-selftest, release-selftest, and build+test of both products.
- `.github/workflows/pr-gate.yml` — contribution gate on opened/reopened PRs (`pull_request_target`); auto-closes PRs from non-approved authors.
- `.github/workflows/issue-gate.yml` — contribution gate on opened/reopened issues; auto-closes issues from non-approved authors.
- `.github/workflows/approve-contributor.yml` — handles `lgtm`/`lgtmi` issue comments to update `.github/APPROVED_CONTRIBUTORS`.
- `.github/workflows/release-candidate.yml` — packages a secret-free ad-hoc release candidate and uploads it as a build artifact.
- `.github/workflows/release.yml` — maintainer publish workflow (workflow_dispatch with tag): validates the approved ref, builds, signs, notarizes, and drafts a GitHub Release.
- `.github/workflows/release-promote.yml` — promotes a reviewed source-draft tag (with reviewed SHA256SUMS digest) to the stable public update channel.
- `.github/APPROVED_CONTRIBUTORS` — tracked allowlist of GitHub handles with `issue`/`pr` capability read by the gate workflows.

### .agents/skills/
- `.agents/skills/rpce-contribution-check/SKILL.md` — contributor preflight skill: secret scanning + guardrails + validation lanes before commit/push.
- `.agents/skills/rpce-contribution-check/scripts/preflight.sh` — the `commit`/`push` preflight implementation (staged-index/outgoing-range secret scan, guardrails, push-boundary checks).
- `.agents/skills/rpce-contribution-check/references/validation-matrix.md` — changed-boundary → minimum focused evidence matrix (which `make` targets to run per touched area).
- `.agents/skills/rpce-contribution-check/agents/openai.yaml` — agent interface metadata (display name, default prompt) for the contribution-check skill.
- `.agents/skills/rpce-merge-pr-batch/SKILL.md` — maintainer skill to safely process an ordered PR batch end to end (disposable worktrees, scoped rp-cli Agent Mode review, exact-head checks, merge-commit merges).
- `.agents/skills/rpce-merge-pr-batch/agents/openai.yaml` — agent interface metadata for the PR-batch merge skill.
- `.agents/skills/rpce-release/SKILL.md` — release skill: secret-free contributor artifact lane plus local-production install and maintainer release orientation.

### docs/
- `docs/architecture/source-layout.md` — contributor-facing source ownership map (where new source/tests/fixtures/guardrails belong).
- `docs/architecture/provider-plugins.md` — the agent-provider plugin seam (provider-neutral core contract vs `RepoPromptClaudeCompatibleProvider` package).
- `docs/open-source-readiness.md` — maintainer inventory of public-readiness work (release metadata, signing, legal).
- `docs/releasing.md` — full release process: the two lanes, universal-arch requirements, signing/notarization/Sparkle.
- `docs/worktrees.md` — app-managed Git worktrees and `.worktreeinclude` for copying local ignored files.
- `docs/investigations/test-coverage-value-audit-ledger-2026-05-29.md` — authoritative audit ledger for the test-coverage value audit (census, retention/consolidation decisions, landed deltas).
- `docs/plans/test-coverage-value-audit-2026-05-29.md` — plan for rebalancing first-party Swift tests around contract value (500-method provisional ceiling).

### App bundle resources
- `AppBundle/Info.plist.template` — Info.plist template with `__PLACEHOLDER__` tokens (bundle id/name/version, signing mode, secure-storage backend) filled by packaging.
- `AppBundle/RepoPrompt.entitlements.template` — Developer ID entitlements template (app-identifier, team-identifier, allow-jit, app-scope bookmarks, MCP mach-lookup names).
- `AppBundle/RepoPrompt.local-self-signed.entitlements.template` — local self-signed variant entitlements (adds disable-library-validation; no team identifier).
- `AppResources/AppIcon.icns` — application icon resource.
- `AppResources/Audio/notificationDing.mp3` — bundled notification sound.

_Indexed 96 entries._
