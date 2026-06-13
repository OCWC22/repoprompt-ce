# Diagnostics — File Index

> Scope: `Features/Diagnostics`. 45 files.

## Subsystem role

This feature houses RepoPrompt CE's in-app instrumentation, perf probes, and the bundled benchmark suite — diagnostics intentionally kept inside the app target rather than split into a separate tool (per `docs/architecture/source-layout.md`). The vast majority are **DEBUG-only**: every file under `AgentMode/` (incl. `Stress/`), `App/`, `CodeMap/`, `Prompt/`, and all `MCP/` files except one are compiled behind `#if DEBUG` and gated by `defaults`/env-var/`__repoprompt_debug_diagnostics` MCP-op toggles, so release builds carry no footprint. The exceptions are the **Settings-visible Repo Bench suite** under `Benchmark/` (a real, ship-in-release edit-benchmark engine wired into the Settings UI, `BenchmarkSettingsView` "Repo Bench") and the always-compiled `MCP/MCPToolExecutionDiagnostics.swift` + `MCP/MCPToolDurationInventory.swift` execution-contract trace types. The hidden MCP surface (`__repoprompt_debug_diagnostics`, legacy `__repoprompt_debug_transport`) routes debug ops into the agent-perf, font-scale, memory/codemap, prompt-latency, read/search-latency, workspace, Sparkle, and transport/routing diagnostics extensions on `ServerNetworkManager`.

## AgentMode/

DEBUG-only Agent Mode performance and stress instrumentation.

- `Features/Diagnostics/AgentMode/AgentModePerfDiagnostics.swift` — DEBUG-only opt-in logger for steady-state Agent Mode hot paths (sidebar delete, session snapshots, structured event buffer); toggled via defaults/`RP_AGENT_MODE_PERF_DIAGNOSTICS`/MCP `agent_perf_metrics`. Key: `AgentModePerfDiagnostics`, `SidebarDeleteBeginContext`, `timestampMSIfEnabled`, `durationEvent`.
- `Features/Diagnostics/AgentMode/AgentTextDerivationPerfDiagnostics.swift` — DEBUG-only wrapper centralizing transcript/code text-derivation metrics (preview/collapse/JSON pretty-print) atop `AgentModePerfDiagnostics`. Key: `AgentTextDerivationPerfDiagnostics`, `Source`, `record(source:startMS:…)`.
- `Features/Diagnostics/AgentMode/MCPAgentLongThreadBaselineInventory.swift` — DEBUG-only code-owned, payload-free baseline/seam inventory for long-thread Agent work (test baselines, active-session-ID writer/construction inventories) surfaced via the diagnostics snapshot. Key: `MCPAgentLongThreadBaselineInventory`, `TestBaseline`, `baselines`.

### AgentMode/Stress/

- `Features/Diagnostics/AgentMode/Stress/AgentChatStressHarness.swift` — DEBUG-only stress harness that drives synthetic transcript churn and captures scroll/pin/streaming telemetry to flush out auto-follow and viewport-jump regressions. Key: `AgentChatStressHarness`, `AgentChatStressTelemetrySnapshot`, `start(currentTabID:)`.
- `Features/Diagnostics/AgentMode/Stress/AgentChatStressHarnessPanel.swift` — DEBUG-only SwiftUI overlay panel with Start/Pause/Resume/Reset/Detach controls and live telemetry/grouping/event JSON blocks for the harness. Key: `AgentChatStressHarnessPanel`.
- `Features/Diagnostics/AgentMode/Stress/AgentChatStressLaunchConfiguration.swift` — DEBUG-only `RP_AGENT_STRESS_*` env-var-parsed launch config (scenario, intervals, churn counts, persisted-session fixtures, thresholds) for the stress harness. Key: `AgentChatStressLaunchConfiguration`, `Scenario`, `MutationRefreshPolicy`.
- `Features/Diagnostics/AgentMode/Stress/AgentModeViewModel+StressHarnessSupport.swift` — DEBUG-only `AgentModeViewModel` extension supplying test/stress hooks: bind/prepare/reset stress sessions and inject mock transcript items. Key: `MockTranscriptRole`, `testPrepareStressSession(tabID:)`, `testResetStressTranscript(tabID:)`.

