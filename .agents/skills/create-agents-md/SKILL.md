---
name: create-agents-md
description: Create or update this repository's root AGENTS.md from agentsmd_작성가이드.md, andrej-karpathy.md, openai-agentsmd-example.md, and verified repository facts. Use when asked to generate, rewrite, improve, or review AGENTS.md for this AI study repository.
---

# Create AGENTS.md

Create a concise, repository-specific agent guide. Treat the three source documents as guidance and examples, not text to copy wholesale.

## Workflow

1. Resolve the repository root with `git rev-parse --show-toplevel`.
2. Read these root files completely:
   - `agentsmd_작성가이드.md`
   - `andrej-karpathy.md`
   - `openai-agentsmd-example.md`
3. Read the current root `AGENTS.md` when it exists. Preserve useful project-specific instructions unless the user explicitly requests a replacement.
4. Inspect `README.md`, top-level paths, manifests, and available scripts to establish current repository facts. Do not infer a stack, command, or directory that is not present.
5. Draft or revise the root `AGENTS.md`.
6. Review the diff and validate with `git diff --check`.

If any source document is missing, stop and identify the missing path. Use the actual filename `andrej-karpathy.md`; do not reproduce misspellings from a request.

## Content Rules

Apply the structure guidance from `agentsmd_작성가이드.md`:

- State the repository purpose and relevant structure.
- Include only verified build, test, lint, or validation commands.
- Define concrete working rules and boundaries.
- Link to detailed documents instead of duplicating long explanations.
- Omit sections that do not apply.
- Use the exact uppercase filename `AGENTS.md` without YAML frontmatter.

Apply the behavioral principles from `andrej-karpathy.md`:

- Surface material assumptions and tradeoffs before editing.
- Prefer the minimum solution without speculative features or abstractions.
- Make surgical changes and avoid unrelated cleanup.
- Define verifiable outcomes and continue until they are checked.

Apply the practical patterns from `openai-agentsmd-example.md`:

- Prefer concrete, executable instructions over vague guidance.
- Consider sections for environment setup, development workflow, testing or validation, and Git or pull requests when verified repository facts justify them.
- Adapt the example's structure to this repository instead of copying it.
- Never copy its project names, package-manager commands, framework assumptions, or toolchain details unless they are independently verified in the current repository.

Translate these principles into short, actionable repository instructions. Match the predominant language and tone of the existing repository documentation.

## Repository Fit

- Keep the guide concise enough to load on every task.
- For this Markdown-focused study repository, prioritize document clarity, correct links and filenames, minimal diffs, and `git diff --check`.
- Do not invent an automated test suite or dependency workflow.
- Reinspect the repository on every invocation so the guide stays accurate as the project evolves.

## Completion

- Confirm every instruction is supported by a source document, a verified repository fact, or the user's request.
- Confirm every changed line is relevant to `AGENTS.md`.
- Do not commit, push, or modify unrelated files unless the user explicitly asks.
