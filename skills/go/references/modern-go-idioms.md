# Modern Go Idioms

Prefer modern language features and stdlib additions over older patterns. AI agents have a known tendency to produce outdated Go due to training data lag. The `modernize` analyzer (in `gopls` and `golangci-lint`) and `go fix ./...` (for registered API migration fixes) can automatically modernize code, but prefer writing modern code from the start.

| Pattern | Modern (preferred) | Old (avoid) |
|---|---|---|
| Min/max | `min(a, b)` / `max(a, b)` (Go 1.21+) | if-else blocks |
| Loop over N | `for i := range n` (Go 1.22+) | `for i := 0; i < n; i++` |
| Slice contains | `slices.Contains(s, v)` | manual loops |
| Slice/map ops | `slices` and `maps` packages (Go 1.21+) | hand-rolled helpers |
| Fallback chain | `cmp.Or(a, b, c)` (Go 1.22+) | chains of `if x == "" { x = fallback }` |
| Error type assert | `errors.AsType[T](err)` (Go 1.26+) | `var target T; errors.As(err, &target)` |
| Pointer to value | `new(expr)` (Go 1.26+) | `x := val; return &x` (for non-composite types) |
| Multi-handler log | `slog.NewMultiHandler` (Go 1.26+) | manual fan-out |
| String split | `strings.Cut` | `strings.Index` + manual slicing |
| String build | `strings.Builder` in loops | repeated `+` or `fmt.Sprintf` |
| WaitGroup goroutine | `wg.Go(fn)` (Go 1.25+) | `wg.Add(1)` + `go func() { defer wg.Done(); ... }()` |

When working on an existing codebase, match the Go version declared in `go.mod` -- do not use features from a newer Go than what the project targets.
