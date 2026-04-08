# Testing Shell Scripts

Most shell scripts are short enough that ShellCheck and manual testing suffice. For scripts maintained long-term or containing reusable library functions, use [bats-core](https://github.com/bats-core/bats-core).

```bash
# test/deploy.bats
setup() {
  load 'test_helper/bats-support/load'
  load 'test_helper/bats-assert/load'
}

@test "fails with no arguments" {
  run ./deploy.sh
  assert_failure
  assert_output --partial "Usage:"
}

@test "fails on unknown environment" {
  run ./deploy.sh unknown
  assert_failure
  assert_output --partial "unknown environment"  # must match the script's actual error text
}
```

Run with `bats test/`. Test argument validation and error paths first; integration tests that invoke real side effects require a configured environment and belong in a separate suite.
