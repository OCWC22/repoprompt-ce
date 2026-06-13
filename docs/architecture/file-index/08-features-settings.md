# Settings — File Index

> Scope: `Features/Settings`. 46 files.

## Subsystem role

The Settings feature owns the dedicated, modeless Settings `NSWindow` and the dozens of SwiftUI panes it routes to via the `SettingsTab` enum + searchable sidebar. Cross-workspace and window preferences persist through a versioned JSON document (`globalSettings.json` under `~/Library/Application Support/RepoPrompt CE/Settings/`), read/written by `GlobalSettingsFileStore` and surfaced to views through the `GlobalSettingsStore`/`GlobalSettingsManager` accessors, with per-window in-memory overlays in `WindowSettingsManager` (no `UserDefaults` for the migrated keys; `SettingKeys` centralizes the few remaining `@AppStorage` literals). Each pane is a small `View` bound either to that store or to a feature view model (API/CLI providers, model presets, agent-mode permissions/models/workflows, MCP, copy/chat presets), following a consistent "section + binding" pattern. The two ViewModels here back the heavyweight API-providers screen and the secure-storage repair flow.

## Models/

- `Features/Settings/Models/GlobalSettingsDocument.swift` — Versioned (`schemaVersion` 2) `Codable` document persisted as `globalSettings.json`: per-workspace copy/chat settings, global defaults, and optional scalar preference groups. Key: `GlobalSettingsDocument`, `currentSchemaVersion`, `replacing(...)`, UUID-keyed dict encode/decode.
- `Features/Settings/Models/GlobalSettingsFileStore.swift` — File-backed reader/writer for the global settings document with default-location resolution, schema-version guarding, and corrupt/future-schema preservation. Key: `GlobalSettingsFileStoring`, `GlobalSettingsFileStore`, `load()`, `loadOrCreateDefault()`, `save(_:)`, `defaultFileURL()`.
- `Features/Settings/Models/GlobalSettingsManager.swift` — Largest model file: defines canonical `SettingKeys`, the per-workspace `CopyGlobalSettings`/`ChatGlobalSettings` structs, global defaults, and the store layer (`GlobalSettingsStore`) exposing typed scalar accessors over the JSON document. Key: `SettingKeys`, `CopyGlobalSettings`, `ChatGlobalSettings`, `GlobalSettingsStore`, `appSettingsFileSystemPreferencesDidChange`.
- `Features/Settings/Models/WindowSettingsManager.swift` — Per-window settings overlay implementing `SettingsManaging`: clones from the global store on first access, holds in-memory copy/chat overrides, and commits/discards them per workspace. Key: `SettingsManaging` (protocol), `WindowSettingsManager`, `commitWorkspace(_:)`, `discardWindowOverrides(for:)`.

## ViewModels/

- `Features/Settings/ViewModels/APISettingsViewModel.swift` — Very large `ObservableObject` driving API/CLI provider configuration: key validation, custom OpenAI-compatible provider testing, model discovery, and per-provider state for the API/CLI panes. Key: `APISettingsViewModel`, `CustomProviderValidationError`.
- `Features/Settings/ViewModels/SecureStorageRepairViewModel.swift` — `@MainActor` VM for the legacy-Keychain repair flow: scans for legacy secure-storage accounts, imports them (with conflict resolution), and deletes legacy entries. Key: `SecureStorageRepairViewModel`, `scan()`, `importAccount(_:replaceExistingTarget:)`, `deleteLegacy(_:)`.

## Views/General/

