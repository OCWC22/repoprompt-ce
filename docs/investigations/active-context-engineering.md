# Active Context Engineering: RepoPrompt CE Architecture Export

Date: 2026-06-11

Purpose: capture how RepoPrompt CE stores, curates, packages, accounts for, and exports repository context, then translate those patterns into an integration model we can build ourselves.

This is a working architecture note. Add future findings here instead of scattering them across chat transcripts.

## Executive Summary

RepoPrompt CE is best understood as a context engineering workbench, not a token-compression engine.

Its core loop is:

```text
workspace roots
  -> actor-backed file and folder catalog
  -> durable per-tab selection state
  -> resolved full-file, slice, codemap, file-tree, and git-diff entries
  -> token accounting by component
  -> reviewable prompt or MCP DTO export
  -> agent handoff
```

The important design decision is that RepoPrompt keeps context structured until late in the pipeline. Files, slices, CodeMaps, file trees, git diffs, prompt text, presets, and token statistics remain separate long enough to be inspected, modified, and exported through MCP. The final prompt string is just one possible rendering of this structured context state.

For our own Active Context Engineering system, the main lesson is to build around a `ContextPack` abstraction, not a flat prompt string. The optimizer should operate on typed context components with provenance, safety rules, protected spans, token budgets, and receipts.

## What RepoPrompt CE Claims To Do

The README describes RepoPrompt CE as "a free, open-source native macOS app and agent orchestrator for context engineering." It assembles focused, reviewable context from files, CodeMaps, repository structure, and Git diffs, then hands that context to AI tools and CLI agents.

Source anchors:

- `README.md:7`
- `README.md:9`
- `README.md:98`
- `README.md:107`
- `README.md:111`

## Key Mental Model

RepoPrompt has two related but distinct state layers:

1. Durable workspace and tab state

   This is what the user has chosen and what should be restored later: workspace roots, compose tabs, selected paths, slices, prompt text, selected meta prompts, active agent session IDs, copy presets, and Context Builder settings.

2. Runtime workspace context state

   This is the live, actor-backed catalog of what files and folders currently exist under each loaded root, plus CodeMap snapshots, search indexes, file tree snapshots, ingress barriers, and path lookup/materialization behavior.

The durable layer stores intent. The runtime layer resolves that intent against the current filesystem.

## Durable Workspace Storage

Workspace data is stored as JSON files under the user's application support directory:

```text
~/Library/Application Support/RepoPrompt CE/Workspaces/
  workspacesIndex.json
  Workspace-<safe-name>-<uuid>/
    workspace.json
    Chats/
    _git_data/
```

Relevant code:

- `Sources/RepoPrompt/Features/Workspaces/ViewModels/WorkspaceManagerViewModel.swift:42`
- `Sources/RepoPrompt/Features/Workspaces/ViewModels/WorkspaceManagerViewModel.swift:92`
- `Sources/RepoPrompt/Features/Workspaces/ViewModels/WorkspaceManagerViewModel.swift:217`
- `Sources/RepoPrompt/Features/Workspaces/ViewModels/WorkspaceManagerViewModel.swift:701`
- `Sources/RepoPrompt/Features/Workspaces/ViewModels/WorkspaceManagerViewModel.swift:1115`
- `Sources/RepoPrompt/Features/Workspaces/ViewModels/WorkspaceManagerViewModel.swift:1140`

`WorkspaceModel` is the durable top-level record. It includes identity, schema version, workspace name, repo paths, presets, copy and chat preset IDs, compose tabs, stashed tabs, and other per-workspace state.

Relevant code:

- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:321`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:336`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:338`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:350`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:355`

## Per-Tab Context State

Each compose tab is an auto-saved working context. The important fields are:

- `selection`: the file, slice, and CodeMap selection.
- `expandedFolders`: UI/file-tree expansion state.
- `promptText`: the user's task instructions.
- `selectedMetaPromptIDs`: reusable meta instructions.
- `activeChatSessionID` and `activeAgentSessionID`: links to active conversations/runs.
- `contextBuilder`: Context Builder instructions and follow-up settings.

Relevant code:

- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:221`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:230`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:233`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:237`
- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:287`

### StoredSelection

`StoredSelection` is the central durable context-selection structure:

