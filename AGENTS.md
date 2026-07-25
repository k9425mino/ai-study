# Repository Agent Guide

## Repository Purpose

This repository contains personal notes for studying AI technologies. Keep documents clear, concise, and easy to revisit. Match the language and style of the file being edited.

## Working Principles

### Think Before Editing

- Do not silently guess when requirements are materially ambiguous.
- State important assumptions and tradeoffs.
- Prefer the simplest valid interpretation; ask when uncertainty could change the result.

### Keep It Simple

- Make the minimum change needed to satisfy the request.
- Do not add speculative features, abstractions, dependencies, or configuration.
- For trivial Markdown changes, use judgment and avoid unnecessary process.

### Make Surgical Changes

- Touch only files and lines related to the request.
- Preserve existing structure and formatting unless changing them is part of the task.
- Do not clean up unrelated content. Remove only leftovers introduced by your own changes.

### Work Toward Verifiable Goals

- Define a clear outcome before starting multi-step work.
- Verify links, filenames, and Markdown structure after documentation changes.
- Review the final diff and run `git diff --check`.
- This repository currently has no build or automated test suite; do not invent commands that do not exist.

Every changed line should be traceable to the user's request.
