# Infrastructure — AI Substrate (Prompts, Catalog, Services) — File Index
> Scope: `Infrastructure/AI` (excl. Providers/). 63 files.

## Subsystem role
This is the provider-neutral AI substrate that sits beneath the concrete agent/model providers (which live in the excluded `Providers/` folder). It owns the cross-provider plumbing for ACP (Agent Client Protocol) session lifecycle and runtime-event parsing, the coordination/leasing/gating machinery for headless agent runs and their MCP tool tracking, the model catalogs (Codex / Claude Code / ACP-dynamic) with reasoning-effort vocabularies, user model presets and their per-mode support rules, and the core AI message/model/query value types plus streaming services. It also houses the prompt layer: the legacy XML/diff/search-replace edit-prompt factory and per-language code-example fixtures, the role-specific Agent Mode prompts, and the provider-neutral RepoPrompt workflow prompt catalog (`rp-build`, `rp-investigate`, `rp-deep-plan`, `rp-review`, `rp-refactor`, `rp-optimize`, `rp-orchestrate`, `rp-reminder`, `rp-oracle-export`) rendered in MCP / CLI / Agent variants. Everything here is shared scaffolding consumed by the per-vendor provider implementations and the Features layer.

## (root AI files)
- `Sources/RepoPrompt/Infrastructure/AI/AIMessage.swift` — value type assembling a prompt from ordered/disableable sections (system, meta, file tree, file blocks, git diff, conversation) with XML getters; also `ConversationEntry`. Key: `AIMessage`, `ConversationEntry`, `systemPromptXML`, `PromptSection`.
- `Sources/RepoPrompt/Infrastructure/AI/AIModel.swift` — central model enum/registry: every supported model + service-tier/effort variants, model-name parsing, display ordering, capability flags. Large file. Key: `AIModel`, `AIModel.fromModelName`, `ModelPickerStringOrdering`.
- `Sources/RepoPrompt/Infrastructure/AI/AIQueriesService.swift` — streaming chat orchestration: per-stream cancellation, partial-chunk buffering, token/cost accounting, reasoning-text normalization. Key: `ChatStreamID`, `ChatTokenInfo`, `ChatStreamOutput`, `PartialBuffer`, `ReasoningTextFormatter`.
- `Sources/RepoPrompt/Infrastructure/AI/BackgroundResponses.swift` — OpenAI Responses background-job model and provider protocol (create/fetch/stream/cancel), Codable job record with status. Key: `ResponsesJobProvider`, `BackgroundResponseJob`, `BackgroundResponseJob.Status`.
- `Sources/RepoPrompt/Infrastructure/AI/OpenAIURLHelper.swift` — base-URL normalization for OpenAI-compatible endpoints: split base vs `vN` version, add scheme, strip trailing slashes/version. Key: `OpenAIURLHelper.splitBaseURLAndVersion`, `normalizeBaseURL`, `normalizeBaseURLString`.
- `Sources/RepoPrompt/Infrastructure/AI/SystemPromptService.swift` — top-level system-prompt assembly service: Discover/MCP-discover prompts (token budget, agent kind, enhancement mode, clarifying questions), language→extension mapping, agent-mode prompt entry point. Large file. Key: `SystemPromptService.discoverPrompt`, `mcpDiscoverPrompt`, `fileExtension(for:)`, `chatCodeFence`.

## ACP/
- `Sources/RepoPrompt/Infrastructure/AI/ACP/ACPAgentProviderFactory.swift` — maps an `AgentProviderKind` to a concrete ACP provider (OpenCode, Cursor); returns nil for non-ACP kinds (Claude Code, Codex exec). Key: `ACPAgentProviderFactory.makeProvider`.
- `Sources/RepoPrompt/Infrastructure/AI/ACP/ACPAgentSessionController.swift` — actor driving the full ACP session lifecycle over a subprocess: launch→initialize→open/load session→prompt→close, with state machine, request timeouts, diagnostics sink, and bootstrap result. Large file. Key: `ACPAgentSessionController`, `State`, `BootstrapResult`, `DiagnosticEvent`, `RequestTimeouts`.
- `Sources/RepoPrompt/Infrastructure/AI/ACP/ACPPromptContentBuilder.swift` — builds ACP prompt content blocks (text + image attachments) from local files or URLs, with MIME detection and base64 encoding. Key: `ACPPromptContentBuilder.blocks`, `imageBlock`, `Error.unreadableLocalImage`.
- `Sources/RepoPrompt/Infrastructure/AI/ACP/ACPProviderSupport.swift` — ACP runtime-event parsing helpers: recursively extract content text, normalize/identify tool names (incl. RepoPrompt MCP tool mapping), machine-identifier detection. Key: `ACPRuntimeEventParsing.extractContentText`, `normalizedToolName`.

