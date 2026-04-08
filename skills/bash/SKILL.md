---
name: bash
description: This skill should be used when writing, reviewing, or modifying shell scripts (.sh files, bash scripts, POSIX sh scripts), or when the user asks to "write a bash script", "fix my shell script", "make this script safe", "handle errors in bash", "parse arguments in bash", "run shellcheck on my script", or "debug my shell script", or asks about shell scripting idioms, quoting, portability, argument parsing, file iteration, or arrays.
---

# Shell Script Quality

Clear, boring scripts beat clever ones. Prefer explicit over implicit, simple over terse.

## Core Principles

The primary goal of shell scripts is **correctness and robustness**. A script that handles edge cases, validates inputs, and fails safely is worth more than one that saves a few forks.

Write correct, robust scripts by:

- quoting all variable expansions (the single biggest source of shell bugs)
- validating inputs and checking exit codes explicitly
- using shell builtins over external commands where equivalent and readable (`[[ ]]` over `test`, parameter expansion over `sed`/`awk` for simple string operations)
- avoiding unnecessary subshells and forks in loops — but not at the expense of clarity
- following POSIX conventions where portability matters; using bash-specific features only when the shebang declares bash

### Know when to stop writing shell

Shell is the right tool for glue code, automation, build scripts, and small utilities. It is the wrong tool for complex business logic, data processing, or anything that needs structured error handling, data types, or testability beyond basic integration tests.

If a script grows to roughly 200 lines or more, needs to parse structured data (JSON, YAML, CSV), requires non-trivial error recovery, or would benefit from real data structures — rewrite it in Go, Python, or another general-purpose language. Do not fight the shell to do things it was not designed to do.

## Target Shell

- **MUST** declare the interpreter on line 1: `#!/usr/bin/env bash` for bash, `#!/bin/sh` for POSIX sh.
- Choose based on requirement: if the script must run on minimal systems (Alpine, BusyBox, `dash`), write POSIX sh. If you need arrays, `[[ ]]`, process substitution, or `local`, use bash.
- **NEVER** mix bash-isms into a script with `#!/bin/sh`.
- Do not assume `/bin/bash` exists on every system; prefer `#!/usr/bin/env bash`.
- Target bash 4.x or later. macOS ships bash 3.2 — document if the script requires a newer version.

## Safety Flags

Every script MUST start with:

```sh
set -euo pipefail
```

- `set -e`: exit immediately on error
- `set -u`: treat unset variables as errors
- `set -o pipefail`: propagate pipeline failures

If the script does not rely on globbing, also add `set -f` to prevent accidental glob expansion of unquoted variables; disable locally with `set +f` when needed.

**`set -e` caveats:**

- Commands in `if`, `&&`, `||`, or `while` conditions are **not** subject to `set -e`. A function called via `if myfunc` silently swallows internal failures. Use explicit `|| return` or `|| exit` inside functions.
- Command substitution (`$(...)`) disables `set -e` inside the substitution.
- **NEVER** rely on `set -e` as a substitute for explicit error checking. It is a safety net, not primary error handling.

**`set -o pipefail` caveats:**

- Always enable `pipefail`. When a pipeline stage exits early intentionally (e.g., `grep -q`, `head -1`), handle the exit explicitly rather than disabling `pipefail` globally:

```sh
if grep -q "pattern" file || true; then ...  # intentional early exit — documented
```

**`set -u` caveats:**

- Use `${var:-}` for a safe empty default or `[[ -v var ]]` to test whether a variable is set.

**NEVER** disable these flags without a documented reason. Handle expected failures explicitly:

```sh
if ! some_command; then
  handle_failure
fi

some_command || true  # intentional — documented reason here
```

## Security

- **NEVER** store secrets in scripts. Read from environment or a secrets manager.
- **NEVER** print or log variables that may contain secrets.
- **NEVER** use `eval` on user input or construct commands via string concatenation.
- Use `--` to signal end of options: `rm -- "$file"`.
- **MUST** validate file paths before `rm -rf`.
- Set `chmod 600` on files containing sensitive data.