## App/

DEBUG-only app-wide restore and font-scale metrics.

- `Features/Diagnostics/App/FontScalePerfDiagnostics.swift` — DEBUG-only opt-in instrumentation for app-wide font-scaling hot paths; toggled via defaults/`RP_FONT_SCALE_PERF_DIAGNOSTICS`/MCP `font_scale_metrics`. Key: `FontScalePerfDiagnostics`, `isEnabled`, `setDebugProcessOverrideEnabled`, `debugStateSnapshot`.
- `Features/Diagnostics/App/WorkspaceRestorePerfLog.swift` — DEBUG-only opt-in logger for workspace/window restore and Agent-activation hot paths; the shared timestamp/event sink reused by CodeMap and Prompt diagnostics. Key: `WorkspaceRestorePerfLog`, `isEnabled`, `event`, `timestampMSIfEnabled`, `formatElapsedMS`.
- `Features/Diagnostics/App/WorkspaceRootLoadDiagnostics.swift` — DEBUG-only async-scoped trace-context bridge correlating workspace-switch metadata to per-root load events without polluting the core store API. Key: `WorkspaceRootLoadDiagnostics`, `Context`, `withContext(_:path:operation:)`, `rootRecordCreatedFields(forPath:)`.
- `Features/Diagnostics/App/WorkspaceSelectionDebugSignature.swift` — DEBUG-only helper computing low-cardinality count fields and a SHA256-derived signature for a `StoredSelection` to fingerprint selection state in diagnostics. Key: `WorkspaceSelectionDebugSignature`, `fields(for:prefix:)`, `signature(for:)`.

## Benchmark/

Settings-visible "Repo Bench" edit-benchmark suite (ships in release; not DEBUG-gated).

### Benchmark/Core/