```swift
struct StoredSelection: Codable, Equatable {
    let selectedPaths: [String]
    let autoCodemapPaths: [String]
    let slices: [String: [LineRange]]
    let codemapAutoEnabled: Bool
}
```

Meaning:

- `selectedPaths`: files or folders selected as full context.
- `autoCodemapPaths`: files represented as CodeMaps/API signatures.
- `slices`: path to selected line ranges.
- `codemapAutoEnabled`: whether RepoPrompt automatically adds related CodeMaps.

Relevant code:

- `Sources/RepoPrompt/Features/Workspaces/WorkspaceModel.swift:148`

This is the minimum object our own system should emulate. It captures user/agent intent without embedding file contents directly.

## Runtime Workspace Context Store

The runtime context core is `WorkspaceFileContextStore`, an actor. It keeps per-root file and folder maps, file-system services, child relationships, CodeMap aggregation, path lookup/materialization, and ingress barriers.

Key structures:

- `WorkspaceRootRecord`
- `WorkspaceFolderRecord`
- `WorkspaceFileRecord`
- per-root maps from relative paths to file/folder IDs
- per-root child folder/file ID maps
- file tree snapshot requests
- CodeMap snapshot dictionaries

Relevant code:

- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/WorkspaceFileContextStore.swift:27`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/WorkspaceFileContextStore.swift:111`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/Models/WorkspaceFileContextModels.swift:264`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/Models/WorkspaceFileContextModels.swift:296`

The actor boundary matters. It lets RepoPrompt keep file-system ingress, path lookup, and context snapshots coherent even as the filesystem changes.

For our system, this maps to a `RepositoryCatalog` service:

```text
RepositoryCatalog
  roots
  file records
  folder records
  content metadata
  structural summaries
  path resolver
  search index
  snapshot API
```

## File Tree Snapshots

RepoPrompt can render several file-tree modes:

- `none`
- `selected`
- `full`
- `folders`
- `auto`

The request can also control display style, root scope, legend inclusion, CodeMap markers, start path, and max depth.

Relevant code:

- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/WorkspaceFileContextStore.swift:6`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/WorkspaceFileContextStore.swift:27`

This is important because a file tree is not merely decoration. It is a low-token orientation surface that helps the model understand repo shape before reading detailed contents.

For our system, file-tree output should be a first-class component in the context pack:

```json
{
  "type": "file_tree",
  "mode": "selected",
  "roots": ["repo"],
  "content": "...",
  "tokens": 1200,
  "provenance": {"snapshot_id": "..."}
}
```

## Resolved Prompt Entries

Before packaging, durable selection is resolved into `ResolvedPromptFileEntry` records:

```swift
struct ResolvedPromptFileEntry {
    let file: WorkspaceFileRecord
    let isCodemap: Bool
    let lineRanges: [LineRange]?
    let mode: PromptFileEntryMode
    let loadedContent: String?
    let rootFolderPath: String?
}
```

Modes:

- `fullFile`
- `sliced`
- `codemap`

Relevant code:

- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/Models/WorkspaceFileContextModels.swift:302`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/Models/WorkspaceFileContextModels.swift:342`

This is the key bridge between "what the user selected" and "what the model receives."

Our equivalent should separate:

- requested path
- resolved absolute path
- logical display path
- representation mode
- line ranges
- content hash
- loaded text, if materialized
- structural summary, if available
- token estimate
- safety/provenance metadata

## Slices

Slices are explicit line ranges preserved from source files. RepoPrompt normalizes, clamps, sorts, and merges adjacent or overlapping ranges. If no valid range remains, it falls back to full-file assembly.

Relevant code:

- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/Slices/SliceAssembly.swift:21`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/Slices/SliceAssembly.swift:49`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/Slices/SliceAssembly.swift:87`

Product lesson: slices are a safer compression primitive than semantic deletion for code-editing tasks because the selected bytes remain verbatim and traceable to source lines.

Our system should prefer:

```text
full file -> relevant slices -> codemap/outline -> deletion compression
```

for code tasks, in that order.

## CodeMaps

CodeMaps are structural/API-signature representations. In packaging, a file marked as CodeMap uses `FileAPI.getFullAPIDescription(...)` when available. If a CodeMap is not available, the implementation falls through to full content when content is loaded.

Relevant code:

- `Sources/RepoPrompt/Features/Prompt/Services/PromptPackagingService.swift:336`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptPackagingService.swift:407`