## Code Style and Formatting

- **MUST** use 2-space indentation (or tabs if the project already uses them — be consistent).
- **MUST** use `snake_case` for variable and function names.
- **MUST** quote all variable expansions: `"$var"`, `"${var}"`, `"$@"`.
- **NEVER** use `$*` — use `"$@"` to preserve argument boundaries.
- Use `${variable}` braces when concatenating: `"${prefix}_suffix"`.
- Use `UPPER_CASE` for environment variables and constants; `snake_case` for locals.
- **MUST** use `readonly` for constants: `readonly MAX_RETRIES=3`.
- Use `local` for variables in functions.
- **MUST** avoid redundant comments that restate what the code obviously does.
- **MUST** avoid comments that leak the original user prompt or meta-context.
- Prefer early returns/exits to reduce nesting.

## Quoting

Incorrect quoting is the root cause of the majority of shell bugs. Unquoted variables undergo word splitting on `IFS` characters AND glob expansion.

- **ALWAYS** double-quote parameter expansions: `"$var"`, `"$(cmd)"`, `"${array[@]}"`.
- Single quotes protect everything literally — no expansion occurs inside `'...'`.
- **NEVER** use `"~"` or `"~/path"` — tilde does not expand inside double quotes. Use `"$HOME"`.
- `export foo=~/bar` does not reliably expand the tilde. Use `export foo="$HOME/bar"`.

## Variables and Expansion

- Use `${var:-default}` for defaults, `${var:?error message}` to abort on unset.
- Prefer parameter expansion over external tools for simple string operations: `${str#prefix}`, `${str%suffix}`, `${str/old/new}`, `${str,,}` (bash 4+).
- **NEVER** put commands in variables. Variables hold data; functions hold code. Use a function or an array.
- Brace expansion (`{1..10}`) happens before variable expansion — `{1..$n}` does not work. Use `for ((i=1; i<=n; i++))`.
- Command substitution strips trailing newlines. Sentinel: `out=$(cmd; printf x); out=${out%x}`.

## Conditionals

- In bash, use `[[ ]]` exclusively. In POSIX sh, use `[ ]` with **all variables quoted**.
- **NEVER** use `==` inside `[ ]` (not POSIX). Use `=`.
- **NEVER** use `-a` or `-o` inside `[ ]`. Use `[ A ] && [ B ]` instead.
- Inside `[[ ]]`, an unquoted RHS of `==` is a glob pattern. Quote for literal comparison.
- For `=~` regex matching, store the regex in a variable and expand unquoted: `[[ $str =~ $re ]]`.
- Use `(( ))` for numeric comparisons in bash. `[[ $foo > 7 ]]` is string comparison.
- `(( ))` returns exit 1 when the result is 0 — careful with `set -e` and `(( count++ ))` when count is 0.
- **NEVER** use unvalidated input in arithmetic contexts (`$(( ))`, `let`, array indices) — allows code injection.
- `cmd1 && cmd2 || cmd3` is **not** if/else. If `cmd2` fails, `cmd3` also runs. Use `if/then/else/fi`.
- Test commands directly: `if cmd; then` — not `cmd; if [ $? -eq 0 ]; then`.

## Functions

- Declare with `foo()` syntax. In bash, `function foo` is also acceptable but adds no value. Avoid `function foo()` — it combines both and is not POSIX.
- **MUST** use `local` for all variables.
- **MUST** document purpose, arguments, and return value/exit code above each function.
- Functions return exit codes (0–255). Return data via stdout.
- **NEVER** use `exit` in a function — use `return`.
- `local var=$(cmd)` masks `cmd`'s exit status. Declare and assign separately:

```sh
local result
result=$(cmd) || return 1
```

The same applies to `export foo=$(cmd)` and `readonly foo=$(cmd)`.

## Input and Arguments

- **MUST** validate required arguments early with a `usage()` function.
- Use `getopts` for option parsing. **NEVER** use `getopt` (the external command) — platform-dependent. Minimal pattern:

