---
title: "Post template (example, not a research article)"
category: writeup  # writeup -> Write-ups page, log -> Research Log page
tags: [template]
# last_modified_at: 2026-10-21  # add when you revise a published post
# published: false              # hides a post without deleting it
---

> **This is a template, not research.** It shows the front matter and formatting this blog supports. To start a new post, copy this file to `_posts/YYYY-MM-DD-your-title.md` and replace the placeholders. Delete this file, or set `published: false` in its front matter, once real posts exist.
{: .notice}

A Research Log entry uses the same format with `category: log`. Keep only the sections it needs.

## Summary

Two or three sentences: what was examined, what was found, and its current status (for example open, fixed, publicly disclosed, or still under investigation).

## Scope

| Field      | Value                            |
|------------|----------------------------------|
| Target     | `protocol or contract name`      |
| Repository | `https://github.com/org/repo`    |
| Commit     | `full 40-character commit hash`  |
| Files      | `src/Example.sol`                |
| Tools      | `tool name and version`          |

## Background

What a reader needs to know about the protocol or mechanism before the details. Cite sources with footnotes[^source] so every claim can be checked.

## Details

Quote code exactly as it appears at the commit listed above, with its file path and line numbers. Fenced code blocks take a language name for highlighting:

```solidity
// src/Example.sol, lines N-M, at commit <hash>
function example(uint256 amount) external {
    // paste the relevant code here, unmodified
}
```

Inline code such as `balanceOf(address)` works too. A diff block is useful for showing a fix:

```diff
- old line
+ new line
```

## Reproduction

The exact commands and environment, so someone else can get the same result:

```bash
forge test --match-test test_Example -vvvv
```

## Impact

What can happen, under which assumptions, and what cannot. Keep what you verified separate from what you suspect.

## Timeline

| Date       | Event       |
|------------|-------------|
| YYYY-MM-DD | Description |

## References

1. Author or project, "Title", URL, accessed YYYY-MM-DD.

[^source]: Footnotes are collected at the bottom of the post automatically. Same format as references: Author or project, "Title", URL, accessed YYYY-MM-DD.
