# Chat · Context Builder · Prompt · Search — File Index
> Scope: `Features/{Chat,ContextBuilder,Prompt,Search}`. 53 files.

## Subsystem role
These four features form the user-facing context-and-conversation core of RepoPrompt CE. **Chat** owns the Oracle conversation surface: chat/preset models, on-disk session history, the large `OracleViewModel` (message lifecycle, model selection, MCP tool plumbing) and the SwiftUI bubble/transcript views. **Context Builder** drives the autonomous discovery agent that explores a workspace to enrich a prompt: token-budget resolution, run lifecycle/ownership, streaming output accumulation, follow-up plan/review/question generation, and its agent/settings/prompts UI. **Prompt** assembles the final copyable/sendable context — copy presets, prompt-section ordering, file-entry snapshots, pre-assembly + packaging services, token accounting, and the very large `PromptViewModel` that ties compose-tab state to assembly and headless plan/review generation. **Search** is the store-backed runtime search facade (PCRE2/wildmatch engine, path filtering, and concurrency admission coordinators) that backs MCP and other non-UI consumers via `WorkspaceFileContextStore` snapshots rather than the UI tree.

## Chat/

### Models/
- `Features/Chat/Models/AIChatMessage.swift` — In-memory chat message value type with content, reasoning, token/cost analytics, allowed file paths, and a `makeLightweight()` shrink path. Key: `AIChatMessage`.
- `Features/Chat/Models/ChatNameExtractor.swift` — Parses and strips a `<chatName=...>` marker from assistant output to auto-name a chat. Key: `ChatNameExtractor.extractAndRemove(from:)`.
- `Features/Chat/Models/ChatPreset.swift` — Chat preset config linking mode (chat/plan/review), model preset, and context strategy; with legacy-mode decoding and built-in defs. Key: `ChatPreset`, `ChatPresetMode`, `ChatPreset.BuiltIn`.

### Models/Presets/
- `Features/Chat/Models/Presets/ChatPresetOverrides.swift` — Sparse override record for built-in chat presets (only non-nil fields differ); supports `trimmed(against:)` and `isEmpty`. Key: `ChatPresetOverrides`.

### Services/
- `Features/Chat/Services/ChatHistoryManager.swift` — Loads/saves/lists chat sessions from disk with bounded-concurrency stub loading, history limits, and session metadata. Key: `ChatHistoryManager`, `ChatSessionMeta`, `ChatHistoryLimit`, `ChatDataError`.
- `Features/Chat/Services/ChatSession.swift` — Codable on-disk session model (messages, selection, workspace/tab/run IDs, preferred model/preset, slug shortID). Key: `ChatSession`, `ChatSessionError`.
- `Features/Chat/Services/StoredMessage.swift` — Minimal Codable per-message storage (raw text, tokens, cost, model name, allowed paths) with back-compat decoding. Key: `StoredMessage`.

### ViewModels/Oracle/
- `Features/Chat/ViewModels/Oracle/OracleViewModel.swift` — Central chat view model (~3.4k lines): message stream/finalize, `MessageReaper` drain, ephemeral state, chat modes/scopes, model selection, session persistence. Key: `OracleViewModel`, `MessageReaper`, `ChatMode`, `EphemeralMessageState`, `ChatSessionScope`.
- `Features/Chat/ViewModels/Oracle/OracleViewModel+MCP.swift` — MCP tool helpers extracted from the legacy MCP VM: model-selection resolution, send-tab context, availability guidance, ISO8601 formatting. Key: `OracleViewModel` ext, `OracleSendTabContext`, `ModelSelectionResult`.

### ViewModels/Presets/
- `Features/Chat/ViewModels/Presets/ChatPresetManager.swift` — Singleton store merging built-in + user chat presets with overrides applied, visibility, and lookup. Key: `ChatPresetManager.shared`.

### Views/
- `Features/Chat/Views/Bubbles.swift` — Markdown chat-bubble rendering with token-usage and file-selection indicators (~950 lines). Key: `TokenUsageIndicator`, `FileSelectionIndicator`.
- `Features/Chat/Views/Buttons.swift` — Reusable chat-control buttons (e.g. clear-chat trash with confirmation popover). Key: `TrashButton`.
- `Features/Chat/Views/ChatMessagesView.swift` — Scrollable transcript view with auto-scroll, near-bottom tracking, debounced scrolling, and session override. Key: `ChatMessagesView`.
- `Features/Chat/Views/ChatSharedComponents.swift` — Shared chat UI primitives incl. layer-backed cancel spinner / loading-with-stop indicator and perf diagnostics. Key: `LoadingIndicatorWithStop`, `LayerBackedCancelSpinner`.

## ContextBuilder/

