---
name: go-table-driven-tests
description: Use when writing tests, creating test functions, adding test cases, or when the user mentions "test", "table-driven", "Go tests", or testing in Go codebases.
---

# Go Table-Driven Tests

## Core Principles

- **Red/green TDD** - Write tests first, confirm they fail (red), then implement until they pass (green)
- **One test function, many cases** - Define test cases in a slice and iterate with `t.Run()`
- **Explicit naming** - Each case has a `name` field that becomes the subtest name
- **Structured inputs** - Use struct fields for inputs, expected outputs, and configuration
- **Helper functions** - Use `t.Helper()` in test helpers for proper line reporting
- **Environment guards** - Skip integration tests when credentials are unavailable

## Table Structure Pattern

```go
func TestFunctionName(t *testing.T) {
    tests := []struct {
        name        string              // required: subtest name
        input       Type                // function input
        want        Type                // expected output
        wantErr     error               // expected error (nil for success)
        errCheck    func(error) bool    // optional: custom error validation
        setupEnv    func() func()       // optional: env setup, returns cleanup
    }{
        {
            name: "descriptive case name",
            input: "test input",
            want: "expected output",
        },
        // ... more cases
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // test implementation using tt fields
        })
    }
}
```

## Field Guidelines

| Field          | Required | Purpose                                              |
| -------------- | -------- | ---------------------------------------------------- |
| `name`         | Yes      | Subtest name - be descriptive and specific           |
| `input`/`args` | Varies   | Input values for the function under test             |
| `want`/`want*` | Varies   | Expected output values (e.g., `wantErr`, `wantResult`) |
| `errCheck`     | No       | Custom error validation function                     |
| `setupEnv`     | No       | Environment setup function returning cleanup         |

## Naming Conventions

- Test function: `Test<FunctionName>` or `Test<FunctionName>_<Scenario>`
- Subtest names: lowercase, descriptive, spaces allowed
- Input fields: match parameter names or use `input`/`args`
- Output fields: prefix with `want` (e.g., `want`, `wantErr`, `wantResult`)

## Common Patterns

### 1. Basic Table Test

```go
func TestWithRegion(t *testing.T) {
    tests := []struct {
        name   string
        region string
    }{
        {"auto region", "auto"},
        {"us-west-2", "us-west-2"},
        {"eu-central-1", "eu-central-1"},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            o := &Options{}
            WithRegion(tt.region)(o)

            if o.Region != tt.region {
                t.Errorf("Region = %v, want %v", o.Region, tt.region)
            }
        })
    }
}
```

### 2. Error Checking with `wantErr`

```go
func TestNew_errorCases(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        wantErr error
    }{
        {"empty input", "", ErrInvalidInput},
        {"invalid input", "!!!", ErrInvalidInput},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := Parse(tt.input)
            if !errors.Is(err, tt.wantErr) {
                t.Errorf("error = %v, want %v", err, tt.wantErr)
            }
        })
    }
}
```

### 3. Custom Error Validation with `errCheck`

```go
func TestNew_customErrors(t *testing.T) {
    tests := []struct {
        name     string
        setupEnv func() func()
        wantErr  error
        errCheck func(error) bool
    }{
        {
            name: "missing config returns ErrMissingConfig",
            setupEnv: func() func() { return func() {} },
            wantErr:  ErrMissingConfig,
            errCheck: func(err error) bool {
                return errors.Is(err, ErrMissingConfig)
            },
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            cleanup := tt.setupEnv()
            defer cleanup()

            _, err := New(context.Background())

            if tt.wantErr != nil {
                if err == nil {
                    t.Fatalf("expected error %v, got nil", tt.wantErr)
                }
                if tt.errCheck != nil && !tt.errCheck(err) {
                    t.Errorf("error = %v, want %v", err, tt.wantErr)
                }
            } else if err != nil {
                t.Errorf("unexpected error: %v", err)
            }
        })
    }
}
```

### 4. Error Checking with `assert.ErrorAssertionFunc` (testify)

