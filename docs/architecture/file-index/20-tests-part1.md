# Tests (part 1) — File Index
> Scope: `Tests/RepoPromptTests/{AgentMode,AI,App,Chat,CodeMap,ContextBuilder,Diagnostics,Diffing,Helpers,Mentions,Prompt,Security}`. 85 files.

## Subsystem role
The XCTest suite under `Tests/RepoPromptTests/` is the unit/integration safety net for the RepoPrompt CE macOS Swift app, imported with `@testable import RepoPrompt` (some with `@_spi(TestSupport)`). Part 1 covers the agent-runtime surfaces (Agent Mode run lifecycle, Codex/Claude-native session controllers, transcript serialization, worktree binding/merge), AI model selection, app-window/dock plumbing, code-map golden parity, context-builder runs, diagnostics inventories, diff/indentation appliers, mention/file-tag pickers, prompt assembly accounting, and secure-storage/keychain/code-signing policy. Most files are deterministic table/matrix tests over pure policy helpers; the heavier `@MainActor async` files exercise concurrency, lifecycle epochs, and persistence round-trips.

## AgentMode/

### (root)
- `Tests/RepoPromptTests/AgentMode/ACPProviderSessionIdentityTests.swift` — verifies ACP (Cursor/agent) provider session identity: runtime session ID published as verified load ID through bootstrap/prompt/shutdown. Key: `ACPProviderSessionIdentityTests`.
- `Tests/RepoPromptTests/AgentMode/AgentAskUserModelsTests.swift` — validates `AgentAskUserInteraction` rejection rules (empty/blank/duplicate question IDs, blank text, duplicate options). Key: `AgentAskUserModelsTests`.
- `Tests/RepoPromptTests/AgentMode/AgentContextExportResolverTests.swift` — checks context-export resolver display file counts and worktree export using physical content while showing logical paths. Key: `AgentContextExportResolverTests`.
- `Tests/RepoPromptTests/AgentMode/AgentExecutionWorktreeSelectionTests.swift` — verifies dedupe of execution worktree selections (prefers non-prunable, normalized-path guard, keeps distinct worktrees). Key: `AgentExecutionWorktreeSelectionTests`.
- `Tests/RepoPromptTests/AgentMode/AgentFileTagWorktreeResolutionTests.swift` — checks `AgentFileTagSuggestionService` resolves @-mentions against bound worktrees using logical paths without leaking hidden session worktree roots. Key: `AgentFileTagWorktreeResolutionTests`.
- `Tests/RepoPromptTests/AgentMode/AgentGitBranchSwitchRelevanceTests.swift` — verifies branch-switch provider-context relevance logic for bound/unbound sessions across logical base vs execution worktree. Key: `AgentGitBranchSwitchRelevanceTests`.
- `Tests/RepoPromptTests/AgentMode/AgentModeChatSwitchActivationTests.swift` — verifies warm compose-tab switches publish the destination transcript synchronously without lingering in-progress load state. Key: `AgentModeChatSwitchActivationTests`.
- `Tests/RepoPromptTests/AgentMode/AgentModeMCPWaitEpochTests.swift` — verifies MCP wait-tracking turn-epoch handling so terminal publication during an epoch gap does not lose a session wait. Key: `AgentModeMCPWaitEpochTests`.
- `Tests/RepoPromptTests/AgentMode/AgentModeRunServiceLifecycleTests.swift` — large suite over `AgentModeRunService` run lifecycle: startup-failure transitions before provider dispatch across codex/claude/openCode agents. Key: `AgentModeRunServiceLifecycleTests`.
- `Tests/RepoPromptTests/AgentMode/AgentModeStopSubmitTargetTests.swift` — verifies composer stop/cancel/submit targeting uses explicit tab-session identity and expected run IDs. Key: `AgentModeStopSubmitTargetTests`.
- `Tests/RepoPromptTests/AgentMode/AgentModeViewModelInactiveRefreshTests.swift` — verifies inactive/active transcript refresh compaction preserves raw render payloads for summary-only tool results. Key: `AgentModeViewModelInactiveRefreshTests`.
- `Tests/RepoPromptTests/AgentMode/AgentOnboardingWizardExitActionTests.swift` — verifies onboarding wizard exit policy marks onboarding seen before continuing to main or falling back to dismiss. Key: `AgentOnboardingWizardExitActionTests`.
- `Tests/RepoPromptTests/AgentMode/AgentProviderContextBuilderTests.swift` — verifies `AgentProviderContextBuilder` initial file tree and fork file-contents block read worktree content while displaying logical paths. Key: `AgentProviderContextBuilderTests`.
- `Tests/RepoPromptTests/AgentMode/AgentRunLifecycleContractsTests.swift` — verifies run-lifecycle contract types are `Sendable` and ownership captures an immutable turn epoch. Key: `AgentRunLifecycleContractsTests`.
- `Tests/RepoPromptTests/AgentMode/AgentRuntimeSidebarViewModelTests.swift` — verifies runtime sidebar metrics store: stale live-zero counts don't mask newer manage-selection file counts. Key: `AgentRuntimeSidebarViewModelTests`.
- `Tests/RepoPromptTests/AgentMode/AgentSessionWorktreeBindingPersistenceTests.swift` — verifies `AgentSession` JSON decoding back-compat and round-trips worktree bindings at serialization version 6. Key: `AgentSessionWorktreeBindingPersistenceTests`.
- `Tests/RepoPromptTests/AgentMode/AgentSessionWorktreeMergePersistenceTests.swift` — verifies `AgentSession` decodes legacy merge-less JSON and round-trips active worktree merge operations at version 6. Key: `AgentSessionWorktreeMergePersistenceTests`.
- `Tests/RepoPromptTests/AgentMode/AgentWorkspaceRootsSidebarStoreTests.swift` — verifies workspace-roots sidebar row projection (primary marking, up/down movability, single-root handling). Key: `AgentWorkspaceRootsSidebarStoreTests`.
- `Tests/RepoPromptTests/AgentMode/AgentWorktreeMergeAttentionTests.swift` — verifies worktree-merge blocker selection picks the most-recent non-terminal conflict/awaiting-commit operation. Key: `AgentWorktreeMergeAttentionTests`.
- `Tests/RepoPromptTests/AgentMode/BranchSwitchBranchListPresentationTests.swift` — verifies branch-list presentation grouping/sorting (current, checked-out-elsewhere, available) by name. Key: `BranchSwitchBranchListPresentationTests`.
- `Tests/RepoPromptTests/AgentMode/ContextBuilderAskUserStateTests.swift` — verifies `ContextBuilderAgentViewModel.TabSession` ask-user state initializes empty and stores structured pending interactions. Key: `ContextBuilderAskUserStateTests`.
- `Tests/RepoPromptTests/AgentMode/RecommendationProviderFilterNormalizationTests.swift` — verifies `GlobalSettingsStore` normalization of recommendation provider filters across removed/empty/legacy raw inputs. Key: `RecommendationProviderFilterNormalizationTests`.
- `Tests/RepoPromptTests/AgentMode/WorkspaceSwitchCleanupTests.swift` — verifies workspace switch clears foreground sessions before slow provider dispose completes, using captured run IDs for background cleanup. Key: `AgentModeWorkspaceSwitchCleanupTests`.
- `Tests/RepoPromptTests/AgentMode/WorktreeMergeReviewStateTests.swift` — verifies `WorktreeMergeSourceBindingResolver` resolution by repo root/name/path and merge-review endpoint construction. Key: `WorktreeMergeReviewStateTests`.

