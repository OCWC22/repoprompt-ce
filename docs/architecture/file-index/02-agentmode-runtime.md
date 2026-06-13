# Agent Mode — Runtime — File Index
> Scope: `Features/AgentMode/{Models,Providers,Recommendations,Services,Runtime}`. 75 files.

## Subsystem role
This is the non-UI engine behind Agent Mode: the data models for chat/transcript/sessions, the provider seam that adapts heterogeneous coding agents (Claude-compatible CLIs, Codex, ACP agents like OpenCode/Cursor, and headless MCP-only agents) to a single internal runtime contract, the auto-recommendation engine for model/provider selection, context-export services that package the workspace selection for a run, and the run/session/transcript lifecycle machinery that drives turns, tracks tool calls, persists transcripts, estimates context usage, and manages worktree bindings. The provider plugin layout (`NativeAgentRuntimeControlling`, the Claude-compatible adapter trio, `ClaudeAgentModeCoordinator`, `AgentRuntimeProviderService`) is documented in `docs/architecture/provider-plugins.md`.

## Models/
- `Features/AgentMode/Models/AgentAttachments.swift` — DTOs for run attachments. Key: `AgentImageSource`, `AgentImageAttachment`, `AgentTaggedFileAttachment`.
- `Features/AgentMode/Models/AgentChatModels.swift` — Core live-transcript chat item model plus persisted variant and runtime footer/duration formatting. Key: `AgentChatItem`, `AgentChatItemPersist`, `AgentChatItemKind`, `AgentMessageRuntimeFooter`, `AgentRuntimeDurationFormatter`.
- `Features/AgentMode/Models/AgentLogModels.swift` — Lightweight agent-log entry type for tool/message/thinking events. Key: `AgentLogEntry`, `AgentLogEntryType`.
- `Features/AgentMode/Models/AgentTranscriptModels.swift` — The canonical persisted transcript schema: turns, activities, tool executions, retention tiers, lifecycle/status enums. Key: `AgentTranscript`, `AgentTranscriptToolExecution`, `AgentTranscriptRetentionTier`, `AgentTranscriptActivityRole`, `AgentTranscriptToolStatus`, `AgentTranscriptSpanLifecycle`.
- `Features/AgentMode/Models/AgentWorkflow.swift` — Built-in workflow templates (Plan&Build, Review, Refactor, etc.) that wrap user input with structured prompts. Key: `AgentWorkflow`, `AgentWorkflowDefinition`.
- `Features/AgentMode/Models/CompressedTranscriptItem.swift` — Collapses consecutive tool/thinking rows into groups for compact history rendering. Key: `CompressedTranscriptItem`, `ToolCallGroup`, `compress(_:)`.
- `Features/AgentMode/Models/UserInteractionModels.swift` — Models for agent-initiated user prompts (discovery questions, approval requests) with stable SHA256-seeded IDs. Key: `DiscoveryQuestion`, `StableUserInteractionIdentity`.

### Models/ModelSelection/
- `Features/AgentMode/Models/ModelSelection/AgentACPModelRegistry.swift` — Thread-safe (NSLock) registry of live + persisted ACP-discovered model snapshots per provider, with warm-up of the standard store. Key: `AgentACPModelRegistry`, `updateDiscoveredModels`, `resolvedSnapshot`, `warmStandardStoreIfNeeded`.
- `Features/AgentMode/Models/ModelSelection/AgentCodexModelRegistry.swift` — Thread-safe registry of live Codex remote models; merges dynamic discovery with static options and persists to `CodexDynamicModelStore`. Key: `AgentCodexModelRegistry`, `updateLiveModels`, `resolvedOptions`.
- `Features/AgentMode/Models/ModelSelection/AgentModel.swift` — Enum of all known agent models (GPT-5.x/Codex variants, Claude sonnet/opus/haiku) plus discovery tags and defaults. Key: `AgentModel`, `AgentModelDiscoveryTag`, `defaultModel`.
- `Features/AgentMode/Models/ModelSelection/AgentModelCatalog.swift` — Large central catalog mapping provider availability to selectable model options, defaults, validation, and recommendation targets. Key: `AgentModelCatalog`, `AvailabilityContext`, `TaskLabelKind`, `isAgentAvailable`.
- `Features/AgentMode/Models/ModelSelection/AgentModelOption.swift` — Value type for one selectable model entry (display name, default flags, reasoning efforts). Key: `AgentModelOption`.
- `Features/AgentMode/Models/ModelSelection/AgentModelSelectionID.swift` — Stable compound `agent:model` identifier used by `agent_manage`/`agent_run` MCP entry points. Key: `AgentModelSelectionID`, `parse(_:)`, `rawValue`.
- `Features/AgentMode/Models/ModelSelection/ClaudeModelSpecifier.swift` — Parses/encodes Claude model+effort strings (`base:effort`) into runtime params. Key: `ClaudeModelSpecifier`, `encodedRaw`, `runtimeModelParam`.