Product lesson: CodeMaps are useful for orientation and dependency awareness, but they are not proof-preserving compression. They should not replace exact code spans when the agent must edit implementation details.

Our system should classify structural summaries as:

```text
representation = "structural_summary"
safe_for = ["orientation", "dependency_map", "symbol_lookup"]
unsafe_for = ["exact_patch", "schema_contract", "line_sensitive_edit"]
```

## Prompt Pre-Assembly

`PromptContextPreAssemblyService.resolve(...)` physicalizes selection, resolves prompt entries, collects CodeMap snapshots, renders the file tree, resolves git diff text, and returns a structured result.

The result still has separate fields:

- `physicalSelection`
- `entries`
- `missingPaths`
- `invalidPaths`
- `codemapSnapshots`
- `fileTreeContent`
- `gitDiff`
- `lookupContext`
- `filePathDisplay`

Relevant code:

- `Sources/RepoPrompt/Features/Prompt/Services/PromptContextPreAssemblyService.swift:63`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptContextPreAssemblyService.swift:82`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptContextPreAssemblyService.swift:116`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptContextPreAssemblyService.swift:186`

This is probably the cleanest internal integration point if we were modifying RepoPrompt itself.

For our own product, this maps to:

```text
ContextResolver.resolve(request) -> PreAssemblyContext
```

where `PreAssemblyContext` is still structured and can be optimized before final rendering.

## Prompt Packaging

Packaging turns resolved entries into XML-like prompt sections:

- `<file_map>`
- `<file_contents>`
- `<git_diff>`
- meta prompt blocks
- user instruction blocks

Relevant code:

- `Sources/RepoPrompt/Features/Prompt/Services/PromptPackagingService.swift:320`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptPackagingService.swift:436`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptPackagingService.swift:461`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptPackagingService.swift:473`
- `Sources/RepoPrompt/Features/Prompt/Services/PromptPackagingService.swift:495`

`PromptAssemblyBuilder` then combines independently produced snippets in caller-supplied order, skipping disabled sections and optionally duplicating user instructions at the top.

Relevant code:

- `Sources/RepoPrompt/Features/Prompt/Models/PromptAssemblyBuilder.swift:12`
- `Sources/RepoPrompt/Features/Prompt/Models/PromptAssemblyBuilder.swift:30`

Product lesson: make final rendering deterministic and sectioned. This improves reviewability, diffability, cache stability, and evaluation.

## Token Accounting

RepoPrompt calculates token counts from an immutable `TokenCalculationSnapshot`:

```swift
struct TokenCalculationSnapshot {
    let promptText: String
    let selectedInstructionsText: String
    let duplicateUserInstructionsAtTop: Bool
    let promptEntries: [PromptFileEntrySnapshot]
    let fileTree: TokenCalculationFileTreeInput
}
```

Each file entry snapshot includes path, CodeMap request flag, line ranges, cached full token count, loaded content, CodeMap content, and CodeMap token count.

Relevant code:

- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/TokenAccounting/TokenCalculationSnapshot.swift:3`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/TokenAccounting/TokenCalculationSnapshot.swift:20`

The token estimate is a cheap UTF-8 byte heuristic:

```swift
Int((Double(bytes) / 4.0) * 1.05)
```

Relevant code:

- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/TokenAccounting/TokenCalculationService.swift:128`

Token calculation evaluates:

- full-file entries
- slice entries
- CodeMap entries
- unresolved CodeMap entries
- file tree content
- prompt text
- selected meta instructions
- duplicate prompt text

Relevant code:

- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/TokenAccounting/TokenCalculationService.swift:251`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/TokenAccounting/TokenCalculationService.swift:299`
- `Sources/RepoPrompt/Infrastructure/WorkspaceContext/TokenAccounting/TokenCalculationService.swift:397`

For our system, token accounting should be provider-specific for production billing and routing. RepoPrompt's heuristic is good enough for local UX feedback, but not enough for strict commercial budget guarantees.

Recommended model:

```text
TokenEstimator
  estimate_local_fast(text, provider_hint)
  count_exact(text, model)
  count_pack_components(context_pack, model)
  explain_delta(before_pack, after_pack)