### AgentMode/ClaudeCompatible/
- `Tests/RepoPromptTests/AgentMode/ClaudeCompatible/ClaudeCompatiblePluginBridgeTests.swift` — verifies only the bridge file imports the provider package and plugin-ID mapping (claude-code/GLM/kimi/custom) round-trips to agent kinds. Key: `ClaudeCompatiblePluginBridgeTests`.
- `Tests/RepoPromptTests/AgentMode/ClaudeCompatible/ClaudeNativeApprovalAndResumeTests.swift` — verifies Claude-native RepoPrompt permission auto-approval matching and allow-payload preserves tool-use ID/updated input. Key: `ClaudeNativeApprovalAndResumeTests`.

### AgentMode/Codex/
- `Tests/RepoPromptTests/AgentMode/Codex/BashToolResultParserRecoveryTests.swift` — verifies `BashToolResultParser` keeps command liveness/metadata contracts in sync for running and terminal command-execution payloads. Key: `BashToolResultParserRecoveryTests`.
- `Tests/RepoPromptTests/AgentMode/Codex/CodexAgentModeCoordinatorLivenessTests.swift` — verifies Codex coordinator watchdog liveness: active thread snapshots reconcile waiting flags and advance lifecycle without spurious stall errors/transcript rows. Key: `CodexAgentModeCoordinatorLivenessTests`.
- `Tests/RepoPromptTests/AgentMode/Codex/CodexGoalSupportDefaultTests.swift` — verifies `CodexGoalSupport.isEnabled` defaults on for a missing UserDefaults key and respects explicit false. Key: `CodexGoalSupportDefaultTests`.
- `Tests/RepoPromptTests/AgentMode/Codex/CodexNativeSessionControllerEventRecoveryTests.swift` — verifies `CodexNativeSessionController` error-notification parsing retains retry metadata/scope across flat and nested payload shapes. Key: `CodexNativeSessionControllerEventRecoveryTests`.
- `Tests/RepoPromptTests/AgentMode/Codex/CodexNativeSessionControllerGoalConfigTests.swift` — verifies agent-mode default options carry goal feature config to both start and resume requests. Key: `CodexNativeSessionControllerGoalConfigTests`.

