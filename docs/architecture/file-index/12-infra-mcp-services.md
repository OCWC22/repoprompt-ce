# Infrastructure — MCP Subsystems (Tools, ApplyEdits, Window Tools) — File Index

> Scope: `Infrastructure/MCP/{Agent,ApplyEdits,AppShared,Policies,ViewModels,WindowTools,WorkspaceApproval}`. 65 files.

## Subsystem role

These folders implement the application side of the bundled RepoPrompt MCP server: the concrete tools the server advertises (file/git/oracle/selection/context-builder/worktree/apply-edits/ask-user/agent control) plus the window-scoped routing, selection, token accounting, and approval machinery that backs them. `WindowTools/` holds one provider per public tool family, wired through a small runtime/catalog and a narrow dependency bundle so providers never reach back into `MCPServerViewModel`. `Agent/` implements the `agent_run`/`agent_manage`/`agent_explore` delegation tools and the actor-based run/session store; `ApplyEdits/` is the standalone search/replace + rewrite edit engine with approval gating and escape-decode fallbacks. `Policies/` expresses tool advertisement/availability/execution-contract decisions in terms of reusable capabilities, while `ViewModels/` (the `MCPServerViewModel` core + extensions) is the tab-first runtime model for selection, token stats, and per-tab context. `AppShared/` carries types shared verbatim with the `repoprompt-mcp` CLI target (bootstrap handshake, constants, socket reader), and `WorkspaceApproval/` mediates user approval of MCP-driven workspace mutations.

## Agent/ , ApplyEdits/ , AppShared/ , Policies/ , ViewModels/ , WindowTools/ , WorkspaceApproval/

### Agent/

- `Infrastructure/MCP/Agent/AgentExploreMCPToolService.swift` — Executes the `agent_explore` tool (start/poll/wait/cancel) spawning short-lived read-only explore child agents; delegates control ops to the run-control service. Key: `AgentExploreMCPToolService`, `execute(args:)`, `executeStart(args:)`.
- `Infrastructure/MCP/Agent/AgentExternalMCPRunStarter.swift` — Starts an external (delegated) MCP agent run against a resolved session target, extracting reasoning-effort suffixes from model strings and binding the request to the tab. Key: `AgentExternalMCPRunStarter`, `start(...)`, `extractReasoningEffort(from:)`, `StartOutcome`.
- `Infrastructure/MCP/Agent/AgentManageMCPToolService.swift` — Executes the `agent_manage` tool (list_agents/list_sessions/handoff/cleanup/etc.) for inspecting and administering agent sessions. Key: `AgentManageMCPToolService`, `execute(args:)`, `HandoffSessionInfo`, `CleanupSessionCandidate`.
- `Infrastructure/MCP/Agent/AgentMCPSelectionResolver.swift` — Resolves a `model_id` (task-label like `explore`/`engineer` or compound `agent:model`) into agent+model components via the catalog and global role defaults. Key: `AgentMCPSelectionResolver`, `resolve(modelID:...)`, `ResolvedSelection`.
- `Infrastructure/MCP/Agent/AgentMCPToolHelpers.swift` — Shared value-parsing utilities (string normalization, bool/timeout coercion, timestamps) used across the agent tool services and snapshot model. Key: `AgentMCPToolHelpers`, `normalizedString(_:)`, `parseBool(_:)`, `parseTimeoutSeconds(_:)`.
- `Infrastructure/MCP/Agent/AgentRunMCPSnapshot.swift` — Value model of an agent run's current state (status, worktree binding, transcript metadata) returned by the agent tools. Key: `AgentRunMCPSnapshot`, `Status`, `WorktreeBinding`.
- `Infrastructure/MCP/Agent/AgentRunMCPToolService.swift` — Executes the `agent_run` tool (start/wait/poll/steer/respond/cancel), oracle export plumbing, and shared run-control logic reused by explore. Key: `AgentRunMCPToolService`, `OracleExportRequest`, `AgentOracleExport`, `StartRun`, `HeartbeatOperation`.
- `Infrastructure/MCP/Agent/AgentRunSessionStore.swift` — Process-wide actor tracking agent run registrations, turn epochs, and async waiters so MCP `wait` calls block until a run completes, needs input, or advances. Key: `AgentRunSessionStore`, `WaitDisposition`, `EpochBeginResult`, `WakeReason`.
- `Infrastructure/MCP/Agent/MCPAgentRoleDefaultsService.swift` — Centralized resolution and persistence of global MCP agent role defaults (recommended vs effective per task label) over `GlobalSettingsStore`. Key: `MCPAgentRoleDefaultsService`, `RoleDefaultResolution`, `MCPAgentRoleDefaultsStoring`.

