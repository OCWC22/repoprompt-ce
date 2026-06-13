# RepoPrompt CE — Architecture Overview & File Index

> A complete, file-by-file index of the RepoPrompt CE codebase, plus a synthesized
> architecture walkthrough. Generated 2026-06-08 by reading every first-party
> source, test, build, and doc file through the RepoPrompt MCP.
>
> Start here, then jump into the per-area index files via [`README.md`](README.md)
> (the master table of contents). This file explains *how the pieces fit*; the
> numbered files (`01`–`22`) explain *what each individual file does*.

Coverage: **~1,127 first-party Swift/C source & test files** indexed individually,
plus **96 build / CI / script / skill / doc entries**, plus the vendored
`CSwiftPCRE2` C library (summarized). The only things not enumerated leaf-by-leaf
are vendored third-party trees (`CSwiftPCRE2`, `Vendor/`, `ThirdPartyLicenses/`,
`.build/`) and pure data fixtures (CodeMap goldens, audio).

---

## 1. What RepoPrompt CE is

RepoPrompt CE is a free, open-source **native macOS app and agent orchestrator for
context engineering** (Apache-2.0, macOS 26+). It assembles focused, reviewable
context from files, CodeMaps, repository structure, and Git diffs, then hands that
context to AI tools and CLI coding agents. It also ships an **MCP server** so
external MCP clients and CLI agents can search repos, inspect files, curate
context, run agent sessions, and orchestrate work through one native UI.

See [`../../../README.md`](../../../README.md) for install/build, and the two
canonical architecture docs this index complements:
[`../source-layout.md`](../source-layout.md) (ownership/placement rules) and
[`../provider-plugins.md`](../provider-plugins.md) (the Agent Mode provider seam).

## 2. SwiftPM targets & build graph

The repo is a single Swift package (`Package.swift`) producing several targets:

| Target | Kind | Role |
| --- | --- | --- |
| `RepoPrompt` | executable (app) | the macOS app; everything under `Sources/RepoPrompt` |
| `repoprompt-mcp` | executable (CLI) | the MCP CLI / bridge; `Sources/RepoPromptMCP` |
| `RepoPromptShared` | library | app↔CLI shared MCP control protocol; `Sources/RepoPromptShared` |
| `RepoPromptC` | C target | first-party C: path search, scoring, wildmatch, string ext |
| `CSwiftPCRE2` | C target | vendored PCRE2 regex engine + sljit JIT (third-party) |
| `TreeSitterScannerSupport` | C target | byte-for-byte upstream JS/Python tree-sitter scanner snapshots (linker ABI fallback only) |
| `RepoPromptClaudeCompatibleProvider` | package product | extracted Claude-compatible provider DTOs/codec/runtime; `Packages/RepoPromptAgentProviders` (path dependency, staged for future external split) |

Builds, runs, and tests are normally driven through **`conductor`** (`Scripts/conductor.py`),
a lane-serialized developer daemon (lanes: `build`, `debugArtifact`, `liveApp`,
`release`, `style`) fronted by `make dev-*` aliases. Plain `make`/`swift`/`Scripts`
are the uncoordinated fallback. Release identity lives in `version.env`; signing,
notarization, Sparkle, and the `APPROVED_CONTRIBUTORS` gate live in `Scripts/`
and `.github/workflows/`. See [`22-docs-build-scripts.md`](22-docs-build-scripts.md).

## 3. Source layout philosophy

Per [`../source-layout.md`](../source-layout.md), `Sources/RepoPrompt` uses a
hybrid **feature / infrastructure** layout (the old `Models/Views/ViewModels/Services/Utils`
buckets were pruned and are guardrail-enforced absent):

- **`App/`** — lifecycle, launch/configuration, commands, composition-root wiring,
  app notifications, Sparkle, root views/view-models. `WindowState` is the
  per-window composition root. → [`01-app.md`](01-app.md)