### AgentMode/ToolCards/
- `Tests/RepoPromptTests/AgentMode/ToolCards/WebSearchToolCardTests.swift` — verifies web-search/web-read/file-search tool-name normalization stays distinct and the web action classifier distinguishes search/read/find/code-search. Key: `WebSearchToolCardTests`.

### AgentMode/Transcript/
- `Tests/RepoPromptTests/AgentMode/Transcript/AgentConversationReplaySerializationTests.swift` — verifies conversation replay equivalent-mode output matches legacy bytes and category metrics across mixed chat item kinds. Key: `AgentConversationReplaySerializationTests`.
- `Tests/RepoPromptTests/AgentMode/Transcript/AgentToolResultPersistencePolicyTests.swift` — verifies persisted tool-result summaries are bounded/structured (oracle metadata for confirmed oracle tools, truncated raw payloads). Key: `AgentToolResultPersistencePolicyTests`.
- `Tests/RepoPromptTests/AgentMode/Transcript/AgentTranscriptActivationRepaintRemountPolicyTests.swift` — verifies transcript activation repaint/remount key computation (produces remount key on hydrated live-bottom activation, suppresses duplicate-revision over-limit). Key: `AgentTranscriptActivationRepaintRemountPolicyTests`.
- `Tests/RepoPromptTests/AgentMode/Transcript/AgentTranscriptCrawlRefreshBenchmarkTests.swift` — opt-in (DEBUG, env/flag-gated) benchmark instrumenting long active-crawl final-turn transcript refresh via `AgentTranscriptDebugInstrumentation`. Key: `AgentTranscriptCrawlRefreshBenchmarkTests`.

## AI/
- `Tests/RepoPromptTests/AI/AIModelPreferenceRegressionTests.swift` — verifies AI model dropdown display names for invalid raw values (planning dropdown shows explicit invalid states, non-planning keeps first-available fallback). Key: `AIModelPreferenceRegressionTests`.
- `Tests/RepoPromptTests/AI/CodexNativeSessionControllerInterruptTests.swift` — verifies Codex interrupt error parsing (active-turn-mismatch actual turn ID extraction, resolved interrupt turn-ID matrix). Key: `CodexNativeSessionControllerInterruptTests`.
- `Tests/RepoPromptTests/AI/ModelPickerStringOrderingTests.swift` — verifies `ModelPickerStringOrdering` ASCII-fold + raw-scalar tie-break ordering and semantic GPT version/tier/reasoning sort. Key: `ModelPickerStringOrderingTests`.