## Providers/
- `Features/AgentMode/Providers/HeadlessAgentProvider.swift` — Protocol for headless CLI agents that operate via MCP tools only, plus the normalized event/usage types and an unsupported stub. Key: `HeadlessAgentProvider`, `AgentStreamEvent`, `AgentLifecycleEvent`, `TokenUsage`, `UnsupportedHeadlessAgentProvider`.

### Providers/ACP/
- `Features/AgentMode/Providers/ACP/ACPAgentProvider.swift` — ACP (Agent Client Protocol) provider abstraction: provider IDs, support results, discovered-session-model snapshot matching, and the provider protocol. Key: `ACPAgentProvider`, `ACPProviderID`, `ACPSupportResult`, `ACPDiscoveredSessionModels`.

### Providers/ClaudeCompatible/
- `Features/AgentMode/Providers/ClaudeCompatible/ClaudeCompatibleHeadlessProviderAdapter.swift` — Headless seam wrapping the core headless provider while carrying the plugin runtime DTO. Key: `ClaudeCompatibleHeadlessProviderAdapter`.
- `Features/AgentMode/Providers/ClaudeCompatible/ClaudeCompatibleModelCatalogAdapter.swift` — Maps Claude-compatible plugin catalog DTOs back onto RepoPrompt's stable `AgentModel` raw values, options, and defaults. Key: `ClaudeCompatibleModelCatalogAdapter`, `catalogSnapshot`, `defaultModelRaw`, `options`.
- `Features/AgentMode/Providers/ClaudeCompatible/ClaudeCompatibleNativeSessionAdapter.swift` — Actor adapter conforming to `NativeAgentRuntimeControlling`; carries plugin runtime DTO and delegates to the core controller. Key: `ClaudeCompatibleNativeSessionAdapter`.
- `Features/AgentMode/Providers/ClaudeCompatible/ClaudeCompatiblePluginBridge.swift` — Agent Mode facade mapping provider kinds, plugin IDs, runtime variants, and backend IDs for the Claude-compatible plugin path. Key: `ClaudeCompatiblePluginBridge`, `pluginID`, `agentKind`, `runtimeVariant`.

## Recommendations/
- `Features/AgentMode/Recommendations/AutoRecommendationEngine.swift` — MainActor service that detects provider availability (no network) and computes model/agent/MCP recommendations for the wizard. Key: `AutoRecommendationEngine`, `ProviderFlags`, `computeProviderStatus`.
- `Features/AgentMode/Recommendations/RecommendationTypes.swift` — Types for the recommendation domain: kinds, providers, availability snapshots, and best-practice profiles. Key: `RecommendationKind`, `RecommendationProviderKind`, `ProviderStatusSnapshot`.

## Services/
- `Features/AgentMode/Services/AgentContextExportResolver.swift` — Resolves the export context source (tab, prompt, selection, worktree bindings) for an agent run and builds it from a request. Key: `AgentContextExportSource`, `AgentContextExportIdentity`, `AgentContextExportSourceBuilder`.
- `Features/AgentMode/Services/AgentProviderContextBuilder.swift` — Builds the initial file-tree snapshot and capped file-contents block handed to a provider at run start. Key: `AgentProviderContextBuilder`, `initialFileTree`, `forkFileContentsBlock`.
- `Features/AgentMode/Services/AgentWorkspaceLookupContextResolver.swift` — Resolves a `WorkspaceLookupContext` for a session, applying worktree binding projection (logical→physical roots). Key: `AgentWorkspaceLookupContextSource`, `AgentWorkspaceLookupContextResolver`, `worktreeBindingFingerprint`.