- `Features/Diagnostics/Benchmark/Core/BelievableCodeFactory.swift` — Generates deterministic, believable TS/Go/Swift code via Mulberry32 to fill filler lines and craft decoys. Key: `BelievableCodeFactory`, `tsUtilityModule(rng:module:approxLines:)`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkAuditor.swift` — Audits generated seeds for solvability, emitting issues and search/replace hints per task. Key: `BenchmarkAuditor`, `BenchmarkAuditResult`, `BenchmarkAuditIssue`, `BenchmarkSearchHint`, `auditSeed(_:)`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkDecoyPlanner.swift` — Plans and produces decoy files (identical-core halos, patch-context mirrors) to tempt incorrect edit locations. Key: `BenchmarkDecoyPlanner`, `DecoyKind`, `DecoySpec`, `CoreRegion`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkDiffApplier.swift` — Applies a model's search/replace edits against the mock filesystem with edit-count and search-block-size guards. Key: `BenchmarkDiffApplier`, `BenchmarkDiffApplicationError`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkDiffParserBridge.swift` — `WorkspaceFilesViewModel` subclass backed by a benchmark mock-FS snapshot so the real diff/path-locate stack can resolve benchmark paths. Key: `BenchmarkWorkspaceFilesViewModel`, `pathLocation(_:exactMatchOnly:profile:rootScopeOverride:)`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkEngine.swift` — Orchestrates per-seed generation → execution across tasks with bounded concurrency. Key: `BenchmarkEngine`, `BenchmarkSeedExecution`, `BenchmarkTaskExecution`, `AsyncSemaphore`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkFileSystem.swift` — Lightweight in-memory mock filesystem (normalized POSIX-relative paths) plus its immutable snapshot type. Key: `BenchmarkMockFileSystem`, `normalize`, `content(for:)`, `setFile(_:content:)`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkModels.swift` — Core benchmark enums and value types (case types per language, difficulty, specs). Key: `BenchmarkCaseType`, `BenchmarkLanguage`, `BenchmarkTaskSpec`, `BenchConfig`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkProgressEvent.swift` — Coarse-grained progress event enum mirroring Context Builder's progress UI. Key: `BenchmarkProgressEvent`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkProjectScaffolder.swift` — Scaffolds a believable per-seed project layout (work files, exporters/importers, index/decoy pools) per language. Key: `BenchmarkProjectScaffolder`, `BenchmarkProjectLayout`, `scaffoldProject(language:rng:config:noise:into:)`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkRandom.swift` — Deterministic Mulberry32 PRNG plus seed utilities for reproducible benchmark generation. Key: `Mulberry32`, `nextUInt32`, `nextInt(upperBound:)`, `BenchmarkSeedUtilities`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkReporter.swift` — Verifies executions and assembles seed/task/final reports with points scaling. Key: `BenchmarkReporter`, `buildReport(coreSeed:executions:)`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkRunStore.swift` — Actor persisting benchmark run history to `UserDefaults` (`benchmarkRunHistory`), ISO8601 JSON. Key: `BenchmarkRunStore`, `shared`, `loadRuns`, `saveRun`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkRunSummary.swift` — Codable summary models for persisted runs (task/seed/type/run rollups with points and pass rates). Key: `BenchmarkRunSummary`, `BenchmarkSeedSummary`, `BenchmarkTaskSummary`, `BenchmarkTypeSummary`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkTaskExecutor.swift` — Builds the prompt, calls the AI model service, and applies output for a single benchmark task with output/context byte/line budgets. Key: `BenchmarkTaskExecutor`, `BenchmarkTaskExecutorError`, `ModelOutputProvider`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkTaskGenerator.swift` — Generates a full seed: scaffolds the project, layers noise, and emits cumulative per-language edit tasks against work files. Key: `BenchmarkTaskGenerator`, `BenchmarkGeneratedSeed`, `generateSeed(_:config:language:subseedIndex:)`.
- `Features/Diagnostics/Benchmark/Core/BenchmarkVerifier.swift` — Grades a task execution against expected output (lenient threshold, soft/hard error penalties, point scaling). Key: `BenchmarkVerifier`, `BenchmarkVerifying`, `GradingPolicy`, `BenchmarkVerifierReasons`, `verify(_:)`.
- `Features/Diagnostics/Benchmark/Core/UnifiedPatchApplier.swift` — Standalone multi-hunk unified-diff applier (`+`/`-`/context lines) used by the unified-patch benchmark cases. Key: `SimpleUnifiedPatchApplier`, `Op`, `Hunk`, `apply(patch:to:)`.

### Benchmark/ViewModels/

- `Features/Diagnostics/Benchmark/ViewModels/BenchmarkSettingsViewModel.swift` — `@MainActor ObservableObject` driving the Repo Bench Settings UI: run/history/leaderboard tabs, progress steps, run orchestration via the engine, and history/rankings. Key: `BenchmarkSettingsViewModel`, `Tab`, `ProgressStep`, `ProgressSummary`.

### Benchmark/Views/

- `Features/Diagnostics/Benchmark/Views/BenchmarkSettingsView.swift` — SwiftUI "Repo Bench" settings screen (header, tab selector, run controls, history/leaderboard) shown in app Settings. Key: `BenchmarkSettingsView`.

## CodeMap/

DEBUG-only CodeMap load-timing instrumentation.

- `Features/Diagnostics/CodeMap/CodeMapInitialRootLoadDiagnostics.swift` — DEBUG-only timing events for initial CodeMap root-load phases (cache rebuild/check, prune, enqueue) emitted through `WorkspaceRestorePerfLog`. Key: `CodeMapInitialRootLoadDiagnostics`, `start`, `cacheRebuild(rootCount:requestCount:startMS:)`.

## MCP/

The hidden `__repoprompt_debug_diagnostics` MCP surface and its per-domain op handlers (all DEBUG-only except the two execution-contract trace/inventory types).