- **`Features/<Feature>/`** — product flows (AgentMode, Chat, ContextBuilder,
  CodeMap, Prompt, Search, Settings, WorkspaceFiles, Workspaces, Diagnostics),
  each typically split into `Models/ ViewModels/ Views/ Services/`.
- **`Infrastructure/<Area>/`** — cross-cutting substrate (AI, MCP, WorkspaceContext,
  VCS, FileSystem, Diffing, Process, Security, SyntaxParsing, UI, Regex,
  Networking, Persistence, Concurrency, Utilities).
- **`Support/`** — Obj-C bridging header. **`ThirdParty/`** — vendored SwiftPCRE2 wrapper.

`make guardrails` (`Scripts/source_layout_guardrails.sh`) enforces these rules
plus the removal of the legacy native file-tree / search-view-model / eager-load
seams.

## 4. Runtime architecture walkthrough

How the major subsystems connect, end to end:

### App shell & composition root → [01](01-app.md)
`RepoPromptApp` (SwiftUI `App`) + `AppDelegate` own app-global services (Sparkle,
appearance, notifications, CLI symlink, dock menu, termination). Each window gets a
**`WindowState`** (built by `WindowStateCompositionFactory`) that instantiates and
wires every per-window view model and service: prompt, oracle/chat, MCP server,
workspace manager, agent mode. **`WindowStatesManager`** is the app-wide registry
that persists/restores window sessions and arbitrates MCP continuity, instance
numbering, and termination. Deep links and notifications route through
`AppDeepLinkRouter` to the right window.

### Workspace context engine (the heart of context engineering) → [14](14-infra-workspacecontext-vcs.md)
**`WorkspaceFileContextStore`** (a large actor catalog, ~4k lines) sits at the
center. An ordered FS-delta ingress pipeline (`WorkspaceFileSystemIngressCoordinator`
+ deferred/replay actors) feeds it from `FileSystemService` ([15](15-infra-filesystem-diffing-process.md)).
The store serves **path lookup** (`PathMatcher`), **search** (`WorkspaceSearchService`
/ `PathSearchIndex`, backed by the C scoring in `RepoPromptC`), **selection**
(`WorkspaceSelectionCoordinator`), **slices** (line ranges + rebase), and **token
accounting**. `Features/WorkspaceFiles` ([07](07-features-codemap-workspacefiles-workspaces.md))
and `Features/Search` ([06](06-features-chat-contextbuilder-prompt-search.md)) are
the view-model/product projections over this store; there is no native tree view.

### VCS & worktrees → [14](14-infra-workspacecontext-vcs.md)
`VCSService` dispatches to `GitBackend` / `JujutsuBackend` behind the `VCSBackend`
protocol. `GitDiffEngine` + snapshot store/publisher manage diff artifacts under
`_git_data`. Worktree creation/merge (planner, include-copier honoring
`.worktreeinclude` per [`../worktrees.md`](../worktrees.md), merge preview/models)
backs Agent Mode's app-managed worktrees.

### CodeMaps, Prompt assembly, Context Builder → [07](07-features-codemap-workspacefiles-workspaces.md), [06](06-features-chat-contextbuilder-prompt-search.md)
**CodeMap** (`CodeMapExtractor`/`CodeMapGenerator` + Swift/TypeScript strategies,
PCRE2- and tree-sitter-driven signature extraction, cached) produces API-signature
maps. **Prompt** assembles the final copyable context (file map, selected files,
slices, CodeMaps, git diff) via `PromptAssemblyBuilder` / `PromptPackagingService`,
with copy presets and token counting. **Context Builder** is the agent that
autonomously explores the repo and curates a selection within a token budget.