## Runtime/

### Runtime/ (top level)
- `Features/AgentMode/Runtime/AgentAttachmentStore.swift` — Imports image attachments into the per-workspace `agent_attachments` directory with image-type validation. Key: `AgentAttachmentStore`, `ImportResult`, `importImageFile`.
- `Features/AgentMode/Runtime/AgentModeMCPPolicyInstaller.swift` — Installs the one-shot, TTL-bound MCP connection policy (granted/restricted tools) for an agent run. Key: `AgentModeMCPPolicyInstaller`, `install`, `additionalTools`.
- `Features/AgentMode/Runtime/AgentModePermissionPreferences.swift` — Storage shim for the tri-state sub-agent permission policy and per-provider overrides, backed by `AgentPermissionSecureStore`. Key: `AgentModePermissionPreferences`, `subagentPermissionPolicy`, `setSubagentPermissionPolicy`.
- `Features/AgentMode/Runtime/AgentModeProcessRunIdentity.swift` — Minimal helper to start/clear/read the per-session process run ID. Key: `AgentModeProcessRunIdentity`, `startFreshProcessRun`, `clearProcessRunID`.
- `Features/AgentMode/Runtime/AgentModeRunLease.swift` — Builds `MCPBootstrapLeaseSpec`/policy installer closures for Agent Mode runs from the shared MCP lease infrastructure. Key: `MCPBootstrapLeaseSpec.agentMode`, `MCPBootstrapLease.agentModePolicyInstaller`.
- `Features/AgentMode/Runtime/AgentModeRunService.swift` — Central run orchestrator: dependency wiring, run start/cancel, terminal publication, and draft restoration across all provider runners. Key: `AgentModeRunService`, `Dependencies`, `Hooks`, `CancellationIntent`, `CancellationCompletion`.
- `Features/AgentMode/Runtime/AgentRunLifecycleContracts.swift` — Runtime binding/epoch identity types capturing tab↔session binding and turn-to-turn transition relationships. Key: `AgentRunBindingIdentity`, `AgentRunTurnEpoch`, `AgentRunEpochTransitionKind`.
- `Features/AgentMode/Runtime/AgentRunTerminalCommitBarrier.swift` — Serializes terminal commit of a run (drain, publish, teardown) exactly once with revision tracking. Key: `AgentRunTerminalCommitBarrier`, `Request`, `AgentRunTerminalCommitRevision`, `AgentSessionRunState.isTerminalForCommit`.
- `Features/AgentMode/Runtime/AgentSession.swift` — Persisted agent-session document model plus session error and token-usage persist types. Key: `AgentSessionError`, `AgentTokenUsagePersist`, `AgentSession`.
- `Features/AgentMode/Runtime/AgentSessionDataService.swift` — Disk-backed session persistence: atomic serialized writes, metadata listing, save/load. Key: `AgentSessionDataService`, `AgentSessionMeta`, `AgentSessionDataError`, `AgentSessionDiskWriter`.
- `Features/AgentMode/Runtime/AgentSessionDataService+Restore.swift` — Streaming sidebar index/restore extension: batched async metadata stream with restore perf logging. Key: `buildSidebarIndexStream`, `sidebarStreamMetadataRecords`.
- `Features/AgentMode/Runtime/AgentSessionMetadataIndex.swift` — Versioned on-disk metadata index of sessions with quarantine records and migration decoding. Key: `AgentSessionMetadataIndex`, `AgentSessionMetadataRecord`, `AgentSessionMetadataQuarantineRecord`.
- `Features/AgentMode/Runtime/AgentSessionRestoreModels.swift` — Value types for sidebar restore: per-session index entries and the sidebar build request/batch. Key: `AgentSessionIndexEntry`, `AgentSessionSidebarBuildRequest`, `AgentSessionSidebarBuildBatch`.
- `Features/AgentMode/Runtime/AgentSessionRestoreSupport.swift` — Pure helpers for restore: title normalization, sidebar ordering/activity dates, cold-restore run-state normalization. Key: `AgentSessionRestoreSupport`, `normalizeColdRestoredRunState`, `shouldPreferSidebarEntry`.
- `Features/AgentMode/Runtime/AgentSessionWorktreeBinding.swift` — Storage-only model binding a workspace logical root to a Git worktree for one session. Key: `AgentSessionWorktreeBinding`, `AgentSessionWorktreeBindingSummary`.
- `Features/AgentMode/Runtime/AgentSessionWorktreeMergeOperation.swift` — Persisted compact metadata for a worktree merge workflow (status state machine, endpoints, fingerprints, artifacts). Key: `AgentSessionWorktreeMergeOperation`, `Status`, `AgentSessionWorktreeMergeSummary`.
- `Features/AgentMode/Runtime/AgentSkillCatalog.swift` — Discovers and resolves slash-command/skill definitions across Claude and generic `.agents` namespace roots with precedence ranking. Key: `AgentSkillCatalog`, `AgentSkillDefinition`, `Source`.
- `Features/AgentMode/Runtime/AgentWorkflowStore.swift` — MainActor `ObservableObject` managing custom markdown workflows in Application Support; watches dir, tracks hidden/featured built-ins. Key: `AgentWorkflowStore`, `customWorkflows`, `hiddenBuiltInIDs`, `featuredWorkflowIDs`.
- `Features/AgentMode/Runtime/PerKeyTaskStore.swift` — Generic MainActor store of per-key `Task`s with auto-cancel-on-replace and bulk cancel. Key: `PerKeyTaskStore`.