## Agents/
- `Sources/RepoPrompt/Infrastructure/AI/Agents/AgentRunCoordinator.swift` — singleton coordinator for headless agent runs: registers/cleans MCP routing, installs per-run client policy, acquires the global connection gate, maps run type to MCP run purpose. Key: `AgentRunCoordinator.shared`, `AgentRunSpec`, `AgentRunType`, `registerRouting`.
- `Sources/RepoPrompt/Infrastructure/AI/Agents/AgentToolTracker.swift` — actor observing MCP tool usage for one agent session keyed by runID (survives connection handovers); basic and enhanced (args+result) observers, connection-wait timeouts. Key: `AgentToolTracker.start`, `registerEnhancedObserver`.
- `Sources/RepoPrompt/Infrastructure/AI/Agents/HeadlessAgentConnectionGate.swift` — actor serializing headless agent connections so only one installs its policy/spawns at a time; cancellation-safe waiter queue and re-checked acquire. Key: `HeadlessAgentConnectionGate.shared`, `waitForClearConnection`, `acquire`.
- `Sources/RepoPrompt/Infrastructure/AI/Agents/HeadlessAgentRunLease.swift` — convenience factory extending `MCPBootstrapLeaseSpec` to build a one-shot headless lease (runID, gateID, client name, window, restricted/additional tools, TTL, purpose). Key: `MCPBootstrapLeaseSpec.headless`.

## ModelCatalog/
- `Sources/RepoPrompt/Infrastructure/AI/ModelCatalog/ReasoningEffort.swift` — Codex reasoning-effort vocabulary (none/minimal/low/medium/high/xhigh) with parse + display order + display names. Key: `CodexReasoningEffort`, `parse`, `displayOrder`.
- `Sources/RepoPrompt/Infrastructure/AI/ModelCatalog/Providers/ACPAIModelCatalog.swift` — ACP dynamic model catalog: Codable provider/model records discovered per session, persisted to `UserDefaults` and reloaded. Key: `ACPDynamicModelStore`, `ACPDynamicModelRecord`, `ACPDynamicProviderRecord`, `ACPDiscoveredSessionModels`.
- `Sources/RepoPrompt/Infrastructure/AI/ModelCatalog/Providers/ClaudeCodeAIModelCatalog.swift` — Claude Code model catalog: static model definitions (Opus/Sonnet/Haiku variants + 1M) with supported effort levels, raw-specifier prefix, compatible-backend descriptors, runtime-specifier mapping from `AIModel`. Key: `ClaudeCodeAIModelCatalog`, `CompatibleBackendModelDescriptor`, `runtimeSpecifierRaw`.
- `Sources/RepoPrompt/Infrastructure/AI/ModelCatalog/Providers/CodexAIModelCatalog.swift` — Codex model catalog: Codable dynamic model/reasoning records (id, model, display, default, supported efforts) with custom decoding. Key: `CodexDynamicModelRecord`, `CodexDynamicReasoningRecord`.

## Models/
- `Sources/RepoPrompt/Infrastructure/AI/Models/ModelPreset.swift` — user-defined model preset value type: id/name/modelString/description, supported modes, legacy pro-editing override, chat-preset mappings; resolves to `AIModel` with fallback and tolerant decoding. Key: `ModelPreset`, `ProEditingOverride`, `model`, `optionalModel`, `isModelResolvable`.
- `Sources/RepoPrompt/Infrastructure/AI/Models/ModelPreset+ModeSupport.swift` — zero-cost helpers deciding whether a preset/`SupportedModes` allows a mode string (chat/plan/review, unknown→true), plus array filtering by mode. Key: `SupportedModes.supports(mode:)`, `ModelPreset.supports(mode:)`, `[ModelPreset].filteredForMode`.