```

## MCP Selection Tooling

RepoPrompt exposes context mutation to agents through `manage_selection`.

Supported operations:

- `get`
- `add`
- `remove`
- `set`
- `clear`
- `preview`
- `promote`
- `demote`

Representation modes:

- `full`
- `slices`
- `codemap_only`

Relevant code:

- `Sources/RepoPrompt/Infrastructure/MCP/WindowTools/MCPSelectionToolProvider.swift:22`
- `Sources/RepoPrompt/Infrastructure/MCP/WindowTools/MCPSelectionToolProvider.swift:29`
- `Sources/RepoPrompt/Infrastructure/MCP/WindowTools/MCPSelectionToolProvider.swift:31`
- `Sources/RepoPrompt/Infrastructure/MCP/WindowTools/MCPSelectionToolProvider.swift:69`
- `Sources/RepoPrompt/Infrastructure/MCP/WindowTools/MCPSelectionToolProvider.swift:104`

This is one of the most valuable integration patterns: agents should not only read files, they should be able to manage a shared, reviewable context set.

Our system should expose similar operations:

```json
{
  "op": "add",
  "paths": ["Sources/App/Core.swift"],
  "mode": "slices",
  "slices": [
    {
      "path": "Sources/App/Core.swift",
      "ranges": [{"start_line": 40, "end_line": 120}]
    }
  ]
}
```

## MCP Workspace Context Export

`workspace_context` builds a DTO for the current tab-scoped context. It can include:

- prompt text
- selection reply
- file blocks
- selected code structure
- file tree
- token stats
- user token stats
- copy preset context
- worktree scope

Relevant code:

- `Sources/RepoPrompt/Infrastructure/MCP/ViewModels/MCPServerViewModel+WorkspaceContext.swift:5`
- `Sources/RepoPrompt/Infrastructure/MCP/ViewModels/MCPServerViewModel+WorkspaceContext.swift:45`
- `Sources/RepoPrompt/Infrastructure/MCP/ViewModels/MCPServerViewModel+WorkspaceContext.swift:76`
- `Sources/RepoPrompt/Infrastructure/MCP/ViewModels/MCPServerViewModel+WorkspaceContext.swift:96`
- `Sources/RepoPrompt/Infrastructure/MCP/ViewModels/MCPServerViewModel+WorkspaceContext.swift:142`
- `Sources/RepoPrompt/Infrastructure/MCP/ViewModels/MCPServerViewModel+WorkspaceContext.swift:234`

This is the best external integration pattern. A commercial service can consume a structured context DTO, optimize it, and return a structured optimized pack plus receipts.

## Auto-Slice Selection

RepoPrompt can automatically convert agent reads and searches into selection entries during agent-mode runs with virtual context.

Rules include:

- a full-file read becomes full selection
- a partial read becomes a slice
- content search result groups become normalized slice entries
- `AGENTS.md` is excluded from auto-selection

Relevant code:

- `Sources/RepoPrompt/Infrastructure/MCP/AutoSliceSelection.swift:14`
- `Sources/RepoPrompt/Infrastructure/MCP/AutoSliceSelection.swift:18`
- `Sources/RepoPrompt/Infrastructure/MCP/AutoSliceSelection.swift:33`
- `Sources/RepoPrompt/Infrastructure/MCP/AutoSliceSelection.swift:37`
- `Sources/RepoPrompt/Infrastructure/MCP/AutoSliceSelection.swift:67`

Product lesson: context should be incrementally learned from the agent's actual exploration path. Reads and searches are evidence of relevance.

Our system should maintain:

```text
ContextEvidence
  source = read_file | search_result | user_selected | dependency_trace | failing_test | git_diff
  path
  ranges
  confidence
  reason
  timestamp
