---
name: md-line-breaks
description: Use when consecutive markdown lines collapse into a single paragraph and need to render separately, e.g., metadata blocks like `**Author:** Name` / `**Date:** Value`.
---

# Markdown Line Breaks

In markdown, consecutive lines without a blank line between them merge into a single paragraph. Append `<br>` to each line (except the last) to force separate lines.

## Example

Without `<br>`, these lines collapse into one paragraph:

```
**Authors:** Alice
**Date:** Jan 01, 2026
**Status:** Draft
```

With `<br>`:

```
**Authors:** Alice<br>
**Date:** Jan 01, 2026<br>
**Status:** Draft
```

## When NOT to apply

- List items, table rows, code blocks (already handle line breaks)
- Lines separated by blank lines (already distinct paragraphs)
