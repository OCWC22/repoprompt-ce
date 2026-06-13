# CodeMap · Workspace Files · Workspaces — File Index
> Scope: `Features/{CodeMap,WorkspaceFiles,Workspaces}`. 46 files.

## Subsystem role
CodeMap is the code-map extraction engine: given workspace files it builds a structured `FileAPI` "API surface" per file (imports, types, functions, signatures, referenced types) via Tree-sitter parsing plus per-language signature heuristics, caches the results on disk by content fingerprint, and renders both the definition blocks and the ASCII file tree that get injected into prompts. WorkspaceFiles owns the in-memory file/folder/git view-model graph for the currently loaded workspace roots — there is no native tree view here; instead it projects roots, drives selection and checkbox state, sorting, expansion, file preview, and uncommitted-git surfaces. Workspaces is the workspace-level management layer: the workspace data model (roots, presets, compose-tab state), the `WorkspaceManagerViewModel` that loads/saves/switches workspaces and applies presets, the switch-session registry that gates switching on active chat/agent sessions, the checkout-refresh seam, and all the SwiftUI views for picking, creating, and managing workspaces and presets.

## CodeMap/

### LanguageStrategies/
- `Features/CodeMap/LanguageStrategies/SwiftCodeMapStrategy.swift` — Swift-specific Tree-sitter codemap pass: discovers type containers (class/struct/enum/actor/extension/protocol) and maps functions/properties into them via range containment. Key: `SwiftCodeMapStrategy`, `TypeBoundary`, `Context`.
- `Features/CodeMap/LanguageStrategies/TypeScriptCodeMapStrategy.swift` — TS/TSX-only codemap pass building class/interface container boundaries and mapping members by smallest-containing declaration (JS uses the legacy path). Key: `TypeScriptCodeMapStrategy`, `ContainerBoundary`, `buildContext`.

### Models/
- `Features/CodeMap/Models/FileAPI.swift` — the structured per-file "API surface" model plus its component records, with computed description/token-count/defined-type-name fields used for prompt injection. Key: `FileAPI`, `FunctionInfo`, `ClassInfo`, `InterfaceInfo`, `ParameterInfo`, `EnumInfo`, `TypeAliasInfo`, `VariableInfo`.

