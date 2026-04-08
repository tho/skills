# Pre-Commit Checklist

Run before every commit:

- [ ] All tests pass (`go test -race ./...`)
- [ ] `go vet ./...` is clean
- [ ] `golangci-lint run --new` is clean (at minimum; `--new-from-merge-base=main` in CI)
- [ ] `govulncheck ./...` reports no issues
- [ ] All exported symbols have doc comments
- [ ] No commented-out code or debug prints
- [ ] No hardcoded credentials
- [ ] `go mod tidy` has been run