### MCP server (core runtime + window tools + CLI) → [13](13-infra-mcp-core.md), [12](12-infra-mcp-services.md), [19](19-cli-shared-packages-ctargets.md)
The bundled server is a `Service`/`Tool` protocol model driven by `ServerController`
+ `MCPConnectionManager` (~9k lines) over Unix-socket and bootstrap-socket
transports. Per-window tool families (`file`, `git`, `oracle`, `selection`,
`context_builder`, `apply_edits`, `worktree`, `ask_user`, agent control) are
assembled by `MCPWindowToolCatalogService` and routed tab-first via
`MCPServerViewModel` + `WindowRoutingService`/`MCPBindingResolver`. `apply_edits`
is a self-contained port-based edit engine with per-(window,tab) approval.
Tool advertisement/availability is centralized in `Policies/`. The `repoprompt-mcp`
CLI executable ([19](19-cli-shared-packages-ctargets.md)) shares the control
protocol and bootstrap handshake via `RepoPromptShared`.

### Agent Mode → [02](02-agentmode-runtime.md), [03](03-agentmode-viewmodels.md), [04](04-agentmode-views-toolcards.md), [05](05-agentmode-views-chrome.md)
The orchestration surface. **Four run paths** (ACP, Claude-native, Codex, Headless),
each with a dedicated runner under `Runtime/Runners/`, all coordinated by
`AgentModeRunService` with terminal commits serialized by
`AgentRunTerminalCommitBarrier`. The **provider seam** (see [provider-plugins](../provider-plugins.md))
keeps a provider-neutral `NativeAgentRuntimeControlling` contract in core, with a
Claude-compatible adapter trio (native-session / headless / model-catalog) bridged
to the `RepoPromptClaudeCompatibleProvider` package via
`ClaudeCompatibleProviderRuntimeBridge` (the lone package import) and
`ClaudeCompatiblePluginBridge`. The **transcript** is a layered policy pipeline
(sanitize → historical-truncate → visibility → compaction/projection) rendered by a
`ToolCardRouter` that dispatches ~30 per-tool result cards. Permissions are
**Keychain-backed and fail-closed** (`AgentPermissionSecureStore`). The giant
`AgentModeViewModel` ([03](03-agentmode-viewmodels.md)) is split into single-concern
`+Extension` files feeding revision-gated UI stores.

### AI providers → [10](10-infra-ai-providers.md), [11](11-infra-ai-prompts-catalog.md)
`AIProviderFactory` is the single dispatch point. **Cloud chat providers** share an
`OpenAIProvider` base (OpenAI/Azure/Gemini/Grok/Groq/DeepSeek/Fireworks/Ollama/Z.AI
subclass it; Anthropic is native; OpenRouter wraps; `CustomOpenAIProvider`
backstops). **CLI/agent providers**: Claude Code (native SDK NDJSON process control
+ headless), Codex (app-server JSON-RPC + `codex exec`), Cursor/OpenCode (ACP).
The **provider-neutral workflow prompt catalog** (`Infrastructure/AI/Prompts/Workflows/`:
build/plan/deep-plan/investigate/optimize/refactor/review/orchestrate/reminder/oracle-export)
is shared by installs and MCP prompt registration.

### UI substrate → [16](16-infra-ui-components.md), [17](17-infra-ui-text-markdown.md)
Reusable SwiftUI/AppKit components, plus the `@`-mention-aware TextKit text-input
stack and markdown rendering + syntax highlighting used across Chat and Agent Mode.

### Security & secure storage → [18](18-infra-syntax-security-utilities.md)
Signing-identity-aware secure storage: `RuntimeCodeSigningDetector` →
`RuntimeCodeSigningPolicy` → backend selection (Developer-ID Keychain,
local-self-signed Keychain, or in-memory `EphemeralSecureKeyValueStore`), so secrets
never leak across signing identities. `ApplicationSecurity` ([01](01-app.md)) adds
release-only anti-tamper.

### Diagnostics → [09](09-features-diagnostics.md)
Mostly `#if DEBUG` instrumentation (Agent Mode perf/stress, MCP debug diagnostics
behind `__repoprompt_debug_diagnostics`, prompt/token recount, restore/font metrics),
plus the **Settings-visible "Repo Bench"** benchmark suite which ships in release.

