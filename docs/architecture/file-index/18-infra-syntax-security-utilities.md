# Infrastructure — SyntaxParsing · Security · Regex · Networking · Persistence · Concurrency · Utilities — File Index
> Scope: those seven Infrastructure areas. 46 files.

## Subsystem role
These are the cross-cutting infrastructure primitives the rest of the app builds on: tree-sitter syntax highlighting and code-map extraction (SyntaxParsing), API-key/secret storage hardened against the macOS code-signing domain the binary runs under (Security), a PCRE2-backed regex engine and pattern-repair toolkit (Regex), shared URLSession HTTP plumbing (Networking), file-backed preset persistence (Persistence), async-await synchronization helpers (Concurrency), and a grab-bag of String/Array/Sequence/Error/JSON/path utilities (Utilities). Security is the most involved: a runtime detector inspects the process's own code signature, a policy maps it to a storage domain (official Developer ID, local self-signed, Apple-development debug, or ephemeral), and a factory selects the matching Keychain or in-memory backend so secrets never cross signing identities. SyntaxParsing pairs a `SyntaxManager` (grammar/language registry) and `ComprehensiveHighlighter` (token-to-NSColor theming) with per-language `Queries/` files, each holding a tree-sitter highlight query plus a code-map (symbol-extraction) query as static string constants.

## SyntaxParsing/ (+ Queries/)
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/SyntaxManager.swift` — central tree-sitter registry; `LanguageType` enum (14 languages) plus the singleton that loads grammars and dispatches highlight/code-map queries. Key: `SyntaxManager`, `LanguageType`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/ComprehensiveHiglighter.swift` — maps tree-sitter token type names (`keyword`, `string.documentation`, etc.) to dark/light `NSColor` attributes for editor highlighting. Key: `ComprehensiveHighlighter`, `TokenStyle`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/QueryResourceLoader.swift` — loads bundled `.scm` query resources from SwiftPM package bundles (currently only TreeSitterPHP's `highlights.scm`). Key: `QueryResourceLoader`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/SwiftQueries.swift` — Swift tree-sitter highlight query + code-map query string constants. Key: `swiftQuery`, `swiftCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/PythonQueries.swift` — Python highlight + code-map query constants. Key: `pythonQuery`, `pythonCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/typeScript.swift` — TypeScript/TSX highlight + code-map query constants (combines highlights/locals/tags). Key: `typeScriptHighlightQuery`, `typeScriptCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/JavaScriptQueries.swift` — JavaScript highlight + code-map query constants. Key: `javascriptQuery`, `javascriptCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/phpQueries.swift` — PHP highlight, tag, basic-fallback, and code-map query constants. Key: `phpHighlightQuery`, `phpTagQuery`, `basicPhpQuery`, `phpCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/RubyQueries.swift` — minimal compiler-safe Ruby highlight + code-map query constants. Key: `rubyHighlightQuery`, `rubyCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/RustQueries.swift` — Rust highlight + code-map query constants. Key: `rustQuery`, `rustCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/cQueries.swift` — C highlight + code-map query constants. Key: `cQuery`, `cCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/cppQueries.swift` — C++ highlight + code-map query constants. Key: `cppQuery`, `cppCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/cSharpQueries.swift` — C# highlight + code-map query constants. Key: `csharpQuery`, `csharpCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/DartQueries.swift` — Dart highlight + code-map query constants. Key: `dartQuery`, `dartCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/GoQueries.swift` — Go highlight + code-map query constants. Key: `goQuery`, `goCodeMapQuery`.
- `Sources/RepoPrompt/Infrastructure/SyntaxParsing/Queries/JavaQueries.swift` — Java highlight + code-map query constants. Key: `javaQuery`, `javaCodeMapQuery`.

