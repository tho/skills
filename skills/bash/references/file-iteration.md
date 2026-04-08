# File Iteration and Reading Input

## NEVER iterate with `for f in $(ls ...)` or `for f in $(find ...)`

Breaks on filenames with spaces, newlines, or glob characters. No fix -- the approach is wrong. **NEVER** parse `ls` output for any purpose -- it is designed for human reading.

```sh
# Right -- use globs
for f in ./*.mp3; do
  [ -e "$f" ] || continue  # guard against no matches (POSIX sh)
  some_command "$f"
done

# Right -- find with -exec
find . -type f -name '*.mp3' -exec some_command {} +

# Right -- find with while read (bash)
while IFS= LC_ALL=C read -r -d '' f; do
  some_command "$f"
done < <(find . -type f -name '*.mp3' -print0)
```

In bash, `shopt -s nullglob` suppresses unmatched globs instead of expanding to the literal pattern.

## Reading lines

**NEVER** use `for line in $(cat file)`. Use `while read`:

```sh
while IFS= read -r line; do
  printf '%s\n' "$line"
done < file
```

`IFS=` preserves whitespace; `-r` prevents backslash interpretation. If a command inside the loop reads stdin (e.g., `ssh`), use a dedicated fd: `while IFS= read -r line <&3; do ...; done 3< file`.

## Populating arrays safely

**NEVER** use `arr=( $(cmd) )`. Use `readarray -t arr < <(cmd)` (bash 4+) or `read -ra arr < <(cmd)`. For bash 3.x compatibility: `IFS=$'\n' read -r -d '' -a arr < <(cmd && printf '\0')`.

## Pipeline variable scope

Variables set in a pipeline subshell are lost. Use process substitution:

```sh
# Wrong -- count is lost
grep foo file | while read -r line; do (( count++ )); done

# Right
while IFS= read -r line; do (( count++ )); done < <(grep foo file)
```