```

Then turn evidence into context selection.

## Agent Provider Context

For provider handoff, RepoPrompt can build an initial file tree and a fork file-contents block. If selected content exceeds a token cap, it emits a summary instead of full content.

Relevant code:

- `Sources/RepoPrompt/Features/AgentMode/Services/AgentProviderContextBuilder.swift:3`
- `Sources/RepoPrompt/Features/AgentMode/Services/AgentProviderContextBuilder.swift:29`
- `Sources/RepoPrompt/Features/AgentMode/Services/AgentProviderContextBuilder.swift:50`

Provider implementation is separated through a bridge package. The provider package intentionally does not import the main app target; the app owns persistence, secure storage, transcript mutation, permission matching, native process control, and runtime construction.

Relevant code:

- `Packages/RepoPromptAgentProviders/README.md:1`
- `Packages/RepoPromptAgentProviders/README.md:9`
- `docs/architecture/provider-plugins.md`

Product lesson: keep optimization logic provider-neutral and put provider-specific rendering/counting/adapters behind a seam.

## Runtime Compaction Is Separate

RepoPrompt has provider/runtime compaction support, but it is not the same as structured context optimization.

Examples:

- Codex slash command `/compact`
- Claude status `compacting`
- Claude `compact_boundary`
- transcript historical truncation for persistence/export sizing
- compressed transcript groups for UI display

Relevant code:

- `Sources/RepoPrompt/Features/AgentMode/Runtime/Codex/CodexAgentModeCoordinator.swift:39`
- `Sources/RepoPrompt/Features/AgentMode/Runtime/Codex/CodexAgentModeCoordinator.swift:1377`
- `Packages/RepoPromptAgentProviders/Sources/RepoPromptClaudeCompatibleProvider/ClaudeSDKNDJSONTranslator.swift:100`
- `Packages/RepoPromptAgentProviders/Sources/RepoPromptClaudeCompatibleProvider/ClaudeSDKNDJSONTranslator.swift:134`
- `Sources/RepoPrompt/Features/AgentMode/Models/CompressedTranscriptItem.swift:5`
- `Sources/RepoPrompt/Features/AgentMode/Runtime/Transcript/AgentTranscriptHistoricalTruncationPolicy.swift`

Do not confuse these with prompt optimization. Runtime compaction is a provider/session behavior. Active Context Engineering should operate before handoff, over typed context components.

## Recommended ContextPack Model

A service-side `ContextPack` should preserve the structure RepoPrompt keeps internally:

```json
{
  "id": "ctxpack_...",
  "task": {
    "instructions": "Fix the failing parser tests",
    "mode": "code_edit",
    "risk": "medium"
  },
  "workspace": {
    "roots": [
      {
        "name": "repo",
        "path_hash": "sha256:...",
        "vcs": {"branch": "main", "commit": "..."}
      }
    ]
  },
  "components": [
    {
      "id": "tree_1",
      "type": "file_tree",
      "mode": "selected",
      "text": "...",
      "tokens": 1200,
      "provenance": {"source": "snapshot"}
    },
    {
      "id": "file_1",
      "type": "file",
      "path": "Sources/App/Core.swift",
      "representation": "slice",
      "ranges": [{"start_line": 40, "end_line": 120}],
      "text": "...",
      "tokens": 800,
      "provenance": {
        "content_hash": "sha256:...",
        "selection_reason": "agent_read"
      }
    },
    {
      "id": "diff_1",
      "type": "git_diff",
      "text": "...",
      "protected": true
    },
    {
      "id": "instructions_1",
      "type": "user_instructions",
      "text": "...",
      "protected": true
    }
  ],
  "constraints": {
    "max_tokens": 64000,
    "protected_component_ids": ["diff_1", "instructions_1"],
    "preserve_exact_for": ["schema", "patch_target", "tool_json"]
  }
}
```

## Optimizer Policy

Recommended policy order:

1. Preserve protected spans exactly.
2. Keep user instructions, tool JSON, schemas, diffs, and patch targets byte-stable.
3. Prefer removing irrelevant components over rewriting relevant ones.
4. Prefer slices over summaries for code-edit tasks.
5. Prefer CodeMaps/structural summaries for orientation-only files.
6. Use deletion compression for chat history, docs, and low-risk background text.
7. Keep deterministic ordering for cache stability.
8. Emit a receipt for every transformation.

## Optimization Receipt

A useful receipt should answer:

- What was included?
- What was excluded?
- What was transformed?
- Which exact spans were protected?
- Which decisions were based on evidence?
- Which token budget was targeted?
- What was the before/after token count?
- Is the output safe for code editing, planning, review, or only retrieval?

Example:

```json
{
  "pack_id": "ctxpack_...",
  "optimized_pack_id": "ctxpack_opt_...",
  "before_tokens": 104000,
  "after_tokens": 62000,
  "model": "gpt-5-codex",
  "decisions": [
    {
      "component_id": "file_1",
      "action": "slice_preserved",
      "reason": "direct patch target",
      "before_tokens": 9000,
      "after_tokens": 1700,
      "protected": true
    },
    {
      "component_id": "file_2",
      "action": "codemap_substituted",
      "reason": "referenced dependency but not edited",
      "before_tokens": 12000,
      "after_tokens": 900
    }
  ],
  "safety": {
    "exact_patch_targets_preserved": true,
    "schemas_preserved": true,
    "tool_json_preserved": true,
    "requires_replay_eval": false
  }
}
```

## Integration Blueprint

### If Integrating With RepoPrompt CE

Best external seam:

```text
MCP workspace_context DTO
  -> Touchdown ContextPack adapter
  -> optimize/evaluate
  -> optimized prompt or optimized selection proposal
