---
name: go
description: Use when writing, reviewing, or modifying Go code (.go files), setting up Go projects, or when the user asks about Go idioms, patterns, error handling, concurrency, testing, linting, or project structure.
---

# Go Code Quality

Clear, boring code beats clever code. This is Go's core value and your core directive.

## Core Principles

All code you write MUST meet the following quality criteria:

* following Go conventions: simplicity, readability, minimal abstraction
* no extra code beyond what is absolutely necessary to solve the problem (i.e. no technical debt)
  * If a well-maintained stdlib package or small dependency solves the problem well, use it instead of reimplementing. Prefer the standard library over third-party packages when the stdlib solution is reasonable.
* maximizing algorithmic big-O efficiency for memory and runtime
* minimizing heap allocations; prefer stack allocation and value types where practical
* using concurrency (goroutines, channels, `sync.WaitGroup`, `errgroup`) where appropriate — not everywhere

## Project Setup

* Use Go modules (`go.mod` / `go.sum`). Never use `GOPATH`-mode.
* Target the latest stable Go release unless the project specifies otherwise.
* Run `go mod tidy` before committing.
* Minimize dependencies. Evaluate whether a dependency is worth the cost before adding it.
* Use the standard `cmd/` and `internal/` layout for applications. Do not over-structure — flat is fine for small projects.
* Use `internal/` to prevent other modules from importing your implementation details.
* Avoid `pkg/` for new packages — it is not a Go convention despite its prevalence.
* **MUST** avoid dot imports (`. "pkg"`).

## Preferred Tools and Libraries

* Use `slog` (stdlib `log/slog`) for structured logging. Do not use `fmt.Println` for operational logs.
* Use `errors.New` and `fmt.Errorf` with `%w` for error wrapping. Do not use `github.com/pkg/errors` (deprecated).
* Prefer `net/http` and `http.ServeMux` (Go 1.22+ enhanced routing) for new HTTP server code.

## Code Style and Formatting

* **MUST** run `gofmt` — non-negotiable. All Go code must be canonically formatted.
* **MUST** use meaningful, descriptive variable and function names.
* **MUST** use short variable names (`i`, `n`, `r`, `w`, `ctx`) only in small scopes where the meaning is obvious. Use descriptive names in larger scopes.
* Use `camelCase` for unexported identifiers, `PascalCase` for exported identifiers. `UPPER_CASE` constants are **not** idiomatic Go — use `PascalCase` or `camelCase` for constants.
* **NEVER** use emoji or unicode that emulates emoji (e.g. ✓, ✗) in code or output, except when testing multibyte character handling.
* **MUST** avoid redundant comments that restate what the code obviously does.
* **MUST** avoid comments that leak the original user prompt or meta-context about the file.
* Keep line length reasonable but do not fight `gofmt` over it.

## Modern Go Idioms

Prefer modern language features and stdlib additions over older patterns. AI agents have a known tendency to produce outdated Go due to training data lag. The `go fix ./...` command (rewritten in Go 1.26) can automatically modernize code, but prefer writing modern code from the start.

* Use `min(a, b)` and `max(a, b)` builtins (Go 1.21+) instead of if-else blocks.
* Use `for i := range n` (Go 1.22+) instead of `for i := 0; i < n; i++`.
* Use `slices.Contains(s, v)` instead of manual loops.
* Use `slices` and `maps` packages (Go 1.21+) for common slice/map operations instead of hand-rolled helpers.
* Use `cmp.Or(a, b, c)` (Go 1.22+) instead of chains of `if x == "" { x = fallback }`.
* Use `errors.AsType[T](err)` (Go 1.26+) instead of `var target T; errors.As(err, &target)`.
* Use `new(expr)` (Go 1.26+) to get a pointer to a value instead of `x := val; &x` helper patterns.
* Use `slog.NewMultiHandler` (Go 1.26+) when fanning out to multiple log handlers.
* Use `strings.Cut` instead of `strings.Index` + manual slicing.
* Use `strings.Builder` for string concatenation in loops instead of repeated `+` or `fmt.Sprintf`.
* Use `sync.WaitGroup.Go(fn)` instead of `wg.Add(1)` + `go func() { defer wg.Done(); ... }()`.