### (root)
- `Features/CodeMap/CodeMapCacheManager.swift` — disk + in-memory cache of `FileAPI` results keyed per root folder, validated by content fingerprint/mod-date and a version number; tracks dirty roots and async load/commit. Key: `CodeMapCacheManager`, `CodeMapContentFingerprint`, `CodeMapCacheFileEntry`, `CodeMapCacheRootFolder`, `CodeMapCacheContainer`.
- `Features/CodeMap/CodeMapCaptureIndex.swift` — indexed/sorted view over Tree-sitter captures (grouped by name, binary-search containment lookups) to avoid O(n²) capture scans. Key: `CodeMapCaptureIndex`, `captures(named:)`, `firstCapture(named:containedIn:)`.
- `Features/CodeMap/CodeMapExtractionMemo.swift` — per-file memoization cache for deterministic extractor calls (JS/TS signatures, function/variable dicts, TS return/annotation/alias parses) keyed by language+line. Key: `CodeMapExtractionMemo`, `jstsSignature`, `matchFunctionLine`.
- `Features/CodeMap/CodeMapExtractor.swift` — standalone helpers that gather relevant `FileAPI` definitions into prompt-ready blocks and render the workspace file tree from explicit selection context. Key: `CodeMapExtractor`, `CodeMapUsage`, `DefinitionBlockResult`, `FileTreeResult`, `FileTreeSelectionContext`, `generateFileTree`.
- `Features/CodeMap/CodeMapExtractor+Snapshots.swift` — extension that captures a Sendable `FileTreeSelectionSnapshot` on the main actor and renders the file tree off it (off-main, cancellable, with depth/token-budget/sibling-cap trimming). Key: `makeFileTreeSnapshot`, `generateFileTree(using:snapshot)`.
- `Features/CodeMap/CodeMapGenerator.swift` — core codemap generator: parses file content with SwiftTreeSitter, runs the codemap query, dispatches to language strategies, and builds the `FileAPI`; carries perf signposts and line-boundary helpers. Key: `CodeMapGenerator`, `generateCodeMap`, `computeLineBoundaries`, `lineNumber(for:)`.
- `Features/CodeMap/CodeMapPCRE2Regex.swift` — thin PCRE2 wrappers for the developer-authored codemap regex constants (compile with UTF/JIT options, byte-range capture extraction). Key: `CodeMapPCRE2Pattern`, `CodeMapPCRE2Match`, `capture`, `trimmedCapture`.
- `Features/CodeMap/CodeMapPerfStats.swift` — lightweight single-thread perf counters/options for codemap generation (parse, capture-loop attribution, signature, type-cleaner timings) used by signposts/diagnostics. Key: `CodeMapPerfStats`, `CodeMapPerfOptions`, `CodeMapSyntaxPerfStats`, `CodeMapSyntaxStartupPerfStats`.
- `Features/CodeMap/CodeScanActor.swift` — concurrency-bounded scanner that drives codemap generation across many files with a permit-based async limiter, cancellation, and DEBUG logging. Key: `CodeScanActor`, `CodeScanAsyncLimiter` (private), `withPermit`.
- `Features/CodeMap/FileTreeSelectionSnapshot.swift` — immutable Sendable snapshot of the folder/file tree + selection used to render trees off the main actor. Key: `FileTreeSelectionSnapshot`, `FileTreeFolderSnapshot`, `FileTreeFileSnapshot`, `FileTreeNodeSnapshot`.
- `Features/CodeMap/JSTSSignatureExtractor.swift` — extracts clean JS/TS signatures from a single declaration line, handling type literals, generics, arrow functions, and function-like vs statement-like brace semantics. Key: `JSTSSignatureExtractor`, `JSTSSignatureContext`, `extract`.
- `Features/CodeMap/LanguageTypeExtractor.swift` — large bank of per-language regex patterns and `matchAny…` helpers extracting function/variable types across Swift, C/C++, Go, TS, Python, etc. Key: `LanguageTypeExtractor`, `swiftFunctionRegex`, `FunctionLineMatch`.
- `Features/CodeMap/ReferencedTypesAccumulator.swift` — accumulates and de-duplicates referenced type names from raw type strings, delegating to `TypeCleaner` with a per-instance cache and perf stats. Key: `ReferencedTypesAccumulator`, `insert(rawType:)`, `types`.
- `Features/CodeMap/SwiftSignatureParser.swift` — small Swift signature helpers, notably top-level `->` return-type extraction with `where`-clause trimming and word-boundary checks. Key: `SwiftSignatureParser`, `extractReturnType`.
- `Features/CodeMap/TopLevelScanner.swift` — shared string scanner that splits/searches at top-level delimiters while tracking nesting of `<>`, `()`, `{}`, `[]`. Key: `TopLevelScanner`, `TrackDelimiters`, `splitTopLevel`, `firstTopLevelRange`.
- `Features/CodeMap/TypeCleaner.swift` — normalizes raw type strings into base type names: strips container generics (Promise/Record/etc.), method-call suffixes, and language built-in primitives, with a cache. Key: `TypeCleaner`, `extractBaseTypes`, `TypeCleanerCacheKey`, primitive sets per language.

## WorkspaceFiles/

### Models/
- `Features/WorkspaceFiles/Models/FileSystemItems.swift` — value types for filesystem entries (path-equality `Folder`/`File`) plus a relative-path helper and a `FileTreeItem` enum for grouped tree rows. Key: `FileSystemItem`, `Folder`, `File`, `FileTreeItem`.

### Utilities/
- `Features/WorkspaceFiles/Utilities/SortingUtils.swift` — sort-method enumeration (name/extension/date/token, asc/desc), allowed-method subsets per surface, and icons; backs file-list and selection ordering. Key: `SortMethod`, `selectedFilesAllowed`, `fileTreeAllowed`.