```sh
usage() { echo "Usage: $0 [-v] [-o outfile] arg" >&2; exit 1; }
verbose=0; outfile=""
while getopts ":vo:" opt; do
  case $opt in
    v) verbose=1 ;;
    o) outfile="$OPTARG" ;;
    :) echo "error: -$OPTARG requires an argument" >&2; usage ;;
    \?) echo "error: unknown option -$OPTARG" >&2; usage ;;
  esac
done
shift $((OPTIND - 1))
```
- **NEVER** pass unvalidated input to `eval`, arithmetic contexts, or `find -exec sh -c 'echo {}'`. Pass as positional arguments: `find . -exec sh -c 'echo "$1"' _ {} \;`.

## Error Handling

- **MUST** write error messages to stderr: `echo "error: ..." >&2`.
- Use a `die` helper:

```sh
die() { echo "error: $*" >&2; exit 1; }
```

- Use `trap` for cleanup:

```sh
TMP_DIR="$(mktemp -d)" || die "failed to create temp dir"
readonly TMP_DIR
trap 'rm -rf "$TMP_DIR"' EXIT
```

- **ALWAYS** check `cd` for failure: `cd /some/dir || die "cannot cd to /some/dir"`.

## Reading Input and Iterating Files

**NEVER** iterate with `for f in $(ls ...)` or `for f in $(find ...)` -- breaks on filenames with spaces or glob characters. **NEVER** parse `ls` output.

Use globs (`for f in ./*.mp3`), `find -exec`, or `while IFS= read -r` with process substitution. **NEVER** use `arr=( $(cmd) )` -- use `readarray -t arr < <(cmd)`.

See **`references/file-iteration.md`** for complete patterns including safe array population, `while read` with dedicated fds, and pipeline variable scope.

## Output

- In POSIX sh, use `printf` over `echo` for predictability. In bash, `echo` is fine for simple messages.
- **NEVER** use `printf "$foo"` — format string injection. Use `printf '%s\n' "$foo"`.
- Use heredocs (`<<EOF`) for multi-line output.

## Redirection

Redirections are processed left to right. Order matters:

```sh
# Wrong — stderr goes to terminal
cmd 2>&1 >logfile

# Right
cmd >logfile 2>&1
```

`sudo cmd > /file` runs the redirect as the current user. Use `sudo sh -c 'cmd > /file'` or `cmd | sudo tee /file >/dev/null`.

- **NEVER** close stderr with `2>&-`. Use `2>/dev/null`.

## File and Path Handling

- Use `mktemp` for temporary files — never hardcode paths.
- Use `${BASH_SOURCE[0]}` (not `$0`) to find a script's own location.
- **NEVER** use `rm -rf` on an unvalidated variable:

```sh
if [[ -n "$tmp_dir" && "$tmp_dir" == /tmp/* ]]; then
  rm -rf "$tmp_dir"
fi
```

- Use `--` to protect against filenames starting with `-`: `cp -- "$file" "$target"`.
- `-e` follows symlinks. A broken symlink returns false. Use `[[ -e "$f" || -L "$f" ]]`.
- **Never export `CDPATH`** — it causes `cd` in child scripts to resolve to unexpected directories and pollutes command substitutions with printed paths.
- An empty `IFS` (`IFS=`) and an unset `IFS` have different effects. Use `local IFS` inside functions or a subshell — do not save/restore with `OIFS="$IFS"`.

## Subshells and External Commands

- Prefer builtins in loops. `result=$(<file)` avoids `cat`.
- Use `command -v` to check tool availability, not `which`.
- Call external commands once for related values to avoid race conditions:

```sh
# Wrong — month and day may span midnight
month=$(date +%m); day=$(date +%d)

# Right
read -r month day year <<< "$(date '+%m %d %Y')"
```

- Use `xargs -0` with `find -print0`. `xargs` without `-0` breaks on whitespace and quotes in filenames.

## Arrays (bash only)

- Use arrays instead of space-delimited strings when items may contain spaces.
- Declare associative arrays explicitly: `declare -A mymap`.
- Quote subscripts when unsetting: `unset -v 'a[0]'`.
- Avoid `(( hash[$key]++ ))` — the key is evaluated as arithmetic, allowing injection. Use a temporary variable.