### ApplyEdits/

- `Infrastructure/MCP/ApplyEdits/ApplyEditsApprovalStore.swift` — Per-(window,tab) store of pending edit reviews and auto-edit toggles, exposing review decisions for the approval flow. Key: `ApplyEditsApprovalStore`, `PendingApplyEditsReview`, `ApplyEditsReviewDecision`, `ApplyEditsApprovalScope`.
- `Infrastructure/MCP/ApplyEdits/ApplyEditsEngine.swift` — Core diff/apply engine that turns a request (rewrite/single/batch) into diff chunks, applies them, and builds the result with perf instrumentation. Key: `ApplyEditsEngine`, `apply(request:to:options:)`, `default`.
- `Infrastructure/MCP/ApplyEdits/ApplyEditsEnginePorts.swift` — Protocol ports decoupling the engine from concrete diff generation, chunk application, and unified-diff rendering. Key: `DiffChunkGenerator`, `DiffChunkApplier`, `UnifiedDiffRendering`.
- `Infrastructure/MCP/ApplyEdits/ApplyEditsEscapeFallback.swift` — Retry helper that re-decodes C-style escape sequences in search/replace text when a literal match fails against the original file. Key: `ApplyEditsEscapeFallback`, `resolveSingle(...)`, `resolveBatch(...)`.
- `Infrastructure/MCP/ApplyEdits/ApplyEditsModels.swift` — Request/operation value types and edit modes for the apply-edits domain. Key: `ApplyEditsRequest`, `ApplyEditsMode`, `ApplyEditsOperation`, `OnMissing`, `ApplyEditsError`.
- `Infrastructure/MCP/ApplyEdits/ApplyEditsRequestBuilder.swift` — Parses normalized MCP tool args into an `ApplyEditsRequest`, enforcing exactly-one-of rewrite/replace/edits and running the echo guard. Key: `ApplyEditsRequestBuilder`, `build(from:)`, `buildFromNormalizedPayload(_:)`.
- `Infrastructure/MCP/ApplyEdits/ApplyEditsResult.swift` — Result value type carrying updated text, diff chunks, unified diffs, stats, and per-edit outcomes plus line-stat derivation. Key: `ApplyEditsResult`, `ApplyEditsStatus`, `ApplyEditsStats`, `ApplyEditsLineStats`.
- `Infrastructure/MCP/ApplyEdits/ApplyEditsService.swift` — Orchestrates preview + host write: computes the result via the engine then persists through the `FileEditHost`, returning file-created/overwritten metadata. Key: `ApplyEditsService`, `run(_:options:)`, `preview(_:options:)`.
- `Infrastructure/MCP/ApplyEdits/EscapeDecoding.swift` — Escape-decoding primitive with none/cStyle/smartHeuristic modes used by the fallback resolver. Key: `EscapeDecoder`, `EscapeDecodingMode`, `decode(_:mode:)`.
- `Infrastructure/MCP/ApplyEdits/FileEditHost.swift` — Protocol abstracting file existence/read/write for the edit service. Key: `FileEditHost`, `fileExists(path:)`, `readText(path:)`, `writeText(path:content:overwrite:)`.
- `Infrastructure/MCP/ApplyEdits/WorkspaceFileEditHost.swift` — `FileEditHost` backed by the workspace file mutation service, resolving paths within the lookup root scope and auto-selecting created files. Key: `WorkspaceFileEditHost`, `writeText(...)`, `WorkspaceFileMutationService` usage.

### AppShared/