### ViewModels/
- `Features/WorkspaceFiles/ViewModels/ExpansionManager.swift` — `@MainActor` manager of folder expansion state with batched, per-root queued expansion work and folder-ID↔relative-path persistence mapping. Key: `ExpansionManager`, `isExpanded`, `setExpanded`.
- `Features/WorkspaceFiles/ViewModels/FileSystemItemViewModel.swift` — shared protocol for file/folder view models plus the `FileSystemItemType` sum type (folder/file) with id/path/hash/equality. Key: `FileSystemItemViewModel`, `FileSystemItemType`.
- `Features/WorkspaceFiles/ViewModels/FileViewModel.swift` — per-file view model: content loading/freshness, syntax-highlight preview policy (SVG/large-file safety, truncation), token metrics, and search content snapshots. Key: `FileViewModel`, `FilePreviewMode`, `FilePreviewSnapshot`, `FileSearchContentSnapshot`, `FileContentFreshnessPolicy`.
- `Features/WorkspaceFiles/ViewModels/FolderViewModel.swift` — observable folder node holding sorted subfolders/files and a unified `children` array, tri-state checkbox aggregation, expansion, validity, and sort-method override (e.g. system roots). Key: `FolderViewModel`, `children`, `checkboxState`, `CheckboxMetrics`.
- `Features/WorkspaceFiles/ViewModels/GitViewModel.swift` — `@MainActor` git surface: per-root git-repo detection, current branch, worktree-context summaries keyed by canonical path, uncommitted-file list with search index, and diff-inclusion mode. Key: `GitViewModel`, `GitDiffInclusionMode`, `unstagedFiles`, `gitWorktreeContext(forStandardizedRootPath:)`.
- `Features/WorkspaceFiles/ViewModels/WorkspaceFilesViewModel.swift` — central file-list view model: owns root folders, file selection (`selectedFiles`/`selectedFileIDs`), checkbox propagation, sorting, path-based selection, and selection-driven file-list projection for the loaded workspace. Key: `WorkspaceFilesViewModel`, `selectedFiles`, `selectFiles(withPaths:)`, `RemovedFolderPathMatcher`.
- `Features/WorkspaceFiles/ViewModels/WorkspaceFilesViewModel+RootShells.swift` — extension exposing lightweight root projections and visible-root filters (excluding system roots) over `rootFolders`. Key: `rootShellProjections`, `visibleRootShellProjections`, `visibleRootFolders`.
- `Features/WorkspaceFiles/ViewModels/WorkspaceRootShellProjection.swift` — small Sendable value describing a root (id/name/full+standardized path/isSystemRoot) for cheap cross-layer hand-off. Key: `WorkspaceRootShellProjection`.

## Workspaces/

### ViewModels/
- `Features/Workspaces/ViewModels/WorkspaceManagerViewModel.swift` — top-level workspace manager: load/decode (with a file-decode cache), save (poll/debounce/explicit), create from draft, switch, apply presets, duplicate detection/cleanup, and active-workspace tracking; defines on-disk storage paths. Key: `WorkspaceManagerViewModel`, `WorkspaceStoragePaths`, `activeWorkspace`, `createWorkspace`, `applyPreset`, `WorkspaceFileLoadResult`.
- `Features/Workspaces/ViewModels/WorkspaceSaveDiagnostics.swift` — typed `WorkspaceSaveSource` literals enumerating every save trigger (poll timer, switch, preset ops, root mutations, normalization, MCP) for save attribution/diagnostics. Key: `WorkspaceSaveSource`.