- `Features/Settings/Views/General/AppearanceSettingsView.swift` — Appearance/general pane bound to `GlobalSettingsStore`: appearance mode, tooltip and latest-file-changes toggles, file-mention picker style, experimental editor flag, spell-check. Key: `AppearanceSettingsView`, `appearanceModeBinding`.
- `Features/Settings/Views/General/LicenseUpdatesSettingsView.swift` — Software-updates pane wired to `SparkleUpdaterManager`: shows update availability, check/install buttons, and the auto-check toggle. Key: `LicenseUpdatesSettingsView`.
- `Features/Settings/Views/General/PromptSettingsView.swift` — Prompt-packaging pane: datetime-in-instructions toggle and import/export/reset of saved instruction prompts via `PromptViewModel`. Key: `PromptSettingsView`, `exportPrompts(to:)` use.

## Views/Components/

- `Features/Settings/Views/Components/ReadOnlyInputBox.swift` — Reusable read-only, selectable text box styled like an input field with placeholder fallback (single- or multi-line). Key: `ReadOnlyInputBox`.
- `Features/Settings/Views/Components/SettingsChatPresetPickerPopover.swift` — Two-pane (preview + list) chat-preset picker popover used in settings, splitting Standard vs MCP-powered presets with hover preview. Key: `SettingsChatPresetPickerPopover<Preview>`, `onSelect`.

## Views/ (root)