## Portability

- POSIX sh: no arrays, `[[ ]]`, `(( ))`, `$'...'`, brace expansion, or process substitution.
- Platform differences: `date -d` is GNU only; `sed -i` needs `''` on BSD/macOS; `grep -P` is not universal (use `-E`); `tr [A-Z] [a-z]` glob-expands brackets and is locale-dependent — use `tr '[:upper:]' '[:lower:]'`.

## Documentation

- Include a header comment for non-trivial scripts: purpose, usage, required environment variables, and dependencies.
- Document functions with purpose, arguments, and return value when the signature alone is not self-explanatory.

Example header comment:

```sh
#!/usr/bin/env bash
# deploy.sh: build and deploy the application to a target environment.
#
# Usage: deploy.sh <environment>
#   environment  One of: staging, production
#
# Requires: AWS_PROFILE, DEPLOY_BUCKET
# Dependencies: aws-cli, jq
set -euo pipefail
```

## Testing

Most shell scripts are short enough that ShellCheck and manual testing suffice. For scripts maintained long-term or containing reusable library functions, use [bats-core](https://github.com/bats-core/bats-core). Test argument validation and error paths first; integration tests belong in a separate suite.

See **`references/testing.md`** for a bats-core setup example.

## Linting and Static Analysis

- **MUST** run [ShellCheck](https://www.shellcheck.net/) on all scripts: `shellcheck -S warning -x <script>`. Verify availability first with `command -v shellcheck`; if absent, inform the user to install it and skip the check rather than erroring out.
- Use `shfmt` for formatting: `shfmt -i 2 -ci -w <script>`.
- **NEVER** suppress ShellCheck warnings without a documented reason inline:

```sh
# shellcheck disable=SC2086  # intentional word-splitting: $flags is a space-separated option list
```

## Pitfall Quick Reference

| Pattern | Problem | Fix |
|---|---|---|
| `for f in $(ls *.mp3)` | Word-splits, globs, mangles names | `for f in ./*.mp3` |
| `cp $file $target` | Word-splits and glob-expands | `cp -- "$file" "$target"` |
| `[ $foo = bar ]` | Fails on empty/spaces | `[ "$foo" = bar ]` or `[[ $foo = bar ]]` |
| `printf "$foo"` | Format string injection | `printf '%s\n' "$foo"` |
| `[[ $foo > 7 ]]` | String comparison, not numeric | `(( foo > 7 ))` |
| `cmd1 && cmd2 \|\| cmd3` | `cmd3` runs if `cmd2` fails | `if/then/else/fi` |
| `local var=$(cmd)` | `local` masks exit status | Declare then assign |
| `export foo=~/bar` | Tilde may not expand | `export foo="$HOME/bar"` |
| `function foo()` | Not portable | `foo()` |
| `cd /foo; bar` | `bar` runs in wrong dir if `cd` fails | `cd /foo \|\| exit 1` |
| `cmd 2>&1 >file` | Wrong order; stderr goes to tty | `cmd >file 2>&1` |
| `for i in {1..$n}` | Brace expansion before variable expansion | `for ((i=1; i<=n; i++))` |
| `hosts=( $(aws ...) )` | Word-splits and globs output | `readarray -t hosts < <(aws ...)` |
| `find . -exec sh -c 'echo {}'` | Code injection via filename | `find . -exec sh -c 'echo "$1"' _ {} \;` |
| `sudo cmd > /file` | Redirect runs as original user | `sudo sh -c 'cmd > /file'` |
| `myprogram 2>&-` | Closing stderr crashes programs | `myprogram 2>/dev/null` |

## Before Committing

- [ ] All variable expansions quoted
- [ ] All `cd` calls check for failure
- [ ] Temporary files cleaned up via `trap`
- [ ] No hardcoded credentials or secrets
- [ ] No debug `echo` or `set -x` left in
- [ ] `shellcheck` is clean
- [ ] `shfmt -d` reports no differences