When working on an existing codebase, match the Go version declared in `go.mod` — do not use features from a newer Go than what the project targets.

## Documentation

* **MUST** include doc comments on all exported functions, types, methods, and package declarations.
* Doc comments start with the name of the thing being documented: `// Run executes the command.`
* **MUST** document function parameters, return values, and error conditions in prose — Go does not have structured docstring syntax.
* Keep comments up-to-date with code changes.
* Include usage examples via `Example` test functions where helpful.

Example:

```go
// ParseConfig reads the configuration file at path and returns
// the parsed Config. Returns an error if the file cannot be read
// or contains invalid TOML.
func ParseConfig(path string) (*Config, error) {
```

## Error Handling

* **MUST** check every error. **NEVER** use `_` to discard errors unless the function's docs explicitly say it cannot fail.
* **NEVER** use `panic` for expected error conditions. `panic` is for programmer bugs and unrecoverable states only.
* **MUST** wrap errors with context using `fmt.Errorf("doing X: %w", err)`.
* **MUST** use `errors.Is` and `errors.As` (or `errors.AsType[T]` on Go 1.26+) for error inspection — never compare error strings.
* Return errors; don't log-and-return (pick one). Let the caller decide what to do.
* Use sentinel errors (`var ErrNotFound = errors.New("not found")`) for errors callers need to check.
* **NEVER** expose internal error details to end users. Translate errors at API boundaries — log the full error server-side, return an opaque message to the client. Use a custom error type that separates internal details from public-safe messages if needed.

## Function Design

* **MUST** keep functions focused on a single responsibility.
* Accept interfaces, return structs. Returning an interface is acceptable when the caller genuinely should not depend on the concrete type (e.g., returning an `io.Reader`).
* If a function takes more than 5 parameters, group them into an options struct.
* For constructors or APIs with many optional settings and sensible defaults, prefer the functional options pattern (`func WithTimeout(d time.Duration) Option`). This keeps the call site readable (`New(addr, WithTimeout(5*time.Second), WithLogger(log))`), makes zero-value defaults explicit, and allows adding options without breaking existing callers.
* `context.Context` is always the first parameter when present. **NEVER** store `context.Context` in a struct field.
* Prefer returning `(T, error)` over output parameters.
* **MUST** use `defer` for resource cleanup (`file.Close()`, `mu.Unlock()`, rows, response bodies, etc.). A resource acquired in a function should have its cleanup deferred immediately after the error check.
* **MUST** check `io.Reader`/`io.Writer` return values — partial reads/writes are valid behavior, not errors.

## Struct and Interface Design

* **MUST** keep types focused on a single responsibility.
* Keep interfaces small — one or two methods is ideal. Define interfaces where they are used, not where they are implemented.
* Use struct embedding for composition, not for pseudo-inheritance. Do not embed types just to "inherit" methods you don't need.
* Use pointer receivers consistently for a given type. Mix pointer and value receivers only when there's a clear reason.
* Make the zero value useful. If the zero value of your struct is not valid, provide a constructor (`NewFoo`).
* Use struct tags correctly (`json`, `db`, etc.) and keep them consistent.

## Concurrency

* **MUST** document goroutine lifecycle — who starts it, what stops it, and how.
* **MUST** use `context.Context` for cancellation and timeouts in concurrent work.
* **MUST** use `sync.Mutex` or channels — never both for the same shared state. Prefer channels for communication between goroutines, mutexes for protecting shared state.
* **NEVER** launch goroutines without a clear shutdown path. Leaked goroutines are bugs.
* **NEVER** rely on goroutine scheduling order. If your code depends on timing, it's broken.
* Use `errgroup.Group` for concurrent tasks that can fail.
* Use `sync.Once` for one-time initialization.
* Close channels from the sender side only.