### Runtime/Claude/
- `Features/AgentMode/Runtime/Claude/ClaudeAbortArtifactFilter.swift` — Suppresses non-actionable Claude CLI abort/parse/lock/diagnostic error noise from the user-facing transcript. Key: `ClaudeAbortArtifactFilter`, `shouldSuppressUserFacingError`.
- `Features/AgentMode/Runtime/Claude/ClaudeAgentModeCoordinator.swift` — MainActor coordinator owning Claude native-runtime sessions: controller factory, per-tab tool tracking handlers, steering/interrupt safe-point logic. Key: `ClaudeAgentModeCoordinator`, `ClaudeControllerFactory`, `toolTrackingHooks`.
- `Features/AgentMode/Runtime/Claude/ClaudeInvalidToolErrorFilter.swift` — Detects Claude `tool_use_error` "No such tool available" results so the placeholder call + error can be dropped. Key: `ClaudeInvalidToolErrorFilter`, `isNoSuchToolAvailableError`.

### Runtime/Codex/
- `Features/AgentMode/Runtime/Codex/CodexAgentModeCoordinator.swift` — Very large MainActor coordinator for Codex native sessions: controller/connection-policy factories, native slash commands (compact/goal/computer-use), send/steer/interrupt orchestration. Key: `CodexAgentModeCoordinator`, `CodexControllerFactory`, `NativeSlashCommand`, `NativeSendOutcome`.
- `Features/AgentMode/Runtime/Codex/CodexComputerUseWorkflow.swift` — Feature-gated (defaults+env) Codex goals/computer-use workflow support with a goal-support change notification. Key: `CodexComputerUseWorkflow`, `CodexNativeFeatureGate`, `codexGoalSupportDidChange`.
- `Features/AgentMode/Runtime/Codex/CodexExecDiagnosticNoiseFilter.swift` — Filters internal Codex CLI stderr (timestamps, Go source refs, JSON-RPC, heartbeat markers) out of the agent log. Key: `CodexExecDiagnosticNoiseFilter`, `shouldSuppress`.
- `Features/AgentMode/Runtime/Codex/CodexSteerAckTracker.swift` — MainActor tracker correlating MCP-initiated Codex steer dispatch attempts with send acknowledgements (buffer/park/timeout). Key: `CodexSteerAckTracker`, `Ack`, `beginAttempt`, `awaitAck`, `resolve`.

### Runtime/Native/
- `Features/AgentMode/Runtime/Native/NativeAgentRuntimeContracts.swift` — The internal `NativeAgentRuntimeControlling` actor contract for interactive native CLI runtimes, plus current Claude-runtime compatibility typealiases. Key: `NativeAgentRuntimeControlling`, `NativeAgentRuntimeEvent`, `NativeAgentRuntimeSessionRef`, `NativeAgentRuntimeEffortLevel`.

