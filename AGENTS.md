# AGENTS GUIDE — adk-go

1. PURPOSE: Playbook for autonomous/codegen agents working in this repo. Keep changes small, typed, tested.
2. MODULE: `google.golang.org/adk`
3. GO VERSION: go 1.24.4 (from go.mod)
4. LICENSE: Apache 2.0. Every source file must carry the Apache 2.0 header (enforced by goheader), except `internal/httprr/*`.
5. REPO STYLE SOURCE: Google Go Style Guide + `.golangci.yml` + GitHub Actions workflows + CONTRIBUTING.md.
6. CURSOR RULES: None found (no `.cursor/rules/*`, no `.cursorrules`).
7. COPILOT RULES: None found (no `.github/copilot-instructions.md`).
8. PRIMARY TOOLING: gofmt/goimports/gofumpt, staticcheck, goheader, golangci-lint.
9. LOCAL IMPORT PREFIX: `google.golang.org/adk` (goimports configured).
10. GOFUMPT: extra-rules enabled, module-path set.
11. STATICCHECK: defaults with selected suppressions (-ST1000,-ST1003,-ST1016,-ST1020,-ST1021,-ST1022; also -QF1001,-QF1008).
12. EXCLUSIONS: `internal/httprr/*` excluded from goheader/errcheck/staticcheck.
13. QUICK HEALTH CHECK (local): `go mod tidy -diff && go build -mod=readonly -v ./... && go test -mod=readonly -v ./... && golangci-lint run`.
14. BUILD (all pkgs): `go build -mod=readonly -v ./...`.
15. BUILD (no module edits): keep `-mod=readonly` to avoid writing go.sum in CI-like runs.
16. RUN EXAMPLE (quickstart help): `go run ./examples/quickstart/main.go help`.
17. RUN EXAMPLE (console): `go run ./examples/quickstart/main.go console`.
18. RUN EXAMPLE (web): `go run ./examples/quickstart/main.go web`.
19. TEST (all): `go test -mod=readonly -v ./...`.
20. TEST (single package): `go test -v ./path/to/package`.
21. TEST (single file): `go test -v ./path/to/package -run TestName` (Go matches substring; escape with `^TestName$` for exact).
22. TEST (single test + table case): `go test -v ./path/to/package -run 'TestName/Case'`.
23. TEST (race, CI nightly style): `go test -race -mod=readonly -v -count=1 -shuffle=on ./...`.
24. TEST (coverage quick): `go test -cover ./...`.
25. TEST (rerun deterministically): add `-count=1` when needed.
26. TEST DATA RECORDING (HTTP): use `-httprecord=<regexp>`; see `internal/httprr`. Many tests include `//go:generate go test -httprecord=.*`.
27. RE-RECORD HTTP TRACES: e.g., `go test -httprecord=Test ./tool/functiontool` or `go test -httprecord=testdata/.*\.httprr ./model/gemini`.
28. LINT (project config): `golangci-lint run`.
29. LINT (list linters): `golangci-lint linters`.
30. FORMAT: gofmt/goimports/gofumpt are run by golangci-lint. You can also run `gofumpt -l -w .` and `goimports -w .`.
31. DEPENDENCY SYNC: `go mod tidy -diff` (CI step). Avoid committing tidy noise unrelated to your change.
32. CI WORKFLOW (go.yml): tidy → build (-mod=readonly) → test (-mod=readonly) and separate golangci-lint job.
33. CI WORKFLOW (nightly.yml): test with `-race -mod=readonly -v -count=1 -shuffle=on ./...`; separate govulncheck job.
34. LOCAL ENV: See `.github/actions/setup/action.yml` for caching; uses go-version from go.mod.
35. ISSUE TEMPLATES: live under `.github/ISSUE_TEMPLATE/` for bug/feature context.
36. GENERAL CONTRIBUTION RULES: Google CLA, Google Open Source conduct, PRs require review, keep PRs small, include testing plan (from CONTRIBUTING.md).
37. ALIGNMENT: adk-python is source-of-truth reference; keep behavior aligned where applicable.
38. DOCUMENTATION CHANGES: user-facing docs live in `adk-docs` repo; open parallel PR there if you change behavior/docs.
39. FILE HEADERS: Ensure new Go files include the Apache 2.0 header matching `.golangci.yml` template (YEAR as 20xx, COMPANY Google LLC).
40. FORMATTING CONVENTIONS: rely on gofmt/goimports/gofumpt; do not hand-tweak spacing. No custom formatters.
41. IMPORT ORDER: standard library first, then third-party, then local `google.golang.org/adk/...`; goimports handles grouping.
42. UNUSED IMPORTS: prohibited; goimports/golangci-lint will fail.
43. NAMING (Go style): exported identifiers in CamelCase; receivers typically short (e.g., `c`, `s`, `h`). Avoid stutter with package names.
44. TYPES: Prefer concrete types; use interfaces for behavior-based abstractions near consumers. Avoid `interface{}`; use generics or concrete types.
45. ERRORS: return errors, do not panic for flow control. Wrap with `%w` when adding context. Compare errors via `errors.Is/As`.
46. ERROR MESSAGES: lower-case, no trailing punctuation. Include actionable context (inputs, state) without leaking secrets.
47. LOGGING: prefer structured/context-aware logging used in the touched package; avoid `fmt.Printf`. Keep logs concise and privacy-safe.
48. CONTEXT: accept `context.Context` as first arg for ops that may block or I/O. Honor cancellation; avoid storing contexts in structs.
49. CONCURRENCY: use channels/`sync` carefully; prefer `errgroup` when aggregating goroutines. Protect shared state; avoid data races (nightly uses `-race`).
50. TIME: prefer injected clocks or time sources when testability matters.
51. RANDOMNESS: avoid global rand without seeding; inject sources for determinism.
52. IO: close resources with `defer` near acquisition; check close errors when relevant.
53. PANICS: only for programmer errors; tests may assert panic when intended.
54. CONFIGURATION: keep flags/config near `cmd/` or package boundaries; follow existing patterns in neighboring files.
55. PUBLIC SURFACE: maintain backward compatibility; avoid breaking exported APIs without discussion/issue.
56. TEST STYLE: table-driven tests preferred; name cases clearly. Use `t.Helper()` in helpers.
57. TEST ISOLATION: avoid network/filesystem unless recorded fixtures (httprr) or temp dirs. Use `t.TempDir()`.
58. TEST ASSERTIONS: standard library `testing`; avoid extra deps unless already in package. Use `cmp` (`github.com/google/go-cmp`) when present.
59. TEST FLAKES: use `-shuffle=on` (nightly) to detect order dependence; fix ordering assumptions.
60. TEST COVERAGE: keep unit tests fast; coverage commands optional but encouraged locally.
61. TEST DATA: store fixtures under package `testdata/`; keep small and readable.
62. EXAMPLES DIRECTORY: minimal feature demos; use `full.NewLauncher()` for multi-mode runs; `prod.NewLauncher()` for leaner builds.
63. RUNNER/LAUNCHER: see `examples/README.md` for launcher options; prefer full launcher for dev, prod launcher for production configs.
64. HTTP RECORD/REPLAY: `internal/httprr` defines `-httprecord` flag. Set regexp to choose which tests/files to re-record.
65. HTTPrecord flag parsing errors: running tests with invalid regexp will fail early; validate regex before use.
66. LINT ENFORCEMENTS: goheader (Apache header), gofumpt (format), goimports (imports), staticcheck (static analysis). Fix lint before PR.
67. SKIP LINT EXCEPTIONS: none; do not add to exclusions without maintainer approval.
68. DEP VERSIONS: avoid manual go.mod edits; use `go get` then `go mod tidy -diff`. Keep changes minimal to the feature.
69. VENDOR: not used; keep modules tidy.
70. CODE ORGANIZATION: prefer small packages with clear responsibilities; keep `internal/` for non-exported helpers.
71. API DESIGN: keep exported APIs consistent with Python ADK semantics when applicable.
72. ERR STRINGS VS LOGS: errors for programmatic handling; logs for operators. Do not log and return same error unless value is different audiences.
73. SECURITY: avoid embedding secrets in code/tests; scrub logs.
74. PRIVACY: redact PII; prefer IDs over raw values in logs/errors.
75. INPUT VALIDATION: validate public-facing inputs early; return clear errors.
76. CLI FLAGS: use cobra where present; keep flag names kebab-case; provide helpful `--help` text.
77. DOCUMENTATION COMMENTS: add Go doc comments to exported identifiers; keep concise and descriptive.
78. LICENSE EXCEPTION: `internal/httprr` has its own LICENSE (see directory).
79. GIT HYGIENE: no commits from agents unless requested. Keep diffs focused.
80. PR TEMPLATE: include testing plan, screenshots/logs for UI/runner as requested in CONTRIBUTING.md.
81. REVIEW READY: run lint/tests before PR; attach command outputs if relevant.
82. THIRD-PARTY CODE: note licenses; keep headers if required.
83. MODEL ALIGNMENT: mirror behavior of Python ADK where possible; check `adk-python` for precedence.
84. DEPLOYMENT: examples runnable via `go run`; production launcher uses `prod.NewLauncher()` (restapi+a2a only).
85. PERFORMANCE: prefer streaming/iterators over full buffering; watch allocations in hot paths.
86. MEMORY: free large buffers; avoid global caches without eviction.
87. METRICS/TRACING: follow existing package patterns if touching telemetry; prefer OpenTelemetry helpers already imported.
88. ERROR WRAPPING VS STRINGS: use `%w` with `fmt.Errorf` instead of concatenation for causal chains.
89. RETRIES: avoid silent retries; back off and cap attempts; surface context cancellation.
90. FILE PATHS: use `t.TempDir()` in tests; avoid relative paths assumptions.
91. ENV VARS: document expected vars in tests/examples; avoid required globals without defaults.
92. MOCKS/STUBS: prefer interfaces near usage; keep fakes simple.
93. SERIALIZATION: use existing marshal helpers; maintain compatibility across languages where protocol shared.
94. CODEGEN: if updating generated files, document generator version/commands in comments; keep reproducible.
95. PANIC HANDLING IN SERVERS: recover only at boundaries; log and fail fast where appropriate.
96. HTTP HANDLERS: validate inputs, set timeouts, close bodies, set headers explicitly.
97. GRPC: follow existing interceptors; propagate context metadata; close connections.
98. DATABASE: use `gorm.io/gorm` present in go.mod; follow package patterns for migrations (check surrounding code before changes).
99. FEATURE FLAGS: keep defaults safe; document in README/PR.
100. RELEASES: no automation notes; follow maintainer guidance if tagging.
101. NIGHTLY ACTIONS: uses `go test -race -count=1 -shuffle=on`; fix any flakes before merging.
102. VULN SCANS: nightly `govulncheck` runs; fix or suppress with justification if triggered.
103. CACHING (CI): Go build/cache keyed on go.mod/go.sum hashes; avoid unnecessary module churn.
104. TEMPORARY DEBUG: do not leave debug prints; use logging with guard or remove.
105. LARGE CHANGES: split into sequential PRs; start with scaffolding and tests.
106. BINARY ARTIFACTS: do not commit; use `testdata` for fixtures only.
107. CODE OWNERSHIP: all changes need review; respect reviewer guidance.
108. FAIL FAST: prefer returning errors early; avoid deep nesting where possible.
109. SWITCH/TYPE ASSERTS: handle default cases; avoid panics on unexpected types.
110. NIL HANDLING: check for nil pointers/maps/slices; prefer zero-value initialization when cheap.
111. SLICE/MAP MUTATION: avoid sharing mutable slices/maps across goroutines without sync.
112. JSON/YAML: maintain field tags and omitempty semantics; keep compatibility.
113. CLI UX: include `--help` examples; keep error messages actionable.
114. EXTERNAL CALLS: wrap with context/timeouts; capture latency metrics when relevant.
115. FILE WATCH: none noted; avoid adding without discussion.
116. PORTS/NETWORK: avoid hard-coded ports in tests; use random/free ports.
117. REGEN INSTRUCTIONS: annotate `//go:generate` lines when adding tooling; ensure commands are runnable.
118. TEST PARALLELISM: use `t.Parallel()` when safe to cut test time; ensure no shared mutation.
119. BENCHMARKS: use `go test -bench .` when adding; keep deterministic.
120. API BREAKAGE: consult maintainers before changes to exported packages; update docs/tests accordingly.
121. DOC LINKS: prefer https://google.github.io/adk-docs/ for public docs.
122. STYLE SOURCE REMINDER: when in doubt, mirror nearby files and Google Go Style.
123. TOOL VERSIONS: golangci-lint action pinned to v2.3.1; go setup via go.mod.
124. LOCAL PATHS: keep code under module root; avoid relative imports.
125. SUBMODULES: none; do not add without approval.
126. WORKSPACES: not used; single module.
127. GENERATED DATA LICENSE: ensure compliance when adding fixtures.
128. PRIVILEGED OPS: avoid; if necessary, document and guard.
129. BUILD TAGS: if introducing, document default behavior and CI coverage.
130. EXCEPTIONS TO RULES: document in PR description and code comments.
131. TRIAGE: new issues should use templates; link to contribution guide.
132. REVIEW NOTES: address reviewer comments promptly; keep follow-up changes scoped.
133. COMPATIBILITY: stay aligned with supported Go version (1.24.x); avoid using features requiring newer toolchain.
134. THIRD-PARTY UPGRADES: include changelog notes in PR when bumping major deps.
135. MODULE CACHE: clean with `go clean -modcache` if CI cache causes issues (rare).
136. LOCAL LINT SPEEDUP: `golangci-lint run --fast` acceptable pre-flight; full run before PR.
137. MEMORY RACES: rely on nightly `-race`; consider `-race` locally for touched packages.
138. SHUFFLE FAILURES: reproduce with `go test -shuffle=on -run ...`; fix ordering issues.
139. ERROR CODES: prefer sentinel errors or typed errors for control flow; avoid string matching.
140. CONFIG SECRETS: use env or secret management; never hard-code.
141. API KEYS IN TESTS: do not embed; use recorded fixtures instead.
142. MULTI-LANG ALIGNMENT: when behavior differs from Python/Java ADK, call it out in PR.
143. CLEANUP: remove unused files/experiments; keep repo lean.
144. STATEFUL SINGLETONS: avoid unless necessary; guard with sync.Once and document.
145. BACKGROUND TASKS: ensure shutdown hooks; propagate context.
146. REENTRANCY: design handlers safe for concurrent calls.
147. HOT PATHS: benchmark before/after for non-trivial changes.
148. PUBLIC DOCS LINK: badge/docs in README; keep URLs current.
149. FINAL CHECK BEFORE PR: tidy diff clean, lint clean, tests clean, headers present.
150. AGENT REMINDER: do not commit unless explicitly asked; keep changes scoped and well-tested.