- `Features/Diagnostics/MCP/DebugProcessMemorySampler.swift` — DEBUG-only Mach-based RSS/physical-footprint sampler with start/mark/snapshot/stop actions for large-workspace memory probes. Key: `DebugProcessMemorySampler`, `DebugProcessMemorySnapshot`, `DebugProcessMemoryMark`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugDiagnostics.swift` — DEBUG-only `ServerNetworkManager` extension defining the hidden `__repoprompt_debug_diagnostics` tool and dispatching every `op` (ping, connection/transport/routing snapshots, shutdown/restart, sleep/large-response probes, etc.). Key: `debugDiagnosticsToolName`, `legacyDebugTransportToolName`, `handleDebugDiagnosticsTool(connectionID:arguments:)`, `isDebugDiagnosticsToolName(_:)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugDiagnosticsAgent.swift` — DEBUG-only handler for the `agent_perf_metrics` op (enable/clear/probe/mark, summaries, diagnostic session snapshots) bridging to `AgentModePerfDiagnostics`. Key: `debugAgentPerfMetricsPayload(op:arguments:)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugDiagnosticsApp.swift` — DEBUG-only handler for the `font_scale_metrics` op (enable/clear/probe/mark + font-state snapshot) bridging to `FontScalePerfDiagnostics`. Key: `debugFontScaleMetricsPayload(op:arguments:)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugDiagnosticsMemoryCodemap.swift` — DEBUG-only handler for large-workspace memory/codemap ops (start/mark/snapshot/stop) driving `DebugProcessMemorySampler`. Key: `debugLargeWorkspaceMemoryPayload(op:arguments:)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugDiagnosticsPrompt.swift` — DEBUG-only handler measuring chat-preview context-build latency (warmups/iterations against a target window). Key: `debugChatPreviewContextLatencyPayload(op:arguments:)`, `debugMeasureChatPreviewContextLatency(op:windowID:warmups:iterations:)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugDiagnosticsReadSearchLatency.swift` — DEBUG-only handler for begin/snapshot/finish read-search latency captures backed by `EditFlowPerf` debug capture. Key: `debugMCPReadSearchCaptureBeginPayload(op:arguments:)`, `debugMCPReadSearchCaptureSnapshotPayload(op:arguments:)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugDiagnosticsWorkspace.swift` — DEBUG-only handler for workspace selection-fixture ops (snapshot/owners/apply) against a chosen window's selection state. Key: `debugWorkspaceSelectionFixturePayload(op:arguments:)`, `debugWorkspaceSelectionFixtureSnapshot(...)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugSparkleDiagnostics.swift` — DEBUG-only handler reporting Sparkle updater status (feed URL vs expected, bundle versions, last/passive appcast checks, manager snapshot). Key: `debugSparkleStatusPayload(op:)`.
- `Features/Diagnostics/MCP/MCPConnectionManager+DebugTransportDiagnostics.swift` — DEBUG-only transport/routing diagnostics (cancellation-probe store, transport ingress/routing snapshots, reconnect waits, routing-state clears). Key: `MCPDebugDiagnosticsProbeStore`, transport/routing payload handlers.
- `Features/Diagnostics/MCP/MCPToolDurationInventory.swift` — Payload-free inventory of the per-server Codex timeout vs dispatch-boundary execution contracts (deadlines, grace, long-synchronous exemptions); not DEBUG-gated. Key: `MCPToolDurationInventory`, `Entry`, `activeTimeoutSeconds`, `entries`.
- `Features/Diagnostics/MCP/MCPToolExecutionDiagnostics.swift` — Always-compiled MCP tool-execution trace event type (contract-selected/started/completed/deadline/cancellation/grace phases) used for execution-contract observability. Key: `MCPToolExecutionTraceEvent`, `Phase`, `isAlwaysEmitted`.

## Prompt/

DEBUG-only prompt/token recount instrumentation.

- `Features/Diagnostics/Prompt/PromptTokenRecountDiagnostics.swift` — DEBUG-only diagnostics for prompt/token recount and selected-path expansion (per-path/per-file phase, read-batch counters) emitted through `WorkspaceRestorePerfLog`. Key: `PromptTokenRecountDiagnostics`, `SelectedPathsState`, `event`, `selectionFields(_:)`.

_Indexed 45 files._