### Tests → [20](20-tests-part1.md), [21](21-tests-part2.md)
~172 XCTest files mirroring the source layout, with the heaviest coverage on MCP
control/routing, Agent Mode, WorkspaceContext search/slices, VCS worktrees, and
JSON-only persistence guards. Package-level tests live with the provider package
([19](19-cli-shared-packages-ctargets.md)).

## 5. Cross-cutting seams to know

- **Provider plugin seam** — two bridge facades isolate the Claude-compatible
  package from core; adding a provider follows the recipe in [provider-plugins](../provider-plugins.md).
- **Shared MCP protocol** — `MCPControlMessages`, `MCPFilesystemIdentity`, and the
  `MCPExternalClientEvent` wire DTO are single-sourced in `RepoPromptShared/MCP`
  (guardrail-enforced) and consumed by both the app and CLI.
- **C bridge** — `Support/RepoPrompt-Bridging-Header.h` forward-declares the
  `tree_sitter_*` constructors and includes the `RepoPromptC` + `CSwiftPCRE2` headers.
- **Source-layout guardrails** — `make guardrails` blocks reintroducing pruned
  buckets, retired native-tree/search symbols, and stray fixtures under source.

## 6. Navigation hotspots (largest / most central files)

Useful entry points and the files most worth understanding first:

- `App/WindowState.swift` (+`WindowStateComposition.swift`) — composition root.
- `Infrastructure/WorkspaceContext/WorkspaceFileContextStore.swift` (~4k lines) — context catalog.
- `Infrastructure/MCP/MCPConnectionManager.swift` (~9k lines) — MCP runtime hub.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel.swift` (~4.3k lines, +10 extensions) — per-window MCP routing/selection.
- `Features/AgentMode/ViewModels/AgentModeViewModel.swift` (~14.5k lines, +~18 extensions) — Agent Mode core.
- `Features/AgentMode/Runtime/Transcript/AgentTranscriptServices.swift` (~7.5k lines) — transcript policy.
- `Infrastructure/AI/Providers/Codex/.../CodexAgentModeCoordinator.swift` (~6.7k lines) — Codex coordination.
- `Features/Prompt/ViewModels/PromptViewModel.swift` (~5.8k lines) — prompt assembly.
- `Features/WorkspaceFiles/ViewModels/WorkspaceFilesViewModel.swift` (~11.7k lines) — file-list projection.
- `Infrastructure/VCS/GitService.swift` (~2.9k lines) and `Features/CodeMap/CodeMapGenerator.swift` (~2.8k lines).
- `Scripts/conductor.py` (~3k lines) — the developer daemon.

## 7. Observations surfaced during indexing (not changes — just notes)

- **Version-number drift across three sources:** `version.env` declares
  `MARKETING_VERSION=1.0.8 / BUILD_NUMBER=9`; `docs/open-source-readiness.md` and
  `docs/releasing.md` still describe the line starting at `1.0.0 (1)`; and
  `App/Changelog.swift` carries an inherited `Changelog.current` of `2.1.24`
  (build 326) from the closed app. Worth reconciling before the next release-doc pass.
- **Empty file:** `Infrastructure/UI/Components/EnhancedTextkitView.swift` is 0 bytes
  (tracked but contentless).
- **Filename typos (preserved):** `Infrastructure/SyntaxParsing/ComprehensiveHiglighter.swift`
  ("Higlighter") and `Infrastructure/UI/TextField/ConstrainedTextKitVIew.swift`
  ("VIew"); the types inside are spelled correctly.
- **A few test filenames don't match their class names** (e.g.
  `WorkspaceSwitchCleanupTests.swift` → `AgentModeWorkspaceSwitchCleanupTests`,
  `DebugSecureStorageRuntimePolicyTests.swift` → `RuntimeCodeSigningPolicyTests`,
  `GitWorktreeContextSummaryTests.swift` → `GitWorktreeContextResolverTests`).
- **CodeMaps were not pre-built** in the RepoPrompt workspace index during this run,
  so the per-file descriptions were read directly from source rather than from
  `get_code_structure` codemaps.

These are recorded for accuracy; nothing in the codebase was modified to produce
this index.