### Runtime/ProviderBindings/
- `Features/AgentMode/Runtime/ProviderBindings/AgentModeProviderBindingService.swift` — MainActor service resolving editable provider controls + runtime permission bindings, including MCP-activation permission-profile policy. Key: `AgentModeProviderBindingService`, `controlsBinding`, `runtimePermission`, `permissionProfileForMCPActivation`.
- `Features/AgentMode/Runtime/ProviderBindings/AgentPermissionSecureStore.swift` — Large Keychain-backed fail-closed permission store: canonical JSON secure documents per provider domain with diagnostics and reset. Key: `AgentPermissionSecureStore`, `AgentPermissionSecureDomain`, `AgentPermissionStorageDiagnostic`, `AgentPermissionStorageResetResult`.
- `Features/AgentMode/Runtime/ProviderBindings/AgentProviderBindingID.swift` — Enum of provider binding identities (codex/claude/openCode/cursor) plus `AgentProviderKind→bindingID` mapping. Key: `AgentProviderBindingID`, `AgentProviderKind.providerBindingID`.
- `Features/AgentMode/Runtime/ProviderBindings/AgentProviderBindingModels.swift` — Cross-provider permission-level identifier and Codex server-entry conformances bridging provider-specific preference enums. Key: `AgentProviderPermissionLevelID`, `CodexIntegrationConfiguration.ServerEntry` conformances.
- `Features/AgentMode/Runtime/ProviderBindings/AgentProviderPermissionProfile.swift` — The effective-permission source enum (userConfigured / mcpSafeDefaults / providerOverride) with per-provider compatibility accessors (sandbox/approval). Key: `AgentProviderPermissionProfile`, `codexSandboxMode`, `codexApprovalPolicy`.
- `Features/AgentMode/Runtime/ProviderBindings/AgentProviderPreferenceSnapshotStore.swift` — Builds Settings vs profile-aware runtime controls/permission snapshots from defaults + secure store, with per-provider revisions. Key: `AgentProviderPreferenceSnapshotStore`, `controlsBinding`, `runtimePermission`, `revision`.

### Runtime/Providers/
- `Features/AgentMode/Runtime/Providers/AgentRuntimeProviderService.swift` — Defines the shared provider taxonomy (`AgentProviderKind`, `ClaudeCodeRuntimeVariant`) including command names, MCP client IDs, and runtime/agent-kind mappings; debug-logging toggle. Key: `AgentRuntimeProviderService`, `AgentProviderKind`, `ClaudeCodeRuntimeVariant`, `enableDebugLogging`.

### Runtime/Runners/
- `Features/AgentMode/Runtime/Runners/ACPIntegratedAgentModeRunner.swift` — MainActor runner driving ACP-provider runs: event consumption, per-tab tool-tracking correlation, error display mapping. Key: `ACPIntegratedAgentModeRunner`, `startRun`, `ConsumeEventsOutcome`.
- `Features/AgentMode/Runtime/Runners/ClaudeIntegratedAgentModeRunner.swift` — MainActor runner for Claude native runs: starts runs via the coordinator, consumes events, optional reasoning-extraction debug. Key: `ClaudeIntegratedAgentModeRunner`, `startRun`, `ConsumeEventsOutcome`.
- `Features/AgentMode/Runtime/Runners/CodexIntegratedAgentModeRunner.swift` — Thin MainActor runner that enables MCP servers then sends a Codex native message via the coordinator. Key: `CodexIntegratedAgentModeRunner`, `startRun`.
- `Features/AgentMode/Runtime/Runners/HeadlessAgentModeRunner.swift` — MainActor runner for headless providers: process-run identity, MCP lease, token accounting, ownership lifecycle. Key: `HeadlessAgentModeRunner`, `startRun`.

### Runtime/ToolTracking/
- `Features/AgentMode/Runtime/ToolTracking/AgentToolTrackingContracts.swift` — Normalized bridge from `AIStreamResult` tool events to provider handlers, plus tracking hooks. Key: `AgentToolStreamEvent`, `AgentToolTrackingHooks`, `from(_:)`.
- `Features/AgentMode/Runtime/ToolTracking/ClaudeAgentToolTrackingHandler.swift` — MainActor handler doing the hard dual-source provider/tracker tool correlation, turn-scoped reset, and RepoPrompt-tool slot enrichment for Claude sessions. Key: `ClaudeAgentToolTrackingHandler`, `ExplicitProviderToolResultAckSnapshot`, `handleProviderToolResult`.