### Views/
- `Features/Workspaces/Views/ManagePresetsView.swift` — sheet for editing a workspace's presets (rename/reorder/delete/apply) with a shared hover button style. Key: `ManagePresetsView`, `HoverButtonStyle`.
- `Features/Workspaces/Views/ManageWorkspacesView.swift` — main workspace-management view: create-draft entry, search/list, rename, hide toggle, global-storage view, and duplicate-cleanup confirmation. Key: `ManageWorkspacesView`, `duplicateGroups`.
- `Features/Workspaces/Views/OptimizedPresetRow.swift` — render-minimizing row for a single preset with reorder/switch/rename/delete callbacks. Key: `OptimizedPresetRow`.
- `Features/Workspaces/Views/OptimizedWorkspaceRow.swift` — render-minimizing row for a single workspace with switch/rename/hide/delete callbacks and delete confirmation. Key: `OptimizedWorkspaceRow`.
- `Features/Workspaces/Views/PresetCreationSheet.swift` — minimal name-prompt sheet that creates a preset via the manager. Key: `PresetCreationSheet`.
- `Features/Workspaces/Views/PresetPickerView.swift` — picker listing the active workspace's presets to apply, plus inline create-new-preset. Key: `PresetPickerView`.
- `Features/Workspaces/Views/WorkspaceEntryRootView.swift` — full-window entry router between full-screen onboarding (`setupGuide`) and the workspace chooser (`workspaces`). Key: `WorkspaceEntryRootView`.
- `Features/Workspaces/Views/WorkspaceLandingView.swift` — recent/landing workspace chooser with compact and expanded layouts, greeting/footer slots, and live refresh on `workspaceListDidChange`. Key: `WorkspaceLandingView`, `LayoutStyle`.
- `Features/Workspaces/Views/WorkspacePickerMenu.swift` — compact, font-scale-aware custom popover for switching workspaces (native-menu-like), with optional save/exit actions. Key: `WorkspacePickerMenu`, `WorkspaceMenuQuery`.
- `Features/Workspaces/Views/WorkspaceSetupView.swift` — "Configure Workspace" form bound to the manager's creation draft (name + selected repo paths, add/remove folders). Key: `WorkspaceSetupView`.
- `Features/Workspaces/Views/WorkspaceSwitchLoadingOverlay.swift` — modal overlay shown during a workspace switch with a progress spinner and async cancel. Key: `WorkspaceSwitchLoadingOverlay`.

### (root)
- `Features/Workspaces/WorkspaceCheckoutRefreshService.swift` — coordinates catalog/search/path/codemap freshness after an in-app git checkout mutation; rebuilds the shared visible search index or scopes session worktrees. Key: `WorkspaceCheckoutRefreshService`, `WorkspaceCheckoutRefreshResult`, `WorkspaceCheckoutSearchIndexRefreshBehavior`, `refreshAfterCheckoutMutation`.
- `Features/Workspaces/WorkspaceModel.swift` — the workspace data model and its persisted parts: repo paths, presets, compose-tab/stash state, stored selections, plus duplicate-detection key/summary/cleanup types. Key: `WorkspaceModel`, `WorkspacePreset`, `ComposeTabState`, `StashedTab`, `StoredSelection`, `WorkspaceRootSetKey`, `WorkspaceDuplicateGroupSummary`, `WorkspaceDuplicateCleanupResult`.
- `Features/Workspaces/WorkspaceRootActions.swift` — pure helpers for reordering/standardizing/uniquing workspace repo paths (canonical-key based move up/down respecting visible-root order). Key: `WorkspaceRootActions`, `WorkspaceRootMoveDirection`, `movedRepoPaths`, `standardizedUniqueRepoPaths`.
- `Features/Workspaces/WorkspaceSwitchingModels.swift` — value types for the switch flow: result outcome, active-session item/snapshot, and the confirmation prompt summarizing sessions that will be terminated. Key: `WorkspaceSwitchResult`, `WorkspaceSwitchSessionItem`, `WorkspaceSwitchSessionSnapshot`, `WorkspaceSwitchConfirmation`.
- `Features/Workspaces/WorkspaceSwitchSessionProviders.swift` — concrete `WorkspaceSwitchSessionProvider`s that report and cancel active chat and context-builder sessions during a switch. Key: `ChatWorkspaceSwitchSessionProvider`, `ContextBuilderWorkspaceSwitchSessionProvider`.
- `Features/Workspaces/WorkspaceSwitchSessionRegistry.swift` — `@MainActor` registry aggregating session providers into a snapshot and cancelling all active sessions before a switch. Key: `WorkspaceSwitchSessionRegistry`, `WorkspaceSwitchSessionProvider`, `snapshot`, `cancelActiveSessions`.

_Indexed 46 files._