- `Infrastructure/MCP/AppShared/MCPBootstrapMessages.swift` — Bootstrap UNIX-socket handshake message types shared by the app's `BootstrapSocketServer` and the `repoprompt-mcp` CLI. Key: `MCPBootstrapRequest`, `MCPBootstrapProtocol`, `MCPBootstrapTiming`.
- `Infrastructure/MCP/AppShared/MCPConstants.swift` — Centralized MCP-layer constants: debug tag, service/protocol version tags, bootstrap protocol version, heartbeat context ID. Key: `MCPConstants`, `serviceVersionTag`, `bootstrapProtocolVersion`, `buildFlavor`.
- `Infrastructure/MCP/AppShared/MCPFilesystemConstants.swift` — Debug-logging switches and helpers for the MCP transport/filesystem layer, gated by `REPOPROMPT_MCP_DEBUG`. Key: `MCPDebugLogging`, `mcpTransportLog(_:)`.
- `Infrastructure/MCP/AppShared/MCPNetworkConfig.swift` — Codable transport-preference config shared by app and CLI with legacy `forceFilesystemTransport` compatibility decoding. Key: `MCPNetworkConfig`, `MCPTransportPreference`.
- `Infrastructure/MCP/AppShared/MCPRoutingState.swift` — Persisted per-client routing records (window/workspace/session affinity) so MCP routing survives restarts, plus its storage helper. Key: `MCPRoutingState`, `ClientRecord`, `MCPRoutingStateStore`.
- `Infrastructure/MCP/AppShared/MCPServiceName.swift` — Structured Bonjour MCP service-name model (device ID + build-flavor tag + protocol tag) with encode/parse. Key: `MCPServiceName`, `MCPBuildFlavor`, `encoded()`.
- `Infrastructure/MCP/AppShared/NewlineDelimitedSocketReader.swift` — Reads newline-delimited frames from a non-blocking socket via `DispatchSourceRead`, yielding frames via callbacks with FD preflight validation. Key: `NewlineDelimitedSocketReader`, `ReadSourceFDPreflight`, `ReadSourceFDPreflightError`.

### Policies/

- `Infrastructure/MCP/Policies/AgentModeMCPToolAdvertisementPolicy.swift` — Decides which tools are hidden from `ListTools` per agent role (minimal read-only set for `explore`, gated externals for others). Key: `AgentModeMCPToolAdvertisementPolicy`, `hiddenToolNames(for:)`.
- `Infrastructure/MCP/Policies/AgentModeMCPToolPolicy.swift` — Execution-time tool restrictions/grants for agent-mode runs, with per-driver granted sets (generic, Claude-native, Codex-native, OpenCode ACP). Key: `AgentModeMCPToolPolicy`, `restrictedTools`, `grantedTools`, `claudeNativeGrantedTools`.
- `Infrastructure/MCP/Policies/DiscoverMCPToolPolicy.swift` — Tool policy for Context Builder/discovery runs: restricts edits/oracle/state-mutation, conditionally grants user interaction. Key: `DiscoverMCPToolPolicy`, `restrictedCapabilities`, `grantedCapabilities`.
- `Infrastructure/MCP/Policies/MCPPolicyGatedTools.swift` — Defines the special/legacy tools hidden from normal connections unless explicitly granted via `additionalTools`. Key: `MCPPolicyGatedTools`, `gatedCapabilities`, `names`.
- `Infrastructure/MCP/Policies/MCPToolCapabilities.swift` — Capability taxonomy and the capability→tool-name map that all policies derive concrete tool sets from. Key: `MCPToolCapability`, `MCPToolCapabilities`, `toolNames(for:)`, `externalName`.
- `Infrastructure/MCP/Policies/MCPToolExecutionContract.swift` — Per-tool execution contracts (bounded deadline vs long/lifecycle/interactive cancellable) and the catalog mapping advertised tool names to contracts. Key: `MCPToolExecutionContract`, `MCPToolExecutionContractCatalog`, `MCPToolExecutionDispatchError`.

### ViewModels/