### Runtime/Transcript/
- `Features/AgentMode/Runtime/Transcript/AgentToolCardRenderSummary.swift` — Codable render-summary for tool cards (title/subtitle/detail/status/op) with small-summary truncation and dictionary projection. Key: `AgentToolCardRenderSummary`, `AgentToolCardRenderStatus`, `inlineSummaryText`.
- `Features/AgentMode/Runtime/Transcript/AgentToolResultPersistencePolicy.swift` — Large policy that sanitizes/sizes tool results for runtime presentation vs persistent storage (byte caps, summary-only, raw-payload retention). Key: `AgentToolResultPersistencePolicy`, `AgentSanitizedToolResult`, `AgentPersistedToolResultSummary`, `AgentToolResultSanitizationPurpose`.
- `Features/AgentMode/Runtime/Transcript/AgentTranscriptHistoricalTruncationPolicy.swift` — Middle-truncates large text fields in old transcript turns during persistence/handoff/compaction, exempting recent turns and structured JSON. Key: `AgentTranscriptHistoricalTruncationPolicy`, `truncatedTranscript`, `maxFieldTokens`.
- `Features/AgentMode/Runtime/Transcript/AgentTranscriptPolicyPipeline.swift` — Orchestrates the transcript materialization pipeline (runtime / persisted / handoff) combining compaction, truncation, and projection. Key: `AgentTranscriptPolicyPipeline`, `Result`, `runtimeTranscript`, `persistedTranscript`, `handoffTranscript`.
- `Features/AgentMode/Runtime/Transcript/AgentTranscriptQualityRepair.swift` — Repairs incomplete/terminal transcript state (finalize pending tool calls) on cold restore or live terminal, per provider runtime. Key: `AgentTranscriptQualityRepair`, `finalizePendingTerminalTools`, `terminalMetadataRepairNeeded`.
- `Features/AgentMode/Runtime/Transcript/AgentTranscriptServices.swift` — The large transcript services hub: conversation replay (equivalent/bounded budgets), replay metrics/categories, and supporting transcript operations. Key: `AgentConversationReplayPolicy`, `AgentConversationReplayBudget`, `AgentConversationReplayMetrics`, `AgentConversationReplayCategory`.
- `Features/AgentMode/Runtime/Transcript/AgentTranscriptToolVisibilityPolicy.swift` — Decides which tool rows to suppress (placeholder names, path-like names, etc.) from the visible transcript. Key: `AgentTranscriptToolVisibilityPolicy`, `shouldSuppressRow`, `isPlaceholderToolName`, `pathSignal`.
- `Features/AgentMode/Runtime/Transcript/AgentWebToolActionPresentation.swift` — Classifies web-tool calls (search / read page / find-in-page) into presentation cards from tool name + args/result JSON. Key: `AgentWebToolActionPresentation`, `Action`, `classify`, `AgentWebToolActionInput`.

### Runtime/Usage/
- `Features/AgentMode/Runtime/Usage/ContextUsageEstimators.swift` — Shared context-usage snapshot model and the `ContextUsageEstimating` protocol contract used by per-provider estimators. Key: `ContextUsageSnapshot`, `ContextUsageSnapshotSource`, `ContextUsageSnapshotConfidence`, `ContextUsageEstimating`.
- `Features/AgentMode/Runtime/Usage/Claude/ClaudeContextUsageEstimator.swift` — MainActor estimator for Claude: queues/dequeues per-turn user-input token estimates and builds turn context usage. Key: `ClaudeContextUsageEstimator`, `enqueueUserTurnEstimate`, `beginTurn`, `ingestUsageSignal`.
- `Features/AgentMode/Runtime/Usage/Codex/CodexContextUsageEstimator.swift` — No-op estimator for Codex (relies on native token-usage events instead of local estimation). Key: `CodexContextUsageEstimator`, `ingestUsageSignal`.

_Indexed 75 files._