- `Features/Settings/Views/AdvancedSettingsView.swift` — Slimmed "Advanced" page grouping File System (gitignore/repoignore/symlinks) and AI Behavior controls, bound to `GlobalSettingsStore`. Key: `AdvancedSettingsView`, `setFileSystemPreference(...)`.
- `Features/Settings/Views/AgentDirectProviderPermissionsView.swift` — Direct-agent scope of Agent Permissions: editable provider-native permission/tool rows for agents launched directly in RepoPrompt. Key: `AgentDirectProviderPermissionsView`, `AgentProviderPermissionsSettingsViewModel`.
- `Features/Settings/Views/AgentModeGeneralSettingsView.swift` — Agent Mode "Overview" tab: compact summary/deep-link rows to canonical Agent Models / Permissions / Context Builder pages plus CLI provider connection status. Key: `AgentModeGeneralSettingsView`, `onNavigate`.
- `Features/Settings/Views/AgentModelsSettingsView.swift` — Unified Agent Models page: single home for Oracle, Built-in Chat, Context Builder agent, and MCP role-default model choices with inline auto-recommendations. Key: `AgentModelsSettingsView`, `AgentModelsSettingsViewModel`.
- `Features/Settings/Views/AgentModeWorkflowsSettingsView.swift` — Settings-native management of Agent Mode workflow prompts (featured/built-in/custom) backed by `AgentWorkflowStore`. Key: `AgentModeWorkflowsSettingsView`, `AgentWorkflow`/`AgentWorkflowDefinition`.
- `Features/Settings/Views/AgentPermissionSettingsComponents.swift` — Shared UI pieces + layout constants for the Agent Permissions shell/subviews (group box, capability rows/chips, risk badge, secure-storage banner). Key: `AgentPermissionSettingsLayout`, `AgentPermissionSettingsGroupBox`, `AgentPermissionRiskBadge`.
- `Features/Settings/Views/AgentPermissionsSettingsView.swift` — Unified Agent Permissions tab hosting Direct-Agents and Sub-Agents scopes under one sidebar entry, sharing a secure-storage diagnostics banner. Key: `AgentPermissionsSettingsView`, `AgentPermissionSettingsScope`.
- `Features/Settings/Views/AgentProviderPermissionControlsComponents.swift` — Stateless provider-native permission/tool control views (Codex/Claude tool sections, permission-level section) reused by Agent Permissions and CLI Providers. Key: `AgentProviderPermissionLevelSection`, `AgentPermissionChromeBinding`.
- `Features/Settings/Views/AgentSubagentPolicySettingsView.swift` — Sub-Agents scope pane: the sub-agent sandbox override policy (Safe Managed / Inherit / Custom) applied when an agent launches a sub-agent via MCP. Key: `AgentSubagentPolicySettingsView`, `AgentSubagentPermissionsSettingsViewModel`.
- `Features/Settings/Views/AIModelDropDown.swift` — Model-selection dropdown menu (Claude Code backend grouping, snapshotting) writing to a `ModelDestination`; used by chat/oracle model pickers in settings. Key: `AIModelDropdown`, `ModelDestination`.
- `Features/Settings/Views/AIQueryBasicSettingsView.swift` — Inner AI-query controls: file-edit-format picker (diff-capability gated) and a temperature disclosure slider via `PromptViewModel`. Key: `AIQueryBasicSettingsView`.
- `Features/Settings/Views/AIQuerySettingsView.swift` — Thin scroll-container wrapper that hosts `AIQueryBasicSettingsView` at a fixed width. Key: `AIQuerySettingsView`.
- `Features/Settings/Views/APISettingsView.swift` — Large "API Providers" pane (Oracle + provider API keys for Anthropic/OpenAI/Gemini/Azure/etc.) with per-provider loading state and the secure-storage-repair entry point. Key: `APISettingsView`, `APISettingsViewModel`.
- `Features/Settings/Views/ChatPresetsSettingsView.swift` — Chat-preset management list (search/filter/add/edit/duplicate/delete) over `ChatPresetManager`; embeddable inside the unified Workflow Presets surface. Key: `ChatPresetsSettingsView`, `PresetFilter`, `embedded`.
- `Features/Settings/Views/ChatSettingsView.swift` — Built-in chat UI settings: Built-in Chat Model, MCP Oracle model presets toggle, and planning prompt; storage via `GlobalSettingsStore`/`ModelPresetsManager`. Key: `ChatSettingsView`, `showModelPresets`.
- `Features/Settings/Views/ClipboardSettings.swift` — Small clipboard/copy pane: toggles for including saved prompts, files, and user instructions in clipboard copies. Key: `ClipboardSettingsView`.
- `Features/Settings/Views/CLIProvidersSettingsView.swift` — Largest view file: CLI provider configuration (Claude Code, Codex, OpenCode, Cursor, Z.AI) with login/auth state, inline provider permissions, and deep-link to Agent Permissions. Key: `CLIProvidersSettingsView`, `AgentProviderPermissionsSettingsViewModel`.
- `Features/Settings/Views/CopyPresetsSettingsView.swift` — Copy-preset management list (search/filter/add/edit/duplicate/delete) over `CopyPresetManager`; embeddable inside Workflow Presets. Key: `CopyPresetsSettingsView`, `PresetFilter`, `embedded`.
- `Features/Settings/Views/CustomProviderSettingsView.swift` — OpenAI-compatible custom provider configuration: endpoint/model entry, model search + enable toggles, and validate-&-save flow via `APISettingsViewModel`. Key: `CustomProviderSettingsView`, `filteredModels`.
- `Features/Settings/Views/GeneralSettingsView.swift` — General settings pane (also defines the `AppearanceMode` enum): appearance, instructions font size, and editor preferences via `GlobalSettingsStore`/`PromptViewModel`. Key: `GeneralSettingsView`, `AppearanceMode`.
- `Features/Settings/Views/KeyboardShortcutsSettingsView.swift` — Global keyboard-shortcuts pane plus its catalog (single source of truth for `KeyboardShortcuts` bindings shown in Settings). Key: `KeyboardShortcutsSettingsView`, `KeyboardShortcutCatalog`, `KeyboardShortcutCatalogSection`.
- `Features/Settings/Views/MCPSettingsView.swift` — MCP server settings pane: auto-start, model-presets visibility, temporary-disable-presets, etc., with scalar prefs flowing through `GlobalSettingsStore`. Key: `MCPSettingsView`, `MCPServerViewModel`.
- `Features/Settings/Views/MCPToolsSettingsView.swift` — Per-window MCP tool enable/disable list with search, backed by `ToolAvailabilityStore` and the window's `MCPServerViewModel`. Key: `MCPToolsSettingsView`, `windowToolsEnabled`.
- `Features/Settings/Views/ModelOverrideSettings.swift` — Singleton `ObservableObject` storing per-model overrides (diff editing, streaming, temperature, Responses-API route) in the JSON-backed store. Key: `ModelOverridesSettings`, `diffOverride(for:)`.
- `Features/Settings/Views/ModelOverrideSettingsView.swift` — Per-model overrides UI grouped by provider with expandable rows, bound to `ModelOverridesSettings` and `APISettingsViewModel`. Key: `ModelOverridesSettingsView`, `computeSections(from:)`.
- `Features/Settings/Views/ModelPresetsSettingsView.swift` — Model-presets settings list (add/edit via `ModelPresetEditView`) with a "presets temporarily hidden" banner; backed by `ModelPresetsManager`. Key: `ModelPresetsSettingsView`.
- `Features/Settings/Views/ModelPresetsSheet.swift` — Standalone sheet variant of model-presets management (header + list + add/edit sheets) over `ModelPresetsManager`. Key: `ModelPresetsSheet`.
- `Features/Settings/Views/OpenRouterSettingsView.swift` — OpenRouter provider pane: API key entry/validation plus advanced configuration (custom headers) via `APISettingsViewModel`. Key: `OpenRouterSettingsView`.
- `Features/Settings/Views/OptimizedModelPicker.swift` — Performance-tuned model picker (caches grouped/sorted models, nested Provider→Models menu) writing to a `ModelDestination`. Key: `OptimizedModelPicker`, `WidthStyle`.
- `Features/Settings/Views/PermissionsSettingsView.swift` — Workspace-operation approvals pane: global approval settings, per-operation policy, and trusted-clients management via `WorkspaceApprovalManager`. Key: `PermissionsSettingsView`.
- `Features/Settings/Views/PromptOrderingSettingsMenu.swift` — Copy-prompt section-ordering UI with drag/hover and a greyed second User-Instructions row; persists order through `PromptViewModel`/`GlobalSettingsStore`. Key: `PromptOrderSettingsView`, `duplicateUserInstructionsAtTop`.
- `Features/Settings/Views/SecureStorageRepairView.swift` — Legacy secure-storage repair UI: an entry banner (`SecureStorageRepairBanner`) and the repair sheet that lists/imports/deletes legacy Keychain accounts via `SecureStorageRepairViewModel`. Key: `SecureStorageRepairView`, `SecureStorageRepairBanner`.
- `Features/Settings/Views/SettingsView.swift` — Root Settings container: searchable sidebar (`List(.sidebar)` + vibrancy), section ordering, and the canonical `SettingsTab` enum that routes every pane. Key: `SettingsView`, `SettingsTab`, `TabSection`, `sidebarSectionOrder`.
- `Features/Settings/Views/SettingsWindowCoordinator.swift` — Owns the dedicated modeless Settings `NSWindow`: lifecycle, title-sync to active tab, frame autosave, and font-scaled sizing. Key: `SettingsWindowCoordinator`, `SettingsWindowModel`, `frameAutosaveName`.
- `Features/Settings/Views/SettingsWindowRootView.swift` — Thin root view that wires `SettingsView` to a window's `WindowState`/`SettingsWindowModel` and injects shared environment objects (managers, font scale). Key: `SettingsWindowRootView`.
- `Features/Settings/Views/SystemSettings.swift` — System/appearance pane bound to `GlobalSettingsStore` (appearance mode, transparency) plus a `SparkleUpdaterManager` reference. Key: `SystemSettingsView`.
- `Features/Settings/Views/WorkflowPresetsSettingsView.swift` — Unified "Workflow Presets" tab that hosts both `CopyPresetsSettingsView` and `ChatPresetsSettingsView` under a Copy/Chat scope picker (view-level composition, no model merge). Key: `WorkflowPresetsSettingsView`, `Scope`.

_Indexed 46 files._