## Security/
- `Sources/RepoPrompt/Infrastructure/Security/KeychainService.swift` — low-level macOS Keychain CRUD over `SecItem*` with interactive vs. non-interactive access modes, named shared services (official-v2, local-self-signed, legacy repair source) and `KeychainError`. Key: `KeychainService`, `KeychainAccessMode`, `SecItemClient`.
- `Sources/RepoPrompt/Infrastructure/Security/KeyManager.swift` — actor caching API keys per `AIProviderType` in memory, lazily backed by `SecureKeysService`; maps each provider to its frozen secure-storage account. Key: `KeyManager`.
- `Sources/RepoPrompt/Infrastructure/Security/SecureKeyService.swift` — provider-agnostic API-key save/get/delete service over the selected `SecureKeyValueStorageBackend`; `SecurePlainStringStoring` protocol for plain-UTF8 values. Key: `SecureKeysService`, `SecurePlainStringStoring`.
- `Sources/RepoPrompt/Infrastructure/Security/SecureKeyValueStorageBackend.swift` — backend protocol (save/get/delete by key) plus `SecureKeyValueStorageFactory` that picks the backend for the runtime signing decision (Developer ID Keychain, local-self-signed Keychain, or ephemeral). Key: `SecureKeyValueStorageBackend`, `SecureKeyValueStorageFactory`.
- `Sources/RepoPrompt/Infrastructure/Security/EphemeralSecureKeyValueStore.swift` — process-local, non-persistent in-memory backend used when runtime signing cannot select a durable domain; lock-guarded `Data` dictionary. Key: `EphemeralSecureKeyValueStore`.
- `Sources/RepoPrompt/Infrastructure/Security/SecureStorageAccountCatalog.swift` — closed `CaseIterable` inventory of every secure-storage account identifier (provider keys, CLI keys, agent-permission documents) with stable string identifiers. Key: `SecureStorageAccount`, `SecureStorageAccountCatalog`.
- `Sources/RepoPrompt/Infrastructure/Security/SecureStorageRepairService.swift` — actor that migrates/imports secrets from a legacy store into the target store, tracking per-account repair state and conflict resolution. Key: `SecureStorageRepairService`, `SecureStorageRepairState`, `SecureStorageRepairRecord`.
- `Sources/RepoPrompt/Infrastructure/Security/LocalSigningIdentityRegistry.swift` — reads/writes the on-disk record (in Application Support) pinning the local self-signed certificate fingerprint + service generation, with strict owner/permission validation. Key: `LocalSigningIdentityRegistry`, `LocalSigningIdentityRecord`.
- `Sources/RepoPrompt/Infrastructure/Security/RuntimeCodeSigningDetector.swift` — inspects the running process's own `SecCode`/`SecStaticCode` signature (validity, leaf cert SHA-256, team id, ad-hoc) to produce `RuntimeCodeSigningInfo`. Key: `RuntimeCodeSigningDetector`, `RuntimeLocalSigningExpectation`.
- `Sources/RepoPrompt/Infrastructure/Security/RuntimeCodeSigningPolicy.swift` — maps signing info + local expectation to a `RuntimeSecureStorageDomain` decision (officialDeveloperID / localSelfSigned / appleDevelopmentDebug / ephemeral). Key: `RuntimeCodeSigningPolicy`, `RuntimeSecureStorageDomain`, `RuntimeSecureStorageDecision`.
- `Sources/RepoPrompt/Infrastructure/Security/SecurityObfuscation.swift` — centralized XOR (key `0x5A`) decode of obfuscated security strings (Sparkle feed URL, public Ed key). Key: `SecurityObfuscation`.

## Regex/
- `Sources/RepoPrompt/Infrastructure/Regex/PCRE2RegexAdapter.swift` — Swift adapter over the PCRE2 engine: env-driven JIT mode and match-limit policy, compile-with-repair, and match-limit profiles for file/path search. Key: `RepoPromptRegexRuntime`, `RepoPromptPCRE2MatchPolicy`, `RepoPromptPCRE2CompileResult`.
- `Sources/RepoPrompt/Infrastructure/Regex/RegexToolkit.swift` — pure-Swift pattern validation/normalization/risk-assessment toolkit (escape whitelist, empty-alternative + unmatched-paren repair). Key: `RegexToolkit`, `RegexPatternFailure`.

