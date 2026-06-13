# Infrastructure — UI Components — File Index
> Scope: `Infrastructure/UI/{Components,Agent,Composer,Tooltip}`. 46 files.

## Subsystem role
The reusable, cross-feature SwiftUI/AppKit substrate that the rest of the app composes on top of. It supplies the shared button styles, checkboxes, code/text rendering views, MCP and workspace approval overlays, popover and menu primitives, hover/debounce/scroll helpers, color and corner extensions, agent approval/question cards, the shared composer chrome, and the floating tooltip overlay system. Components here are intentionally feature-agnostic (font-scale aware, light/dark adaptive, AppKit-bridged where SwiftUI falls short) so Chat, Agent Mode, Settings, and the toolbar can all reuse them without duplication.

## Components/
- `Infrastructure/UI/Components/AppSidebarSizing.swift` — sidebar width constants and font-preset-scaled width resolvers for the app and agent sidebars. Key: `AppSidebarSizing`, `AgentSidebarSizing`.
- `Infrastructure/UI/Components/BubbleColors.swift` — adaptive (light/dark) color palette for chat bubbles, copy icons, and related UI. Key: `BubbleColors`.
- `Infrastructure/UI/Components/ButtonStyles.swift` — shared pill/round/selector/popover button styles plus font-preset scaling helpers and hover effects. Key: `ButtonScale`, `CustomButtonStyle`, `HoverableButton`, `SmallRoundButtonStyle`, `SelectorButtonStyle`, `PopoverButtonStyle`, `RoundedBorderButtonStyle`, `SimpleHoverEffect`.
- `Infrastructure/UI/Components/CGColorExtensions.swift` — appearance-aware hover-outline color returned directly as a `CGColor` for `CALayer.borderColor`. Key: `CGColor.hoverOutline`.
- `Infrastructure/UI/Components/CheckboxView.swift` — tri-state (checked/unchecked/mixed) checkbox button with hover background. Key: `CheckboxView`.
- `Infrastructure/UI/Components/CheckRecommendationsButton.swift` — button + banner that post a notification to open the recommendation wizard for a window. Key: `CheckRecommendationsButton`, `RecommendationSetupBanner`.
- `Infrastructure/UI/Components/CodeBlockView.swift` — code block renderers: plain copyable block, syntax-highlighted block, and collapsible variant. Key: `CodeBlock`, `SyntaxHighlightedCodeBlock`, `CollapsibleCodeBlock`.
- `Infrastructure/UI/Components/CodeViewMetrics.swift` — centralized layout constants (line height, gutter widths, copy-button offset) for code views. Key: `CodeViewMetrics`.
- `Infrastructure/UI/Components/CollapsibleUserMessage.swift` — shared collapse/expand user-message bubble used by Chat and Agent Mode, with content-width observation. Key: `CollapsibleUserMessage`.
- `Infrastructure/UI/Components/CompactDualActionButton.swift` — compact two-part button (main action + secondary icon action) with per-part hover/press state. Key: `CompactDualActionButton`.
- `Infrastructure/UI/Components/ContinuousMouseTrackingView.swift` — `NSViewRepresentable` reporting continuous mouse move/enter/exit positions to SwiftUI. Key: `ContinuousMouseTrackingView`, `MouseTrackingNSView`.
- `Infrastructure/UI/Components/DebouncedHoverModifier.swift` — view modifier that debounces hover events via a Combine subject. Key: `DebouncedOnHoverModifier`, `View.debouncedOnHover`.
- `Infrastructure/UI/Components/Debouncer.swift` — generation-token gate that coalesces delayed work items so only the latest runs. Key: `WorkItemGate`.
- `Infrastructure/UI/Components/DualClickPopoverButton.swift` — split button whose main part fires an action and secondary part opens a popover, with keyboard-shortcut support. Key: `DualActionButton`.
- `Infrastructure/UI/Components/EnhancedTextkitView.swift` — empty placeholder file (0 bytes); no declarations. Key: (none).
- `Infrastructure/UI/Components/ExpandableBottomPanel.swift` — reusable floating bottom panel with material background, expand/collapse, and optional resize handle (terminal / agent file-tree panels). Key: `ExpandableBottomPanel`.
- `Infrastructure/UI/Components/FileButtonStyle.swift` — bordered control-background button style for file-row buttons. Key: `FileButtonStyle`.
- `Infrastructure/UI/Components/HighlightTextKitView.swift` — `NSTextView` wrapper with Neon/Tree-sitter syntax highlighting and editing. Key: `HighlightedTextKitView`, `NamedRange.nsRange`.
- `Infrastructure/UI/Components/IconBadgeButtonStyle.swift` — minimal, no-clip button style for toolbar icon buttons that carry an overlay badge. Key: `IconBadgeButtonStyle`.
- `Infrastructure/UI/Components/MCPApprovalOverlayView.swift` — full-screen blocking takeover overlay for MCP client approval requests. Key: `MCPApprovalOverlayView`.
- `Infrastructure/UI/Components/MCPConnectionsCounter.swift` — compact live-updating MCP connections counter button that opens the status window. Key: `MCPConnectionsCounter`.
- `Infrastructure/UI/Components/MCPServerToggleView.swift` — toolbar toggle + popover content controlling per-window MCP tool enablement and visual state. Key: `MCPServerToggleView`, `MCPServerPopoverContent`, `MCPToolbarVisualState`.
- `Infrastructure/UI/Components/MCPStatusView.swift` — full MCP server dashboard: connections, live tool activity, auto-approve management. Key: `MCPStatusView`.
- `Infrastructure/UI/Components/ModelDestination.swift` — abstraction decoupling a picked model from where/how it is applied (getter/applier, including binding-backed). Key: `ModelDestination`.
- `Infrastructure/UI/Components/NotificationsButtonView.swift` — bell toolbar item aggregating dismissible/mutable docs, tutorial, and RepoPrompt-101 notifications. Key: `NotificationsButtonView`, `NotificationItem`.
- `Infrastructure/UI/Components/NSColorExtensions.swift` — dynamic-provider `NSColor`s for hover outline and search highlight that track runtime appearance changes. Key: `NSColor.hoverOutlineColor`, `NSColor.searchHighlightColor`.
- `Infrastructure/UI/Components/RecommendationToolbarButtonView.swift` — toolbar button opening the recommendation wizard popover, driven by an observed active-recommendations state. Key: `RecommendationToolbarButtonView`, `RecommendationToolbarStateObserver`.
- `Infrastructure/UI/Components/ResizeHandle.swift` — draggable horizontal resize bar with hover cursor and min/max height clamping. Key: `ResizeHandle`.
- `Infrastructure/UI/Components/RoundedCornerRect.swift` — per-corner rounding shape and `cornerRadius(_:corners:)` view helper. Key: `RoundedCorner`, `RectCorner`, `View.cornerRadius`.
- `Infrastructure/UI/Components/ScrollableCodeTextView.swift` — TextKit `NSScrollView`/`NSTextView` for large monospace code/output with fixed max height and internal scrolling. Key: `ScrollableCodeTextView`.
- `Infrastructure/UI/Components/ScrollOffsetPreferenceKey.swift` — SwiftUI preference key carrying a `CGPoint` scroll offset. Key: `ScrollOffsetPreferenceKey`.
- `Infrastructure/UI/Components/ScrollOffsetReader.swift` — `NSView` that observes its enclosing clip view and reports coalesced, deferred scroll offsets. Key: `ScrollObserverView`.
- `Infrastructure/UI/Components/SettingsButton.swift` — generic icon button that toggles a sheet of arbitrary content. Key: `SettingsButton`.
- `Infrastructure/UI/Components/StableMenuButton.swift` — SwiftUI-labelled button backed by an AppKit `NSMenu` so open menus survive SwiftUI invalidations (model pickers). Key: `StableMenuButton`, `StableMenuItem`, `StableMenuItemStyle`, `StableMenuPresenter`.
- `Infrastructure/UI/Components/TooltipBubble.swift` — font-scaled tooltip bubble view plus tooltip runtime/state container used by the overlay system. Key: `TooltipBubble`, `TooltipRuntime`.
- `Infrastructure/UI/Components/UpdateAvailableToolbarPill.swift` — toolbar pill surfacing Sparkle "update available" state via an observed snapshot. Key: `UpdateAvailableToolbarPill`, `UpdateAvailableToolbarStateObserver`.
- `Infrastructure/UI/Components/WorkspaceApprovalOverlayView.swift` — full-screen blocking overlay for workspace operation approval requests (aligned with the MCP overlay). Key: `WorkspaceApprovalOverlayView`.