```

Best internal seam:

```text
PromptContextPreAssemblyResult
  -> ContextPack adapter
  -> optimizer
  -> PromptPackagingService-compatible entries/snippets
```

Avoid integrating after `PromptAssemblyBuilder.build()` if the goal is safe optimization. At that point, the system has lost most typed structure.

### If Building Our Own System

Minimum services:

```text
RepositoryCatalog
SelectionStore
ContextResolver
PromptPreAssembler
TokenAccountingService
ContextOptimizer
ReceiptService
ProviderRenderer
MCPOrHTTPFacade
```

Minimum durable state:

```json
{
  "workspace_id": "...",
  "roots": ["..."],
  "tabs": [
    {
      "id": "...",
      "selection": {
        "selected_paths": [],
        "codemap_paths": [],
        "slices": {},
        "auto_codemap_enabled": true
      },
      "prompt_text": "",
      "meta_prompt_ids": [],
      "context_builder": {}
    }
  ]
}
```

Minimum runtime snapshot:

```json
{
  "snapshot_id": "...",
  "root_records": [],
  "file_records": [],
  "folder_records": [],
  "codemap_snapshots": {},
  "file_tree": {},
  "git": {},
  "search_index_version": "..."
}
```

## Fable 5 / Claude Code Repo Navigation Use Case

The first concrete product use case should be Claude Code driving RepoPrompt MCP
to navigate a repository and hand a high-value, token-efficient context pack to
Claude Fable 5 on the API.

This is not another tokenizer, compressor, KV cache, quantization, kernel, or
production-router lane. It is the prompt/context layer above those systems:
which repository facts should be selected, represented, protected, rendered, and
charged to an expensive high-capability model.

Current source facts to treat as planning inputs:

- Claude Fable 5 is already API-available as `claude-fable-5` as of June 9,
  2026.
- June 22, 2026 is the end of the temporary subscription inclusion window, not
  the API launch date.
- Fable 5 has a 1M token context window by default and up to 128k output tokens.
- Pricing is $10 per million input tokens and $50 per million output tokens.
- Prompt caching keeps the existing 90% input token discount.
- Fable 5 requires 30-day data retention and is not available under zero data
  retention.
- Integrations must handle `stop_reason: "refusal"`, fallback metadata, and
  billing differences when a request is refused before output.

Primary sources:

- `https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5`
- `https://platform.claude.com/docs/en/release-notes/overview`
- `https://www.anthropic.com/claude/fable`

### Core Loop

```text
RepoPrompt MCP search/read/selection evidence
  -> ContextPack
  -> optimized selection proposal
  -> Fable 5 render
  -> receipt-backed coding result
```

The goal is not to make Fable 5 read everything. The goal is to make Claude Code
use RepoPrompt MCP as the context workbench so Fable 5 sees only the context that
is valuable for the current coding step.

### ContextPack Contract for Claude Code

The Claude Code pack should include:

- task instructions and user constraints;
- workspace roots, branch, commit, dirty state, and worktree identity;
- selected files with representation mode: `full`, `slice`, or `codemap`;
- file tree orientation component;
- git diff and merge/patch context as protected components;
- AGENTS.md, RTK.md, and repo-local instruction files as protected components;
- tool schemas and tool JSON as protected components when present;
- token counts by component and by representation mode;
- selection evidence from reads, searches, user selection, dependency traces,
  failing tests, and git diff;
- provider target metadata: `provider = "anthropic"`,
  `model = "claude-fable-5"`, context limit, output limit, retention class, and
  fallback policy;