## Networking/
- `Sources/RepoPrompt/Infrastructure/Networking/HTTPClient.swift` — `HTTPClient` protocol + `DefaultHTTPClient` over URLSession with preconfigured shared clients (UI-critical, discovery, AI, AI-streaming) and `HTTPResponse`. Key: `HTTPClient`, `DefaultHTTPClient`, `HTTPResponse`.
- `Sources/RepoPrompt/Infrastructure/Networking/HTTPDecoding.swift` — off-main-thread JSON decoding helper via a detached utility task. Key: `HTTPDecoding`.

## Persistence/
- `Sources/RepoPrompt/Infrastructure/Persistence/Presets/PresetFileStore.swift` — file-backed store for workflow/model presets in Application Support, with schema versioning and corrupt/future-document preservation guards. Key: `PresetFileStore`.

## Concurrency/
- `Sources/RepoPrompt/Infrastructure/Concurrency/AsyncMutex.swift` — actor-based async mutex with cancellation-aware waiter queue and a `withLock` helper. Key: `AsyncMutex`.
- `Sources/RepoPrompt/Infrastructure/Concurrency/AsyncScope.swift` — structured async setup/teardown helper that awaits both entry and cleanup phases around an operation. Key: `AsyncScope`.
- `Sources/RepoPrompt/Infrastructure/Concurrency/TaskSemaphore.swift` — async-await counting semaphore (actor) limiting concurrent heavy work, with acquire/release and a `withPermit` helper. Key: `TaskSemaphore`.

## Utilities/ (+ Collections/)
- `Sources/RepoPrompt/Infrastructure/Utilities/StringExtensions.swift` — large String extension grab-bag: model-name truncation, C-backed fuzzy similarity scoring, similarity thresholds, and more. Key: `extension String`.
- `Sources/RepoPrompt/Infrastructure/Utilities/String+Slug.swift` — `slugify` producing URL-friendly slugs (lowercase, separator collapse, max-length truncation, "untitled" fallback). Key: `String.slugify`.
- `Sources/RepoPrompt/Infrastructure/Utilities/ArrayExtensions.swift` — `chunked(into:)` windowing and in-place order-preserving dedup for Hashable arrays. Key: `Array.chunked`, `Array.removeDuplicatesInPlace`.
- `Sources/RepoPrompt/Infrastructure/Utilities/SequenceExtension.swift` — `asyncCompactMap` for awaiting an async transform across a sequence. Key: `Sequence.asyncCompactMap`.
- `Sources/RepoPrompt/Infrastructure/Utilities/ErrorExtensions.swift` — `asFriendlyString()` converting known custom AI/provider error types (OpenAI, Anthropic) into user-facing messages. Key: `Error.asFriendlyString`.
- `Sources/RepoPrompt/Infrastructure/Utilities/JSONDictionaryHelpers.swift` — pretty-print/parse JSON dictionaries and typed accessors (string/bool) tolerant of NSNumber values. Key: `JSONDictionaryHelpers`.
- `Sources/RepoPrompt/Infrastructure/Utilities/RelativePath.swift` — pure string-based relative-path computation (boundary-safe prefix match, no filesystem I/O). Key: `RelativePath`.
- `Sources/RepoPrompt/Infrastructure/Utilities/StandardizedPath.swift` — path standardization helpers (absolute, relative `.`/`..` resolution, join, NUL detection). Key: `StandardizedPath`.
- `Sources/RepoPrompt/Infrastructure/Utilities/CheckoutPathIdentity.swift` — canonical identity for comparing workspace roots / Git worktree paths / checkout bindings (tilde expansion, standardization, Unicode normalization). Key: `CheckoutPathIdentity`.
- `Sources/RepoPrompt/Infrastructure/Utilities/Shortcuts.swift` — `KeyboardShortcuts.Name` definitions for app-wide shortcuts (preset switching, save, compose-tab and Agent-window management). Key: `KeyboardShortcuts.Name`.
- `Sources/RepoPrompt/Infrastructure/Utilities/Collections/BoundedArray.swift` — `appendBounded(_:maxCount:)` appends while evicting oldest elements past a cap. Key: `Array.appendBounded`.

_Indexed 46 files._