### Services/
- `Features/ContextBuilder/Services/ContextBuilderBudgetResolver.swift` — Picks plan vs discovery token budget consistently across UI and MCP run paths. Key: `ContextBuilderBudgetResolver.resolveBudget(...)`.
- `Features/ContextBuilder/Services/ContextBuilderDefaults.swift` — Centralized CB defaults (token budgets, enhancement mode, clarifying-question and auto-plan toggles, timeout). Key: `ContextBuilderDefaults`, `PromptEnhancementMode`.

### ViewModels/
- `Features/ContextBuilder/ViewModels/ContextBuilderAgentViewModel.swift` — Main CB agent VM (~4.1k lines): run state, per-tab sessions, discovery/follow-up generation, ask-user interaction, plan status/errors. Key: `ContextBuilderAgentViewModel`, `AgentRunState`, `AgentRun`, `ContextBuilderFollowUpType`, `ContextBuilderPlanStatus`, `ContextBuilderGenerationError`.
- `Features/ContextBuilder/ViewModels/ContextBuilderAssistantOutputAccumulator.swift` — Incrementally accumulates streamed assistant output and maintains a capped preview without re-copying the full string. Key: `ContextBuilderAssistantOutputAccumulator`.
- `Features/ContextBuilder/ViewModels/ContextBuilderResponseType+Headless.swift` — Maps `ContextBuilderResponseType` to `HeadlessMode` (clarify → nil). Key: `ContextBuilderResponseType.headlessMode`.
- `Features/ContextBuilder/ViewModels/ContextBuilderRunLifecycle.swift` — Run origin (UI vs MCP), terminal outcome → `AgentRunState`, and the `ContextBuilderRunRecord` holding provider/task/output teardown state. Key: `ContextBuilderRunRecord`, `ContextBuilderRunOrigin`, `ContextBuilderRunTerminalOutcome`.

### Views/
- `Features/ContextBuilder/Views/ContextBuilderAgentView.swift` — Primary CB agent UI (~1.65k lines): header/model selection, ask-user wizard card, instructions input, run controls. Key: `ContextBuilderAgentView`.
- `Features/ContextBuilder/Views/ContextBuilderPromptsOverlay.swift` — Custom-instruction prompt model + JSON storage and the overlay editor for managing CB prompts. Key: `ContextBuilderPrompt`, `ContextBuilderPromptStorage.shared`.
- `Features/ContextBuilder/Views/ContextBuilderSettingsView.swift` — CB settings page (shared budgets, enhancement, timeout, UI-run vs MCP-run toggles); defers agent/model choice to Agent Models page. Key: `ContextBuilderSettingsView`.
- `Features/ContextBuilder/Views/ContextBuilderView.swift` — Thin container wiring `ContextBuilderAgentView` to `WindowState` and toggling the window kind. Key: `ContextBuilderView`.

## Prompt/

### Models/
- `Features/Prompt/Models/FilesTab.swift` — Persisted files-surface enum (Selected Files vs Context Builder) with legacy "Apply XML" decoding and CE default. Key: `FilesTab`.
- `Features/Prompt/Models/PromptAssemblyBuilder.swift` — Combines ordered prompt-section snippets into the final string honoring disabled sections and optional duplicated user instructions. Key: `PromptAssemblyBuilder`, `PromptSection`.
- `Features/Prompt/Models/PromptFileEntry.swift` — Lightweight tuple of a file plus codemap flag and optional line ranges for prompt assembly. Key: `PromptFileEntry`.
- `Features/Prompt/Models/PromptStorage.swift` — Reads/writes user-saved prompts as JSON in Application Support via a serial queue; import/export shape. Key: `PromptStorage.shared`, `PromptExport`.

### Models/Copy/
- `Features/Prompt/Models/Copy/BuiltInCopyPresets.swift` — Built-in copy preset definitions with stable UUIDs (Standard/Plan/Manual/DiffFollowUp/CodeReview). Key: `BuiltInCopyPresets`.
- `Features/Prompt/Models/Copy/CopyCustomizations.swift` — Per-workspace override deltas over a copy preset (include flags, tree/codemap/git modes, meta-prompt IDs). Key: `CopyCustomizations`.
- `Features/Prompt/Models/Copy/CopyOptionExtensions.swift` — Human-readable caption strings for `FileTreeOption` and `CodeMapUsage`. Key: `FileTreeOption.caption`, `CodeMapUsage.caption`.
- `Features/Prompt/Models/Copy/CopyPreset.swift` — Copy preset model + kinds and `GitInclusion`; nil behavior flags mean "use current UI state". Key: `CopyPreset`, `CopyPresetKind`, `GitInclusion`.
- `Features/Prompt/Models/Copy/GitInclusion+MCP.swift` — Maps an MCP git-scope string (none/selected/all) to `GitInclusion`. Key: `GitInclusion.fromMCPScope(_:)`.

### Models/Presets/
- `Features/Prompt/Models/Presets/CopyPresetOverrides.swift` — Sparse Codable override record for built-in copy presets (include flags, tree/codemap/git, prompt IDs). Key: `CopyPresetOverrides`.