## Security

* **NEVER** store secrets, API keys, or passwords in code. Use environment variables or a secret manager.
  * Ensure `.env` files are declared in `.gitignore`.
  * **NEVER** print or log URLs containing API keys.
* **NEVER** log sensitive information (passwords, tokens, PII).
* **MUST** use `crypto/rand` for security-sensitive random values — never `math/rand`.
* **MUST** validate and sanitize all external input.
* **MUST** use parameterized SQL queries — never string interpolation.
* **MUST** run `govulncheck ./...` to scan for known vulnerabilities in dependencies. Integrate it into CI.
* **NEVER** use `init()` unless absolutely necessary (registering drivers, codecs). Prefer explicit initialization so the call site is visible and auditable.
* **NEVER** use global mutable state. Pass dependencies explicitly — globals make it impossible to reason about data flow and are a common source of concurrency bugs and test pollution.

## Testing

**MUST** use table-driven tests for functions with multiple input/output cases. For table-driven test patterns and conventions, use the `go-table-driven-tests` skill.

* **MUST** use red/green TDD: write tests first, confirm they fail (red), then implement until they pass (green).
* Follow the Arrange-Act-Assert pattern.
* **MUST** write tests for all new exported functions and methods.
* **MUST** use the standard `testing` package as the foundation.
* **MUST** run tests with `-race` in CI (`go test -race ./...`). Data races are bugs, not warnings.
* Use `t.Helper()` in test helper functions for proper line reporting.
* Use `t.Parallel()` where tests are independent.
* Use `testdata/` directory for test fixtures (it is ignored by `go build`).
* Use `httptest.NewServer` for HTTP integration tests.
* Use fuzz tests (`func FuzzXxx(f *testing.F)`) for functions that parse or process untrusted input.
* Skip tests that depend on external services or credentials with `t.Skip()` and a clear message, rather than letting them fail. Use a helper like `skipIfNoCreds(t)` for integration tests.
* Consider `github.com/stretchr/testify` (`assert`/`require`) when it would simplify assertions — particularly for deep equality checks, nil checks, and producing readable diffs on failure. If the project already uses testify, prefer it for consistency.
* **NEVER** run tests you generate without first saving them to a `_test.go` file.
* **NEVER** delete test files or test fixtures created during testing.
* Do not commit commented-out tests.

## Benchmarking and Profiling

* Use `testing.B` for benchmarks. Name benchmarks `BenchmarkXxx`.
* **NEVER** run benchmarks in parallel — they will compete for resources and produce invalid results.
* **NEVER** manipulate benchmarks to satisfy performance constraints.
* Use `b.ResetTimer()` after expensive setup.
* Use `b.ReportAllocs()` to track allocations.
* Use `pprof` for CPU and memory profiling before optimizing.
* Prefer `make([]T, 0, n)` when the slice capacity is known ahead of time to reduce allocations.

## Linting and CI

* **MUST** use `golangci-lint` (v2) with at minimum: `govet`, `staticcheck`, `errcheck`, `unused`, `gosimple`.
* **MUST** run `go vet ./...` — it catches real bugs.
* Run `go fix ./...` periodically to adopt modern idioms recommended by the Go team's modernize analyzers.
* Enable `gofumpt` for stricter formatting if the project uses it.
* When adopting linting incrementally, use `--new-from-merge-base=main` in CI to lint only changed lines; use `--new` (`-n`) or `--new-from-rev REV` locally.
* CI should run: `go build ./...`, `go test -race ./...`, `go vet ./...`, `golangci-lint run --new-from-merge-base=main`, `govulncheck ./...`.

## Before Committing

* All tests pass (`go test -race ./...`)
* `go vet ./...` is clean
* `golangci-lint run --new` is clean (at minimum; `--new-from-merge-base=main` in CI)
* `govulncheck ./...` reports no issues
* All exported symbols have doc comments
* No commented-out code or debug prints
* No hardcoded credentials
* `go mod tidy` has been run
