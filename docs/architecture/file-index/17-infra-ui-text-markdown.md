# Infrastructure — UI Text, Markdown & Mentions — File Index
> Scope: `Infrastructure/UI/{Markdown,Mentions,Services,TextField}`. 27 files.

## Subsystem role
This is the reusable UI substrate for text rendering and text input in the macOS app. `Markdown/` turns model/chat markdown into highlighted `NSAttributedString`s (a Markdownosaur-derived compiler, a regex syntax highlighter with caching, streaming-aware scheduling, and file-link click handling). `TextField/` is the TextKit/AppKit input stack: plain and attributed `NSViewRepresentable` editors plus an `@`-mention engine (text-view detection, coordinator, floating overlay, custom layout-manager token drawing). `Mentions/` holds the shared mention models, assets, and the file/folder suggestion services those views query. `Services/` provides small async UI helpers (await-user-response, mouse-event monitoring, open-panel wrapping).

## Markdown/
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/EnhancedMarkdownCompiler.swift` — `Markdown.MarkupVisitor` that compiles markdown markup into a styled `NSAttributedString` (headings, lists, tables, code, optional monospaced/force-color); adapted from Markdownosaur. Key: `EnhancedMarkdownCompiler`, `OrderedListMarkerGenerator`.
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/EnhancedMarkdownView.swift` — SwiftUI view that renders a complete pre-built attributed string, regenerating off-main on text/font changes and hosting it via `AttributedTextView`. Key: `EnhancedMarkdownView`.
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/MarkdownTextView.swift` — Streaming-aware markdown SwiftUI view: defines render signature/append-delta/cadence types that decide skip-vs-compile during token streaming, then renders. Key: `MarkdownTextView`, `MarkdownRenderSignature`, `MarkdownStreamingAppendDelta`, `MarkdownRenderCadence`, `MarkdownRenderSchedulingDecision`.
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/CodeHighlighter.swift` — Language-agnostic regex syntax highlighter applying color attributes; compiles rules once, chunks input, and skips oversized/HTML-dense blocks to avoid UI stalls. Key: `CodeHighlighter`.
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/CodeHighlightCache.swift` — `@MainActor` `NSCache`-backed cache of pre-highlighted code attributed strings keyed by `language|hash|fontSize`, delegating to `CodeHighlighter`. Key: `CodeHighlightCache`.
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/PreHighlightedCodeBlock.swift` — SwiftUI view that displays an already-highlighted attributed code string with code-block background/border and a hover copy button (no re-parsing). Key: `PreHighlightedCodeBlock`.
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/MarkdownFileLinkInteraction.swift` — Parses a markdown link destination into a normalized file path + optional line number (rejects http/https/mailto), enabling click-to-open of local file references. Key: `MarkdownFileLinkTarget`.
- `Sources/RepoPrompt/Infrastructure/UI/Markdown/TextViewRangeSafety.swift` — `NSRange`/`NSTextView` extensions for range clamping and safe selection restore after attributed-string replacement, preventing out-of-bounds crashes. Key: `NSRange.clamped(to:)`, `TextViewSelectionRestorePolicy`.

## Mentions/
- `Sources/RepoPrompt/Infrastructure/UI/Mentions/MentionModels.swift` — Public mention data model: suggestion rows, token payload attached to attributed ranges, kinds, and the `.mentionToken` attribute key. Key: `MentionSuggestion`, `MentionTokenPayload`, `MentionKind`, `NSAttributedString.Key.mentionToken`.
- `Sources/RepoPrompt/Infrastructure/UI/Mentions/MentionAssets.swift` — App-lifetime singleton of heavy shared mention resources (compiled `@token` regex, folder/file SF Symbols) with background prewarm. Key: `MentionAssets`.
- `Sources/RepoPrompt/Infrastructure/UI/Mentions/MentionSuggestionService.swift` — `@MainActor` service supplying folder/file `@`-mention suggestions over a `WorkspaceFilesViewModel`, including the synthetic "Selected" pseudo-folder and folder browsing. Key: `MentionSuggestionService`.
- `Sources/RepoPrompt/Infrastructure/UI/Mentions/AgentFileTagSuggestionService.swift` — `@MainActor` fuzzy file-tag suggestion engine for agent input: caches workspace file candidates per lookup context and scores/ranks them into `MentionSuggestion`s. Key: `AgentFileTagSuggestionService`.
- `Sources/RepoPrompt/Infrastructure/UI/Mentions/FileMentionPickerStyle.swift` — User-facing picker style enum (compact/expanded) and its derived configuration (max results, visible rows, overlay width, subtitles). Key: `FileMentionPickerStyle`, `FileMentionPickerConfiguration`.

## Services/
- `Sources/RepoPrompt/Infrastructure/UI/Services/AsyncUserWaiter.swift` — Generic `@MainActor` "await a user response with optional timeout" primitive built on a checked continuation; used by question/instruction flows. Key: `AsyncUserWaiter<Response>`.
- `Sources/RepoPrompt/Infrastructure/UI/Services/InputMonitor.swift` — AppKit local mouse-event monitor with throttling/dedup/state tracking that surfaces mouse-down, drag, and double-click callbacks. Key: `InputMonitor`.
- `Sources/RepoPrompt/Infrastructure/UI/Services/OpenPanelService.swift` — `@MainActor` shared wrapper around `NSOpenPanel` exposing async folder and image-file pickers, preferring sheet presentation over blocking `runModal()`. Key: `OpenPanelService`.

## TextField/
- `Sources/RepoPrompt/Infrastructure/UI/TextField/TextKitView.swift` — SwiftUI `NSViewRepresentable` wrapping an `NSTextView` for plain text (spell-check, mono font, wrap/horizontal-scroll, edit-change callbacks). Key: `TextKitView`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/AttributedTextKitView.swift` — `NSViewRepresentable` editor that adds `@`-mention support (shared token regex, mention suggestions, commit-to-token) on top of `TextKitView`'s API. Key: `AttributedTextKitView`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/ConstrainedTextKitVIew.swift` — Fixed-height `TextKitView` fork that disables soft-wrap and vertical scroll, allowing only horizontal panning. Key: `ConstrainedTextKitView`, `NoVerticalScrollView`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/MentionTextView.swift` — `NSTextView` subclass that detects `@…` ranges, converts accepted suggestions into attributed tokens, and repairs TextKit invariants; defers heavy logic to a delegate. Key: `MentionTextView`, `MentionTextViewDelegate`, `MentionNavigationCommand`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/MentionCoordinator.swift` — `@MainActor` brain of the mention flow: implements `MentionTextViewDelegate`, debounces queries to `MentionSuggestionService`, manages folder-drill levels, and drives the overlay/commit. Key: `MentionCoordinator`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/MentionOverlayController.swift` — `@MainActor` controller for the floating suggestion popup window: placement (above/below), screen-bounds clamping, sizing, and row-click forwarding. Key: `MentionOverlayController`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/MentionSuggestionView.swift` — SwiftUI suggestion list/row hosted in the borderless overlay window, bridging AppKit mutable state into SwiftUI observation. Key: `MentionSuggestionListModel`, `MentionSuggestionRowView`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/MentionDrawingLayoutManager.swift` — `NSLayoutManager` subclass that draws mention-token background pills by enumerating the `.mentionToken` attribute, with container-safety guards. Key: `MentionDrawingLayoutManager`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/ResizableTextField.swift` — User-resizable SwiftUI text field built on an image-paste-aware `NSTextView`, with pasteboard image handling and feature flags. Key: `ResizableTextField`, `ResizableTextFieldFeatures`, `ImageAwareTextView`, `ImagePasteboardTypes`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/TextFieldMentionHelpers.swift` — `@MainActor` helper wiring file-tag mentions into an `ImageAwareTextView`: owns an above-placed overlay, debounced suggestion tasks, and commit finalization via `AgentFileTagSuggestionService`. Key: `FileTagMentionHelper`.
- `Sources/RepoPrompt/Infrastructure/UI/TextField/TextFieldResizeHandle.swift` — SwiftUI drag handle view (grip dots + resize cursor) that adjusts a bound height between min/max. Key: `TextFieldResizeHandle`.

_Indexed 27 files._