## App/
- `Tests/RepoPromptTests/App/AppPlatformUtilityRecoveryTests.swift` — verifies `AgentSessionDeepLinkRoute` URL round-trip / invalid-scoped-route rejection and appcast parser version selection. Key: `AppPlatformUtilityRecoveryTests`.
- `Tests/RepoPromptTests/App/DockMenuTests.swift` — verifies `DockMenuController` builds dock menu items with correct metadata and routes nil-sender actions (activate/new-window). Key: `DockMenuTests`.
- `Tests/RepoPromptTests/App/WindowCloseCoordinatorDecisionTests.swift` — verifies window-close coordinator allow/block decisions across termination, user confirmation, workspace deletion, and MCP/active-session impact. Key: `WindowCloseCoordinatorDecisionTests`.

## Chat/
- `Tests/RepoPromptTests/Chat/ChatNameExtractorTests.swift` — verifies `ChatNameExtractor.extractAndRemove` parses `<chatName=...>` markers (quoted/unquoted/self-closing/absent/empty) and strips them from content. Key: `ChatNameExtractorTests`.

## CodeMap/
Data dirs: `Fixtures/` holds per-language source fixtures (c, cpp, dart, go, java, js, php, py, rb, rs, swift, ts, tsx) and `Goldens/` holds the expected `.codemap.txt` snapshots plus `fixture-tree.txt` consumed by the golden tests below.
- `Tests/RepoPromptTests/CodeMap/Helpers/CodeMapFixtureRunner.swift` — test helper defining `CodeMapFixture`, fixture relative-path catalogs (core/expanded/edge), golden loading and file-tree rendering used by code-map tests. Key: `CodeMapFixtureRunner`.
- `Tests/RepoPromptTests/CodeMap/CodeMapGoldenTests.swift` — verifies per-language fixtures match golden code-map descriptions and snapshot file-tree marks code-map fixtures across modes. Key: `CodeMapGoldenTests`.
- `Tests/RepoPromptTests/CodeMap/FileViewModelAcceptedCodeMapTests.swift` — verifies `File` code-map lifecycle: rejects path mismatches, toggles the accepted-codemap flag, clears API/line count on nil. Key: `FileViewModelAcceptedCodeMapTests`.
- `Tests/RepoPromptTests/CodeMap/Fixtures/swift/smoke.swift` — Swift code-map fixture input (protocol `Greeter`, `FriendlyGreeter`, `makeGreeter`) parsed against its golden. Key: `Greeter`/`FriendlyGreeter` fixture.

## ContextBuilder/
- `Tests/RepoPromptTests/ContextBuilder/ContextBuilderAssistantOutputAccumulatorTests.swift` — verifies `ContextBuilderAssistantOutputAccumulator` message-boundary accumulation and incremental preview matches legacy whole-output compaction. Key: `ContextBuilderAssistantOutputAccumulatorTests`.
- `Tests/RepoPromptTests/ContextBuilder/ContextBuilderNestedMCPFailureTests.swift` — DEBUG `@MainActor` test that a nested-read handler completion plus an exact MCP response-delivery failure settles the outer context builder run. Key: `ContextBuilderNestedMCPFailureTests`.
- `Tests/RepoPromptTests/ContextBuilder/ContextBuilderRunLifecycleTests.swift` — verifies context-builder run lifecycle terminal claim + continuation fire exactly once via `ContextBuilderRunRecord`. Key: `ContextBuilderRunLifecycleTests`.