## Prompts/Code Examples/
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/CodeExamples.swift` — `CodeExamples` protocol: the generic per-language snippet interface (search/replace old/new lines, rewrite/create, network-manager examples, negative examples) consumed by the prompt factory. Key: `CodeExamples` protocol.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/SwiftExamples.swift` — `CodeExamples` impl holding Swift search/replace, rewrite/create, and negative-example snippet constants. Key: `SwiftExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/CExamples.swift` — `CodeExamples` impl with C-language edit-prompt example snippets. Key: `CExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/CppExamples.swift` — `CodeExamples` impl with C++ edit-prompt example snippets. Key: `CppExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/CSharpExamples.swift` — `CodeExamples` impl with C# edit-prompt example snippets. Key: `CSharpExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/DartExamples.swift` — `CodeExamples` impl with Dart edit-prompt example snippets. Key: `DartExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/GoExamples.swift` — `CodeExamples` impl with Go edit-prompt example snippets. Key: `GoExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/JavaExamples.swift` — `CodeExamples` impl with Java edit-prompt example snippets. Key: `JavaExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/JavaScriptExamples.swift` — `CodeExamples` impl with JavaScript edit-prompt example snippets. Key: `JavaScriptExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/PHPExamples.swift` — `CodeExamples` impl with PHP edit-prompt example snippets. Key: `PHPExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/PythonExamples.swift` — `CodeExamples` impl with Python edit-prompt example snippets. Key: `PythonExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/RustExamples.swift` — `CodeExamples` impl with Rust edit-prompt example snippets. Key: `RustExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/TSXExamples.swift` — `CodeExamples` impl with TSX (TypeScript+JSX) edit-prompt example snippets. Key: `TSXExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Code Examples/TypeScriptExamples.swift` — `CodeExamples` impl with TypeScript edit-prompt example snippets. Key: `TypeScriptExamples`.

## Prompts/Legacy/
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/ChatPrompt.swift` — legacy chat system prompt string: XML `<file>`/`<change>` code-modification format with plan-first + escaping rules. Key: `chatPrompt`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/ChatPromptFull.swift` — legacy chat prompt variant restricted to `rewrite`/`create` actions (whole-file format). Key: `chatPromptFull`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/ChatSearchReplace.swift` — legacy chat prompt teaching the `===`-delimited search/replace edit format (indented). Key: `chatSearchReplacePrompt`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/ChatSearchReplaceNoIndent.swift` — search/replace chat prompt variant without indentation encoding. Key: `chatSearchReplacePromptNoIndent`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/DiffExamples.swift` — large set of JSON-diff worked examples (correct/incorrect) with `<s#>` indentation encoding for the diff-prompt format. Key: `diffExamples`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/DiffQuery.swift` — reusable diff-prompt fragments: assistant preamble, analysis task, per-file diff structure, and overall-summary JSON delimiter block. Key: `assistantPreamble`, `analysisTask`, `changeSummary`, `diffStructurePrompt`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/DirectDiffPrompt.swift` — prompt for a diff-generator that emits an accurate `###JSON_START###`-delimited diff from a proposed change + original context. Key: `directDiffPrompt`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/FileEditDiffIndent.swift` — file-edit prompt that integrates placeholdered snippets into a file via XML modify instructions, with indentation encoding. Key: `fileEditDiffPromptIndent`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/FileEditDiffPrompt.swift` — file-edit prompt (modify/rewrite) integrating placeholdered snippets, no indentation encoding. Key: `fileEditDiffPrompt`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/FileEditPrompt.swift` — file-edit prompt that outputs a full rewrite with placeholders/comments removed so the result compiles. Key: `fileEditPrompt`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/WholeFormatClipboard.swift` — whole-file clipboard chat prompt (`rewrite`/`create` actions) for clipboard-based edit flows. Key: `wholeClipboard`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Legacy/xmlPrompt.swift` — base XML edit-format prompt: `<file>`/`<change>` with start/end selectors and `<t#>`/`<s#>` indentation encoding. Key: `xmlPrompt`.