- protected spans and safety status.

The pack is a structured object until the last moment. The final prompt string is
only one rendering of this object.

### SelectionPolicy

Use this policy for Claude Code sessions:

1. Keep user instructions, repo instructions, git diff, schemas, tool JSON, and
   patch targets byte-stable.
2. Use full files only for direct patch targets or small files whose entire
   content is required for correctness.
3. Use slices for likely edit regions, failing functions, parser boundaries,
   test failures, and directly referenced call sites.
4. Use CodeMaps for dependencies, interfaces, and orientation files that do not
   need exact bytes.
5. Use file tree output for repo shape before detailed file content.
6. Exclude unrelated files rather than rewriting relevant files.
7. Use deletion/summarization only for chat history, docs, logs, and low-risk
   background text.
8. Keep deterministic ordering for prompt-cache stability.
9. Require a receipt for every exclusion, representation change, or fallback.

Safe coding default:

```text
direct edit target -> full or slice
referenced dependency -> codemap
repo orientation -> file tree
chat/docs/log background -> deletion or summary only if replay-safe
```

### Fable-Gated Routing

Routing in this use case is an orchestration policy around context selection,
not a production semantic router.

Allowed helper-model roles:

- classify task intent and risk;
- rank likely files from file tree and search hits;
- propose slice ranges;
- identify repeated schema/tool blocks;
- decide whether a Fable call is necessary for the current step;
- run cheap receipt linting or static safety checks.

Required Fable 5 roles:

- final high-capability coding/reasoning over the optimized pack;
- hard architecture decisions;
- edits that require cross-file semantic consistency;
- final review when patch correctness depends on exact repo context.

The receipt should label helper-model decisions as advisory. It must not claim
closed-loop production routing until a live gateway is actually dispatching
traffic.

### FableRenderer

`FableRenderer` should render a `ContextPack` into stable XML-like sections:

```text
<task>
<repo_instructions>
<workspace>
<file_tree>
<selection_evidence>
<file_contents>
<codemaps>
<git_diff>
<tool_schemas>
<constraints>
```

Rendering rules:

- stable section order and stable path ordering for prompt-cache reuse;
- protected components copied exactly;
- slices include path and source line ranges;
- CodeMaps are marked unsafe for exact patching;
- Fable model metadata is included outside the user task, not mixed into patch
  targets;
- multi-turn flows preserve provider thinking-block requirements instead of
  treating runtime compaction as context optimization;
- refusal handling records `stop_reason: "refusal"` and `stop_details` when
  present, then applies the configured fallback policy.

### ContextReceipt

Every Fable-bound pack should emit a receipt:

```json
{
  "pack_id": "ctxpack_...",
  "optimized_pack_id": "ctxpack_opt_...",
  "client": "claude_code",
  "provider": "anthropic",
  "model": "claude-fable-5",
  "before_tokens": 240000,
  "after_tokens": 68000,
  "cache_stability": "stable_ordering",
  "retention_class": "30_day_required",
  "fallback_policy": "retry_on_allowed_claude_model_after_refusal",
  "decisions": [
    {
      "path": "Sources/App/Core.swift",
      "representation": "slice",
      "reason": "direct patch target from search and failing test evidence",
      "ranges": [{"start_line": 40, "end_line": 120}],
      "protected": true
    }
  ],
  "excluded": [
    {
      "path": "docs/archive/old-plan.md",
      "reason": "no evidence from task, search, diff, or dependency trace"
    }
  ],
  "fallback_events": [],
  "safety": {
    "repo_instructions_preserved": true,
    "git_diff_preserved": true,
    "schemas_preserved": true,
    "patch_targets_exact": true,
    "codemap_used_for_patch_target": false
  }
}
```

### End-to-End Workflow

1. Claude Code opens a task and asks RepoPrompt MCP for file tree, search, and
   relevant reads.
2. RepoPrompt auto-selection evidence records reads/searches as full-file or
   slice candidates.
3. A cheap deterministic or helper-model scout ranks files and proposes a
   `manage_selection preview`.
4. RepoPrompt returns a reviewable selection with token stats.
5. The ContextPack builder adapts `workspace_context` into structured
   components with protected spans.
6. The optimizer applies SelectionPolicy and emits an optimized selection
   proposal.