## Diagnostics/
- `Tests/RepoPromptTests/Diagnostics/BenchmarkDiffApplierTests.swift` — verifies the benchmark diff applier applies generated diff chunks for a modify change against a mock filesystem. Key: `BenchmarkDiffApplierTests`.
- `Tests/RepoPromptTests/Diagnostics/MCPAgentLongThreadBaselineInventoryTests.swift` — verifies the long-thread baseline inventory is payload-free and covers required Item-Zero seams (writer/construction/publisher/cleanup counts, revision). Key: `MCPAgentLongThreadBaselineInventoryTests`.
- `Tests/RepoPromptTests/Diagnostics/MCPToolDurationInventoryTests.swift` — DEBUG test that `MCPToolDurationInventory` covers the full advertised tool catalog (26 entries) and projects execution timeout contracts. Key: `MCPToolDurationInventoryTests`.

## Diffing/
- `Tests/RepoPromptTests/Diffing/DiffChunkTextApplierTests.swift` — verifies diff-chunk text applier adjusts later chunk start lines for cumulative insertion/deletion offsets. Key: `DiffChunkTextApplierTests`.
- `Tests/RepoPromptTests/Diffing/DiffGenerationUtilityRoutingTests.swift` — verifies `DiffGenerationUtility` replace-all bypasses duplicate-match ambiguity and applies cumulative offsets for positive/negative deltas. Key: `DiffGenerationUtilityRoutingTests`.
- `Tests/RepoPromptTests/Diffing/DiffParserRecoveryTests.swift` — verifies `DiffParserUtils.extractFileEntries` handles quoted/space paths, smart quotes, self-closing deletes, and truncated bodies. Key: `DiffParserRecoveryTests`.
- `Tests/RepoPromptTests/Diffing/IndentationCorrectionUtilityRecoveryTests.swift` — verifies `IndentCorrectionUtility.reIndentUsingSearchBlock` converts between space/tab styles and absorbs leaked leading tabs. Key: `IndentationCorrectionUtilityRecoveryTests`.
- `Tests/RepoPromptTests/Diffing/UnifiedDiffGeneratorRecoveryTests.swift` — verifies `UnifiedDiffGenerator` normalizes absolute/space paths in headers, summarizes creates/deletes, and carries cumulative delta across sorted hunks. Key: `UnifiedDiffGeneratorRecoveryTests`.

## Helpers/
- `Tests/RepoPromptTests/Helpers/GitWorktreeTestSupport.swift` — test support enum to drive `git` worktree fixtures and poll for a stable `GitWorktreeDescriptor` (branch/head). Key: `GitWorktreeTestSupport`.
- `Tests/RepoPromptTests/Helpers/RepoRoot.swift` — test helper that locates the repo root by walking up to `Package.swift` + `Sources/RepoPrompt`, with relative-path utilities. Key: `RepoRoot`.
- `Tests/RepoPromptTests/Helpers/StoreBackedWorkspaceSearchSharedAdmissionTestLease.swift` — DEBUG actor providing a serialized lease so store-backed workspace-search admission tests don't run concurrently. Key: `StoreBackedWorkspaceSearchSharedAdmissionTestLease`.
- `Tests/RepoPromptTests/Helpers/TestProcessRunner.swift` — test helper that runs a subprocess draining stdout/stderr into a `TestProcessResult`. Key: `TestProcessRunner`.
- `Tests/RepoPromptTests/Helpers/TestProcessRunnerTests.swift` — verifies `TestProcessRunner` drains large child output (64KB from `head /dev/zero`) without deadlock. Key: `TestProcessRunnerTests`.

## Mentions/
- `Tests/RepoPromptTests/Mentions/FileMentionPickerStyleTests.swift` — verifies `FileMentionPickerStyle` compact vs expanded configuration values (max results, rows, width, subtitles) and raw-value normalization. Key: `FileMentionPickerStyleTests`.
- `Tests/RepoPromptTests/Mentions/MentionCoordinatorTests.swift` — verifies `MentionCoordinator` workspace reuse keeps suggestions and commit/removal behavior when the file manager changes. Key: `MentionCoordinatorTests`.
- `Tests/RepoPromptTests/Mentions/MentionOverlayControllerTests.swift` — verifies `MentionOverlayController` visible-row-limit normalization and highlighted-row accessibility traits. Key: `MentionOverlayControllerTests`.
- `Tests/RepoPromptTests/Mentions/MentionSuggestionServiceTests.swift` — verifies `MentionSuggestionService` compact search caps/unsubtitled rows vs expanded larger cap with parent-path subtitles. Key: `MentionSuggestionServiceTests`.
- `Tests/RepoPromptTests/Mentions/TextFieldMentionHelpersTests.swift` — verifies `FileTagMentionHelper` text-field mention selection: click-then-accept commits the clicked suggestion. Key: `TextFieldMentionHelpersTests`.