## Prompts/Workflows/
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/RepoPromptWorkflowID.swift` — enum of workflow ids (build/investigate/deepPlan/reminder/oracleExport/review/refactor/orchestrate/optimize) with `rp-*` command names and MCP/install ordering. Key: `RepoPromptWorkflowID`, `commandName`, `mcpPromptOrder`, `installOrder`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/RepoPromptWorkflowPrompts.swift` — root enum/namespace for provider-neutral workflow prompt renderers; carries the versioned changelog (skill content version) that drives managed-skill reinstalls. Key: `RepoPromptWorkflowPrompts`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPromptCatalog.swift` — declarative descriptor catalog (id, description, arguments) for each workflow, indexed by id, used to advertise MCP prompts. Key: `WorkflowPromptCatalog.descriptors`, `WorkflowPromptDescriptor`, `WorkflowPromptArgument`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPromptVariant.swift` — variant enum (mcp/cli/agent) selecting tool-invocation syntax; supplies the rpce-cli preamble + MCP↔CLI command mapping table. Key: `WorkflowPromptVariant`, `preamble`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPromptSharedFragments.swift` — shared rendering fragments: variant-aware example selector and the Phase-0 workspace-verification block (skipped for agent variant). Key: `RepoPromptWorkflowPrompts.example`, `workspaceVerificationBlock`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Convenience.swift` — convenience accessors exposing CLI and Agent variants of each workflow (rpBuildCLI, rpReviewAgent, etc.). Key: `rpBuildCLI`, `rpBuildAgent`, `rpRefactorAgent`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Build.swift` — `rp-build` workflow: quick-scan → context_builder plan → optional oracle → implement directly; variant-aware tool naming. Key: `rpBuildCore`, `rpBuild`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+DeepPlan.swift` — `rp-deep-plan` workflow: delegation-heavy planning that ends at `docs/plans/<topic>-<date>.md` with no implementation. Key: `rpDeepPlan`, `rpDeepPlanCore`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Investigate.swift` — `rp-investigate` read-only workflow: orchestrate explore + pair investigators + context_builder + oracle, write findings to an investigation report. Key: `rpInvestigateCore`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Optimize.swift` — `rp-optimize` iterative perf loop: instrument debug metrics, baseline, then plan→delegate→re-measure→ask oracle until target met. Key: `rpOptimize`, `rpOptimizeCore`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+OracleExport.swift` — `rp-oracle-export` workflow: clarify Question/Plan/Review intent and export a prompt file for GPT Pro on ChatGPT, with budget/fast-path rules per variant. Key: `rpOracleExport`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Orchestrate.swift` — `rp-orchestrate` workflow: plan, decompose, and dispatch work across multiple agents (fresh-agent-per-item default, roles-only escape hatch). Key: `rpOrchestrate`, `rpOrchestrateCore`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Refactor.swift` — `rp-refactor` workflow: analyze structure, reduce duplication/complexity, suggest concrete improvements without changing logic. Key: `rpRefactor`, `rpRefactorCore`, `rpRefactorCLI`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Reminder.swift` — `rp-reminder` workflow: terse table reminding the agent to use RepoPrompt MCP/CLI tools instead of built-in grep/cat/Edit equivalents. Key: `rpReminder`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/Workflows/WorkflowPrompt+Review.swift` — `rp-review` code-review workflow: scope changes via git, gather context via context_builder, produce actionable review feedback. Key: `rpReview`, `rpReviewCore`, `rpReviewCLI`.

## Prompts/ (root)
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/AgentModePrompts.swift` — role-specific Agent Mode prompts (explore/engineer and standard), tied to `ListTools` advertisement; defines export-delegation audience classification. Key: `AgentModePrompts`, `ExportDelegationAudience`.
- `Sources/RepoPrompt/Infrastructure/AI/Prompts/PromptFactory.swift` — the edit-prompt factory: `PromptConfig` (role + capabilities + language/format options) drives assembly of legacy XML/diff/search-replace prompts with the right code examples. Key: `PromptConfig`, `PromptConfig.Role`.

_Indexed 63 files._