- `Infrastructure/MCP/ViewModels/HeadlessMode+MCP.swift` — Maps `HeadlessMode` cases (plan/review/chat) to their MCP mode-name strings. Key: `HeadlessMode.mcpModeName`.
- `Infrastructure/MCP/ViewModels/MCPReadFileAutoSelectionCoordinator.swift` — Window-scoped response-lane coordinator that defers `read_file`/eligible `file_search` auto-selection mutations, draining accepted intents only when stable selection state is required. Key: `MCPReadFileAutoSelectionCoordinator`, `Intent`, `Route`, `DrainRequirement`, `ContextKey`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+CopyPresets.swift` — Parses MCP `copy_preset` args (string shorthand or object) into a selector for preset resolution. Key: `MCPServerViewModel.CopyPresetSelector`, `parseCopyPresetSelector(from:)`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+SelectionCore.swift` — Core selection types and code-map-usage gating for MCP (codemap origin classification, selection-source abstraction, globally-disabled codemap handling). Key: `MCPServerViewModel.CodemapEntry`, `CodemapOrigin`, `SelectionSource`, `effectiveMCPCodeMapUsage(_:)`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+SelectionEngine.swift` — Builds `StoredSelection` from parsed manage_selection inputs via the workspace selection mutation service, reporting invalid/codemap-unavailable paths. Key: `MCPServerViewModel.buildStoredSelection(...)`, `BuildStoredSelectionResult`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+SelectionParsing.swift` — Parses manage_selection path/slice arguments, including `#L` line-range suffix detection and slice-error collection. Key: `MCPServerViewModel.ManageSelectionInputs`, `parseManageSelectionInputs(...)`, `isLineRangeSuffix(_:)`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+SelectionReply.swift` — Assembles selection-reply DTOs for tab-scoped/virtual contexts, stabilizing virtual selections against live tab state and capturing user preset state. Key: `MCPServerViewModel.stabilizedVirtualSelection(...)`, `TabSelectionData`, `buildUserPresetState()`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+TabContext.swift` — Tab-first runtime context model and routing/binding logic (snapshots of a compose tab plus MCP metadata, run leases, worktree bindings). Key: `MCPServerViewModel.TabContextSnapshot`, `TabScopedContext` plumbing.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+TokenStats.swift` — Computes token-stat DTOs (total/files/prompt/tree/meta/git breakdown) shared by `workspace_context` and `manage_selection`, including virtual-context overrides. Key: `MCPServerViewModel.makeTokenStats(...)`, `ToolResultDTOs.TokenStats`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel+WorkspaceContext.swift` — Builds the `workspace_context` snapshot DTO (selection/files/code/tree/tokens) for a tab-scoped context with preset resolution and worktree scoping. Key: `MCPServerViewModel.buildTabWorkspaceContext(...)`, `ToolResultDTOs.PromptContextDTO`.
- `Infrastructure/MCP/ViewModels/MCPServerViewModel.swift` — The central MCP server view model: registers/executes tools, manages connections/run leases/close-safety, and owns selection debug logging. Key: `MCPServerViewModel`, `RequestMetadata`, `WindowMCPCloseSafetyState`, `MCPRunToolCleanupClaim`.

### WindowTools/