## Agent/
- `Infrastructure/UI/Agent/AgentApprovalCard.swift` — inline card presenting an agent approval request with details and approve/deny actions. Key: `AgentApprovalCard`.
- `Infrastructure/UI/Agent/AgentLogEntryRowView.swift` — reusable agent/discovery log row with icon/color mapping and compact/regular styles. Key: `AgentLogEntryRowView`, `AgentLogRowStyle`.
- `Infrastructure/UI/Agent/AgentModelOptionsMenuContent.swift` — model-picker menu content and warning visuals (e.g. fast-Codex usage warning) for agent model selection. Key: `AgentModelSelectionWarningVisuals`, `AgentModelSelectionSummaryLabel`.
- `Infrastructure/UI/Agent/AgentProviderSettingsMenuAction.swift` — "Connect more CLI providers" menu action/section that opens the CLI providers settings tab. Key: `AgentProviderSettingsMenuAction`, `AgentProviderSettingsMenuSection`.
- `Infrastructure/UI/Agent/AgentQuestionCard.swift` — reusable agent question card handling option selection, custom text input, timeout, and submit/skip. Key: `AgentQuestionCard`.
- `Infrastructure/UI/Agent/TimeoutCountdownView.swift` — small countdown timer view for question/instruction timeouts with urgency styling. Key: `TimeoutCountdownView`.

## Composer/
- `Infrastructure/UI/Composer/ComposerChrome.swift` — shared floating composer bubble chrome (background, padding, sizing, bottom-occlusion) used by both Chat and Agent Mode. Key: `ComposerChrome`.

## Tooltip/
- `Infrastructure/UI/Tooltip/AnchorGeometryView.swift` — `NSViewRepresentable` reporting an anchor rect in window coordinates with coalesced/deferred updates to avoid layout loops. Key: `AnchorGeometryView`, `TrackingView`.
- `Infrastructure/UI/Tooltip/TooltipOverlayController.swift` — `@MainActor` controller that shows/repositions/hides the floating tooltip `NSWindow` overlay. Key: `TooltipOverlayController`.

_Indexed 46 files._