When using testify, prefer [`assert.ErrorAssertionFunc`](https://pkg.go.dev/github.com/stretchr/testify/assert#ErrorAssertionFunc) over `func(error) bool` for the error assertion field. Its signature matches testify's own assertion functions, so `assert.NoError` and `assert.Error` drop in as values — no wrapper needed. Custom checks get full testify reporting via `assert.TestingT`.

```go
func TestParse(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        wantErr assert.ErrorAssertionFunc
    }{
        {"valid input", "foo", assert.NoError},
        {"empty input", "", assert.Error},
        {"specific error", "???", func(t assert.TestingT, err error, args ...any) bool {
            return assert.ErrorIs(t, err, ErrInvalidInput, args...)
        }},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := Parse(tt.input)
            tt.wantErr(t, err)
        })
    }
}
```

This is more declarative than `wantErr bool` (which can't identify which error) and cleaner than `func(error) bool` (which lacks testify's reporting).

### 5. Environment Setup with `setupEnv`

```go
func TestNew_envVarOverrides(t *testing.T) {
    tests := []struct {
        name     string
        setupEnv func() func()
        options  []Option
        wantErr  error
    }{
        {
            name: "endpoint from env var",
            setupEnv: func() func() {
                os.Setenv("SERVICE_ENDPOINT", "http://localhost:8080")
                return func() { os.Unsetenv("SERVICE_ENDPOINT") }
            },
            wantErr: nil,
        },
        {
            name: "option overrides env var",
            setupEnv: func() func() {
                os.Setenv("SERVICE_ENDPOINT", "http://localhost:8080")
                return func() { os.Unsetenv("SERVICE_ENDPOINT") }
            },
            options: []Option{
                func(o *Options) { o.Endpoint = "http://custom:9090" },
            },
            wantErr: nil,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            cleanup := tt.setupEnv()
            defer cleanup()

            _, err := New(context.Background(), tt.options...)

            if !errors.Is(err, tt.wantErr) {
                t.Errorf("error = %v, want %v", err, tt.wantErr)
            }
        })
    }
}
```

## Integration Test Guards

For tests requiring real credentials, use a skip helper:

```go
// skipIfNoCreds skips the test if service credentials are not set.
// Use this for integration tests that require a live service connection.
func skipIfNoCreds(t *testing.T) {
    t.Helper()
    if os.Getenv("SERVICE_API_KEY") == "" {
        t.Skip("skipping: SERVICE_API_KEY not set")
    }
}

func TestCreate(t *testing.T) {
    tests := []struct {
        name    string
        input   Record
        wantErr error
    }{
        {
            name:  "create valid record",
            input: Record{Name: "example"},
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            skipIfNoCreds(t)

            // test implementation
        })
    }
}
```

## Test Helpers

Use `t.Helper()` in helper functions for proper line number reporting:

```go
func setupTestRecord(t *testing.T, ctx context.Context, client *Client) string {
    t.Helper()
    skipIfNoCreds(t)

    id := "test-" + randomID()
    err := client.Create(ctx, Record{ID: id})
    if err != nil {
        t.Fatalf("failed to create test record: %v", err)
    }
    return id
}

func cleanupTestRecord(t *testing.T, ctx context.Context, client *Client, id string) {
    t.Helper()
    err := client.Delete(ctx, id)
    if err != nil {
        t.Logf("warning: failed to cleanup test record %s: %v", id, err)
    }
}
```

## Checklist

When writing table-driven tests:

- [ ] Table struct has `name` field as first field
- [ ] Each test case has a descriptive name
- [ ] Input fields use clear naming (match parameters or use `input`)
- [ ] Expected output fields prefixed with `want`
- [ ] Iteration uses `t.Run(tt.name, func(t *testing.T) { ... })`
- [ ] Error checking uses `errors.Is()` for error comparison
- [ ] Environment setup includes cleanup in `defer`
- [ ] Integration tests use `skipIfNoCreds(t)` helper
- [ ] Test helpers use `t.Helper()` for proper line reporting
- [ ] Test file is `*_test.go` and lives next to the code it tests

## Best Practices

### Fatalf vs Errorf

Use `t.Fatalf` to stop a subtest immediately on a precondition failure (e.g., unexpected error). Use `t.Errorf` when the test can continue — this reveals whether failures are systematic or isolated to specific cases.

```go
got, err := ParseConfig(tt.input)
if !errors.Is(err, tt.wantErr) {
    t.Fatalf("ParseConfig() error = %v, want %v", err, tt.wantErr)
}
if tt.wantErr == nil && got.Port != tt.want.Port {
    t.Errorf("ParseConfig().Port = %d, want %d", got.Port, tt.want.Port)
}
```

Always include both actual and expected values in error messages: `t.Errorf("got %q, want %q", got, tt.want)`.

### Maps for Test Cases

Consider using a map instead of a slice for test cases. Map iteration order is non-deterministic, which ensures test cases are truly independent:

```go
tests := map[string]struct {
    input string
    want  string
}{
    "empty string":       {input: "", want: ""},
    "single character":   {input: "x", want: "x"},
    "multi-byte glyph":   {input: "🎉", want: "🎉"},
}

for name, tt := range tests {
    t.Run(name, func(t *testing.T) {
        got := process(tt.input)
        if got != tt.want {
            t.Errorf("got %q, want %q", got, tt.want)
        }
    })
}
```

### Parallel Testing

Add `t.Parallel()` calls to run test cases in parallel. The loop variable is automatically captured per iteration:

```go
func TestFunction(t *testing.T) {
    tests := []struct {
        name string
        input string
    }{
        // ... test cases
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel() // marks this subtest as parallel
            // test implementation
        })
    }
}
```

### Testify

Consider `github.com/stretchr/testify` (`assert`/`require`) when it would simplify assertions — particularly for deep equality checks, nil checks, and producing readable diffs on failure. Avoid it for trivial comparisons where a one-line `if` + `t.Errorf` is just as clear. If the project already uses testify, prefer it for consistency.

When using testify, prefer `assert.ErrorAssertionFunc` for the error assertion field in table tests — see pattern 4 above.

## References

- [Go Wiki: TableDrivenTests](https://go.dev/wiki/TableDrivenTests) - Official Go community best practices for table-driven testing
- [Go Testing Package](https://pkg.go.dev/testing/) - Standard library testing documentation
- [Prefer Table Driven Tests](https://dave.cheney.net/2019/05/07/prefer-table-driven-tests) - Dave Cheney's guide on when and why to use table-driven tests over traditional test structures