### Services/
- `Features/Prompt/Services/PromptContextAccountingService.swift` — Dormant value-based orchestration resolving a stored selection into prompt-entry snapshots + token-accounting inputs, with no VM deps. Key: `PromptContextAccountingRequest`, `PromptContextAccountingResult`.
- `Features/Prompt/Services/PromptContextGitDiffPolicy.swift` — Holds the user-facing message for the deferred complete-worktree git-diff limitation. Key: `PromptContextGitDiffPolicy`.
- `Features/Prompt/Services/PromptContextPreAssemblyService.swift` — Pre-assembly request/runner producing resolved entries, file tree, and selected/complete git-diff artifacts before packaging. Key: `PromptContextPreAssemblyRequest`, `SelectedGitDiffArtifactPolicy`.
- `Features/Prompt/Services/PromptPackagingService.swift` — Builds the final packaged prompt text: code fences, XML-escaped title snippet, meta instructions, git-diff artifact handling (~840 lines). Key: `PromptPackagingService`, `MetaInstruction`.

### ViewModels/
- `Features/Prompt/ViewModels/PromptViewModel.swift` — Core prompt VM (~5.8k lines): compose tabs, files-tab selection, model/format selection, file-tree/codemap/git settings (copy + chat variants), git-artifact publishing. Key: `PromptViewModel`, `FileTreeOption`, `FilesTabSelection`, `GitArtifactPublishError`.
- `Features/Prompt/ViewModels/PromptViewModel+HeadlessPlan.swift` — Headless generation: `HeadlessMode`, frozen `HeadlessContextSnapshot`, and `buildHeadlessAIMessage(...)` for plan/chat/review without activating a tab. Key: `HeadlessMode`, `HeadlessContextSnapshot`.
- `Features/Prompt/ViewModels/PromptViewModel+PromptSnapshotEntries.swift` — Builds/caches `PromptFileEntry` snapshots for chat projection keyed on selection/slices/codemap versions. Key: `promptSnapshotEntriesForChatCached()`, `hasPromptSnapshotEntriesForChat()`.
- `Features/Prompt/ViewModels/TokenCountingViewModel.swift` — Published token/char/codemap/git-diff counts with dirty-flag-driven recomputation and per-file/folder token info (~1k lines). Key: `TokenCountingViewModel`, `DirtyKind`.

### ViewModels/Copy/
- `Features/Prompt/ViewModels/Copy/CopyPresetManager.swift` — Singleton store merging built-in + user copy presets with overrides applied, visibility, and lookup. Key: `CopyPresetManager.shared`.

### Views/Components/
- `Features/Prompt/Views/Components/FilePreviewPopover.swift` — Large file-preview popover with syntax highlighting (tree-sitter), slices-only toggle, codemap mode, and status banners. Key: `FilePreviewPopover`.
- `Features/Prompt/Views/Components/SelectedFilesGrid.swift` — Grid of selected prompt files showing codemap/slices/full display kind, badges, and remove action. Key: `SelectedFilesGrid`, `FileDisplayKind`.

## Search/
- `Features/Search/SearchMatch.swift` — Core search engine + result models (~3k lines): `FileSearchActor`, PCRE2/wildmatch engines with a compiled-pattern cache, and the `SearchResults`/`SearchMatch` data types. Key: `FileSearchActor`, `SearchMatch`, `SearchMode`, `SearchOptions`, `SearchResults`, `RegexEngine`, `RegexCache`.
- `Features/Search/SearchPathFiltering.swift` — Path-filter spec/clauses (exact file/folder, glob, legacy prefix) and snapshot-based path matching via wildmatch. Key: `SearchPathFilterSpec`, `SearchPathClause`, `FileSearchPathSnapshot`, `FileSearchPathFilterResult`.
- `Features/Search/StoreBackedWorkspaceSearch.swift` — Store-backed runtime search facade for MCP/non-UI consumers, operating on `WorkspaceFileContextStore` catalog snapshots and search-service readiness. Key: `StoreBackedWorkspaceSearch.search(...)`.
- `Features/Search/StoreBackedWorkspaceSearchAdmissionCoordinator.swift` — Bounded-concurrency admission for broad content searches (per-store/global queues, fair share, backpressure errors with retry-after) (~810 lines). Key: `StoreBackedWorkspaceSearchAdmissionCoordinator`, `StoreBackedWorkspaceSearchAdmissionError`, `BroadSearchAdmissionClass`.
- `Features/Search/StoreBackedWorkspaceSearchContentFetchAdmissionCoordinator.swift` — Bounds content-search descriptor/read work before store/filesystem actors; exact reads bypass this ingress (~750 lines). Key: `StoreBackedWorkspaceSearchContentFetchAdmissionCoordinator`.

_Indexed 53 files._