## Prompt/
- `Tests/RepoPromptTests/Prompt/PromptContextAccountingServiceTests.swift` — verifies `PromptContextAccountingService` preserves stored selection order after batch lookup under concurrent reads. Key: `PromptContextAccountingServiceTests`.
- `Tests/RepoPromptTests/Prompt/PromptContextPreAssemblyServiceTests.swift` — verifies `PromptContextPreAssemblyService` resolves worktree content and logicalizes the rendered file tree. Key: `PromptContextPreAssemblyServiceTests`.
- `Tests/RepoPromptTests/Prompt/PromptContextResolvedFileTreeTests.swift` — verifies resolved-file-tree rendering truth table (include flag x mode -> rendersFileTree/effectiveMode). Key: `PromptContextResolvedFileTreeTests`.
- `Tests/RepoPromptTests/Prompt/PromptMigrationRemovalTests.swift` — verifies prompt section-order/copy-preset resolution falls back to current defaults without old migration paths (drops legacy diffFormatting section). Key: `PromptMigrationRemovalTests`.
- `Tests/RepoPromptTests/Prompt/PromptResourceMirrorRemovalTests.swift` — verifies bundled prompt mirror is removed, canonical legacy prompt sources preserved, and no source/test references the old mirror path. Key: `PromptResourceMirrorRemovalTests`.
- `Tests/RepoPromptTests/Prompt/WorkflowPromptCatalogTests.swift` — verifies `RepoPromptWorkflowID` MCP-prompt order and install order/command-name lists stay stable (rp-build, rp-investigate, …). Key: `WorkflowPromptCatalogTests`.

## Security/
- `Tests/RepoPromptTests/Security/AgentPermissionSecureStoreTests.swift` — verifies agent-permission secure store reads canonical plain documents (Codex approval/sandbox/reviewer/bash policy) from the secure-string backend. Key: `AgentPermissionSecureStoreTests`.
- `Tests/RepoPromptTests/Security/DebugSecureStorageRuntimePolicyTests.swift` — verifies `RuntimeCodeSigningPolicy` routes verified persistent domains (developerID/local-self-signed/apple-development-debug) to isolated services. Key: `RuntimeCodeSigningPolicyTests`.
- `Tests/RepoPromptTests/Security/KeychainServiceTests.swift` — verifies keychain service adds UI-skip for non-interactive reads and omits it for interactive reads, via a fake `SecItem` client. Key: `KeychainServiceTests`.
- `Tests/RepoPromptTests/Security/LocalSigningIdentityRegistryTests.swift` — verifies `LocalSigningIdentityRegistry` loads owner-only versioned records and rejects missing/wrong-owner/insecure/malformed registries. Key: `LocalSigningIdentityRegistryTests`.
- `Tests/RepoPromptTests/Security/SecureStorageAccountCatalogTests.swift` — verifies `SecureStorageAccountCatalog` freezes the exact ordered set of account identifiers (provider API keys, CLI keys, agent permission keys). Key: `SecureStorageAccountCatalogTests`.
- `Tests/RepoPromptTests/Security/SecureStorageRepairServiceTests.swift` — verifies `SecureStorageRepairService` scan classifies known accounts non-interactively and continues past per-account failures. Key: `SecureStorageRepairServiceTests`.
- `Tests/RepoPromptTests/Security/SecureStorageRepairViewModelTests.swift` — verifies `SecureStorageRepairViewModel` does not scan until invoked, then records account states (legacy read, target untouched). Key: `SecureStorageRepairViewModelTests`.

_Indexed 85 files._