- `Infrastructure/MCP/WindowTools/MCPAgentControlToolProvider.swift` — Builds the `agent_explore` / `agent_run` / `agent_manage` tool definitions for delegating to child agent sessions. Key: `MCPAgentControlToolProvider`, `buildTools()`, `agentExploreTool()`.
- `Infrastructure/MCP/WindowTools/MCPAgentSessionControlToolProvider.swift` — Builds the agent self-control tools `share_thoughts` / `set_status` / `wait_for_next_user_instruction`. Key: `MCPAgentSessionControlToolProvider`, `shareThoughtsTool()`, `setStatusTool()`, `waitForNextInstructionTool()`.
- `Infrastructure/MCP/WindowTools/MCPApplyEditsToolProvider.swift` — Builds the `apply_edits` tool (rewrite/single/batch modes) wiring the request builder, edit service, and approval flow. Key: `MCPApplyEditsToolProvider`, `applyEditsTool()`, `EditSummary`.
- `Infrastructure/MCP/WindowTools/MCPAskUserToolProvider.swift` — Builds the unified `ask_user` clarifying-question tool that blocks for a user response, routed by run purpose; only visible when granted. Key: `MCPAskUserToolProvider`, `askUserTool()`.
- `Infrastructure/MCP/WindowTools/MCPContextBuilderToolProvider.swift` — Builds the `context_builder` tool (autonomous context exploration → plan/review/question) and its Codable result mapping to MCP values. Key: `MCPContextBuilderToolProvider`, `ContextBuilderToolResult`.
- `Infrastructure/MCP/WindowTools/MCPFileToolProvider.swift` — Builds the file-family tools `file_actions` / `get_code_structure` / `get_file_tree` / `read_file` / `file_search`. Key: `MCPFileToolProvider`, `fileActionsTool()`, `readFileTool()`, `fileSearchTool()`.
- `Infrastructure/MCP/WindowTools/MCPGitToolProvider.swift` — Builds the `git` tool (status/diff/log/blame/etc.) with worktree-scope warnings and trunk-comparison helpers. Key: `MCPGitToolProvider`, `gitTool()`, `worktreeWarning(from:)`.
- `Infrastructure/MCP/WindowTools/MCPOracleToolProvider.swift` — Builds the oracle tool family `oracle_utils` / `ask_oracle` / `oracle_send` / `oracle_chat_log`. Key: `MCPOracleToolProvider`, `oracleUtilsTool()`, `askOracleTool()`, `oracleSendTool()`.
- `Infrastructure/MCP/WindowTools/MCPPromptContextToolProvider.swift` — Builds the `workspace_context` (snapshot/export/list_presets) and `prompt` tools. Key: `MCPPromptContextToolProvider`, `workspaceContextTool()`, `promptTool()`.
- `Infrastructure/MCP/WindowTools/MCPSelectionToolProvider.swift` — Builds the `manage_selection` tool (get/add/remove/set/clear/preview/promote/demote across full/slices/codemap modes). Key: `MCPSelectionToolProvider`, `manageSelectionTool()`.
- `Infrastructure/MCP/WindowTools/MCPWindowToolCatalogService.swift` — Window-scoped service that aggregates all providers, materializes and caches the ordered tool list grouped by family. Key: `MCPWindowToolCatalogService`, `MCPWindowToolProviding`, `tools`, `invalidateToolsCache()`.
- `Infrastructure/MCP/WindowTools/MCPWindowToolContext.swift` — Narrow per-call context (tool name + window ID) passed to provider implementations. Key: `MCPWindowToolContext`.
- `Infrastructure/MCP/WindowTools/MCPWindowToolDependencies.swift` — Constructor-time dependency bundle of narrow services/closures handed to providers instead of an `MCPServerViewModel` reference. Key: `MCPWindowToolDependencies`, `ExecuteTool`, `WorkspaceSearch`, `ResolveContextBuilderTab`.
- `Infrastructure/MCP/WindowTools/MCPWindowToolGroup.swift` — Ordered public tool-family enum mapping each group to its ordered tool names (drives catalog ordering). Key: `MCPWindowToolGroup`, `orderedToolNames`.
- `Infrastructure/MCP/WindowTools/MCPWindowToolNames.swift` — Stable string constants for every window-scoped MCP tool name. Key: `MCPWindowToolName`.
- `Infrastructure/MCP/WindowTools/MCPWindowToolRuntime.swift` — Factory that wraps each provider's implementation in a `Tool` with a freshness policy and shared `executeTool` dispatch. Key: `MCPWindowToolRuntime`, `tool(name:freshnessPolicy:...)`, `MCPToolFreshnessPolicy`.
- `Infrastructure/MCP/WindowTools/MCPWindowWorkspaceToolHelpers.swift` — Pure shared helpers for workspace-oriented providers (path display formatting, code-structure token budgeting, search-arg shaping). Key: `MCPWindowWorkspaceToolHelpers`, `CodeStructureBudgetSelection`, `mcpDisplayPath(...)`.
- `Infrastructure/MCP/WindowTools/MCPWorktreeToolProvider+Merge.swift` — Merge-op extension of the worktree provider (preview/apply/status/continue/abort) for session-bound worktree merges. Key: `MCPWorktreeToolProvider.executeMerge(op:args:)`, `SessionResolution`.
- `Infrastructure/MCP/WindowTools/MCPWorktreeToolProvider.swift` — Builds the `manage_worktree` tool (list/show/create/bind/select/unbind plus merge ops) over the VCS service and repo-target resolver. Key: `MCPWorktreeToolProvider`, `manageWorktreeTool()`, `Operation`.

### WorkspaceApproval/

- `Infrastructure/MCP/WorkspaceApproval/WorkspaceApprovalManager.swift` — Singleton `ObservableObject` that queues and presents user-approval requests for MCP-triggered workspace mutations, persisting settings to `UserDefaults`. Key: `WorkspaceApprovalManager`, `pendingRequest`, `isApprovalOverlayVisible`.
- `Infrastructure/MCP/WorkspaceApproval/WorkspaceApprovalTypes.swift` — Value types for the approval flow: operation kinds, risk levels, request/result/settings models with display metadata. Key: `WorkspaceApprovalOperation`, `WorkspaceApprovalRequest`, `WorkspaceApprovalResult`, `WorkspaceApprovalSettings`, `WorkspaceApprovalRiskLevel`.

_Indexed 65 files._
