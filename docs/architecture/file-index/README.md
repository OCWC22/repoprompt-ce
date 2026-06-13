# RepoPrompt CE — File Index (master TOC)

A complete, file-by-file index of the RepoPrompt CE codebase, organized by
subsystem. Read [`00-overview.md`](00-overview.md) first for the architecture
walkthrough and how the pieces connect; then drill into the per-area files below.

- **Coverage:** ~1,127 first-party Swift/C source & test files indexed
  individually + 96 build/CI/script/skill/doc entries + the vendored
  `CSwiftPCRE2` C library (summarized).
- **Generated:** 2026-06-08 by reading every file through the RepoPrompt MCP.
- **Companion docs:** [`../source-layout.md`](../source-layout.md) (ownership/placement),
  [`../provider-plugins.md`](../provider-plugins.md) (Agent Mode provider seam),
  [`../../open-source-readiness.md`](../../open-source-readiness.md),
  [`../../releasing.md`](../../releasing.md), [`../../worktrees.md`](../../worktrees.md).

## Table of contents

| # | File | Scope | Files |
| --- | --- | --- | --- |
| 00 | [Overview](00-overview.md) | Architecture walkthrough + how subsystems connect | — |
| 01 | [App](01-app.md) | `Sources/RepoPrompt/App` — lifecycle, composition root (`WindowState`), commands, notifications, Sparkle, root views | 36 |
| 02 | [Agent Mode — Runtime](02-agentmode-runtime.md) | `Features/AgentMode/{Models,Providers,Recommendations,Services,Runtime}` — run paths, provider adapters, sessions, transcript policy, permissions | 75 |
| 03 | [Agent Mode — View Models](03-agentmode-viewmodels.md) | `Features/AgentMode/ViewModels` — `AgentModeViewModel` + extensions, UI stores, onboarding/recommendation VMs | 39 |
| 04 | [Agent Mode — Tool Cards & Transcript Views](04-agentmode-views-toolcards.md) | `Features/AgentMode/Views/{ToolCards,Transcript}` — `ToolCardRouter`, per-tool result cards, unified diff, scroll engine | 37 |
| 05 | [Agent Mode — Screen & Chrome Views](05-agentmode-views-chrome.md) | `Features/AgentMode/Views` (excl. ToolCards/Transcript) — main view, input bar, sidebar, pills, runtime sidebar, popovers | 40 |
| 06 | [Chat · Context Builder · Prompt · Search](06-features-chat-contextbuilder-prompt-search.md) | `Features/{Chat,ContextBuilder,Prompt,Search}` — oracle/chat, agentic context curation, prompt assembly, search adapters | 53 |
| 07 | [CodeMap · Workspace Files · Workspaces](07-features-codemap-workspacefiles-workspaces.md) | `Features/{CodeMap,WorkspaceFiles,Workspaces}` — signature extraction, file/git view models, workspace switching | 46 |
| 08 | [Settings](08-features-settings.md) | `Features/Settings` — JSON-only settings stores + all settings panes | 46 |
| 09 | [Diagnostics](09-features-diagnostics.md) | `Features/Diagnostics` — DEBUG instrumentation + Settings-visible Repo Bench | 45 |
| 10 | [Infrastructure — AI Providers](10-infra-ai-providers.md) | `Infrastructure/AI/Providers` — cloud chat + CLI/agent providers, `AIProviderFactory` | 65 |
| 11 | [Infrastructure — AI Substrate](11-infra-ai-prompts-catalog.md) | `Infrastructure/AI` (excl. Providers) — ACP, agents, model catalog, prompts, workflow catalog, AI services | 63 |
| 12 | [Infrastructure — MCP Subsystems](12-infra-mcp-services.md) | `Infrastructure/MCP/{Agent,ApplyEdits,AppShared,Policies,ViewModels,WindowTools,WorkspaceApproval}` | 65 |
| 13 | [Infrastructure — MCP Core Runtime](13-infra-mcp-core.md) | `Infrastructure/MCP` (flat root) — Service/Tool model, server controller, transports, registries, routing | 43 |
| 14 | [Infrastructure — WorkspaceContext & VCS](14-infra-workspacecontext-vcs.md) | `Infrastructure/{WorkspaceContext,VCS}` — context store, indexing, search/slices/tokens, git/jj, worktrees, diff | 69 |
| 15 | [Infrastructure — FileSystem · Diffing · Process](15-infra-filesystem-diffing-process.md) | `Infrastructure/{FileSystem,Diffing,Process}` — FS service + ignore rules, diff engine, CLI/process launch | 58 |
| 16 | [Infrastructure — UI Components](16-infra-ui-components.md) | `Infrastructure/UI/{Components,Agent,Composer,Tooltip}` — reusable SwiftUI/AppKit substrate | 46 |
| 17 | [Infrastructure — UI Text, Markdown & Mentions](17-infra-ui-text-markdown.md) | `Infrastructure/UI/{Markdown,Mentions,Services,TextField}` — TextKit @-mention input, markdown rendering | 27 |
| 18 | [Infrastructure — Syntax/Security/Utilities](18-infra-syntax-security-utilities.md) | `Infrastructure/{SyntaxParsing,Security,Regex,Networking,Persistence,Concurrency,Utilities}` | 46 |
| 19 | [CLI · Shared · Provider Package · C Targets](19-cli-shared-packages-ctargets.md) | `RepoPromptMCP`, `RepoPromptShared`, `Packages/RepoPromptAgentProviders`, `RepoPromptC`, `TreeSitterScannerSupport`, `Support`, `ThirdParty`, `CSwiftPCRE2` | 56 (+vendored) |
| 20 | [Tests (part 1)](20-tests-part1.md) | `Tests/RepoPromptTests/{AgentMode,AI,App,Chat,CodeMap,ContextBuilder,Diagnostics,Diffing,Helpers,Mentions,Prompt,Security}` | 85 |
| 21 | [Tests (part 2)](21-tests-part2.md) | `Tests/RepoPromptTests/{MCP,Services,WorkspaceContext,Workspaces}` + root tests | 87 |
| 22 | [Build · Scripts · CI · Skills · Docs](22-docs-build-scripts.md) | `Package.swift`, `Scripts/`, `.github/workflows`, `.agents/skills`, `docs/`, app bundle resources | 96 |

## How to use this index

- **Onboarding / "where does X live?"** — skim [`00-overview.md`](00-overview.md),
  then open the area file and search for the symbol or path.
- **Each entry** is `` `path/relative/to/repo/root` — one-line responsibility. Key: `Types`, `funcs` ``.
- **Authoritative placement rules** remain in [`../source-layout.md`](../source-layout.md);
  this index describes what exists, that doc governs where new code goes.

## Regenerating

This index was produced by fanning out subagents that each enumerated a scope via
the RepoPrompt MCP `get_file_tree` (authoritative file set), read economically with
`get_code_structure` / `read_file`, and wrote one `NN-*.md` file. It is a generated
snapshot: re-run the same partition to refresh after large refactors. It is an
informational artifact and is not part of the build or guardrails.