7. FableRenderer produces the Fable 5 prompt with stable section ordering.
8. Claude Code sends the final high-value context to `claude-fable-5`.
9. The result is evaluated by patch application, test outcome, token cost,
   cache stability, refusal/fallback state, and receipt integrity.

### Validation Fixture

The first fixture should be a Claude Code-style task over a medium repository
where naive full-context handoff is wasteful or exceeds the chosen budget.

Acceptance checks:

- direct edit targets stay full or sliced with exact source bytes;
- dependency files become CodeMaps only when not edited;
- unrelated files are excluded with reasons;
- AGENTS.md, RTK.md, user task, git diff, schemas, tool JSON, and patch targets
  are protected;
- naive full-context token count is compared with optimized ContextPack token
  count;
- prompt-cache stability is evaluated through deterministic section ordering;
- simulated Fable refusal records refusal category, fallback model, billing
  state, and final status;
- patch applies and tests pass, or expected failures are reported with receipt
  evidence.

Live RepoPrompt MCP validation is still open in this environment:
`rp-cli -c bind_context -j '{"op":"list"}'` failed because the RepoPrompt app
was not running or MCP was not enabled. Re-run the live binding before claiming
MCP execution proof.

## Product API Sketch

```http
POST /v1/context-packs/build
POST /v1/context-packs/optimize
POST /v1/context-packs/evaluate
GET  /v1/context-packs/{id}/receipt
POST /v1/selection/propose
POST /v1/selection/apply
```

`build` should take repo snapshot references plus task instructions and produce a structured context pack.

`optimize` should take a context pack plus model/budget/safety constraints and produce an optimized pack.

`evaluate` should replay or statically validate that protected facts, schemas, patch targets, and instructions survived.

`selection/propose` should return a reviewable selection change instead of mutating state directly.

For the Fable 5 use case, `build` consumes a RepoPrompt `workspace_context`
snapshot, `optimize` returns a Fable-gated selection proposal, and `evaluate`
records patch/test/fallback outcomes into the receipt.

## What To Reuse Conceptually

Reuse these patterns:

- Durable selection object separate from loaded contents.
- Actor/service-backed runtime catalog separate from durable user intent.
- Full/slice/codemap representation modes.
- File tree as a first-class orientation component.
- Git diff as a separate protected component.
- Token accounting by component.
- MCP/agent tools for selection management.
- Review/preview before handoff.
- Provider-neutral bridge layer.

Do not copy these as-is:

- macOS app storage assumptions.
- byte-count token heuristic for strict billing.
- UI transcript compression as prompt compression.
- provider `/compact` as context optimization.
- CodeMap as a semantic-equivalence guarantee.

## Defaults And Remaining Questions

Defaults resolved by the Fable 5 / Claude Code use case:

- Repository identity should start with branch, commit, dirty state, and
  worktree identity from RepoPrompt's live workspace context.
- Optimization should return both a structured pack and a proposed selection
  mutation; direct mutation requires review or explicit apply.
- Always protect user instructions, repo instructions, git diffs, schemas, tool
  JSON, and direct patch targets.
- Safety for code editing is proved through exact protected-span preservation,
  patch application, test outcome, and receipt integrity.
- First exact provider accounting target is Anthropic/Fable 5.
- First external surface should be MCP plus HTTP-compatible service endpoints.
- First quality measure is patch/test success plus regression notes, with LLM
  judge or human review as secondary evidence.

Remaining questions:

- Do enterprise receipts need uploaded-tarball identity in addition to live
  worktree identity?
- Which Anthropic tokenizer/counting path should be authoritative for strict
  billing before send?
- Which fallback Claude model is approved after a Fable refusal for each
  customer policy?
- How long should receipts and rejected context candidates be retained under
  the Fable 5 30-day retention requirement?

## Current Conclusion

RepoPrompt CE's defensible architecture is not a compressor. It is a structured context state machine with late rendering and reviewable handoff.

Our Active Context Engineering service should copy that architecture shape:

```text
structured context in
  -> evidence-aware selection
  -> safe transformation policy
  -> provider-aware token accounting
  -> deterministic rendering
  -> receipt-backed optimized context out
```

The commercial wedge is not "fewer tokens" alone. It is fewer unsafe tokens, with provenance and receipts.
