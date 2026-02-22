# Contributing to unhallucinate

Thanks for your interest in contributing.

## What's welcome

- **New failure modes (FM-XX)**: If you've encountered a specific, repeatable way AI generates wrong code that isn't already in `skill/references/ai-failure-modes.md`, open an issue or PR with the pattern, what causes it, which doc type prevents it, and common wrong defaults.

- **Template improvements**: Better section structure, missing fields, or clearer guidance in `skill/references/document-templates.md`.

- **Context engineering patterns**: New strategies for feeding specs into AI context effectively, tested in real projects.

- **Audit checklist items**: New checks for `skill/references/audit-checklist.md` that map to documented failure modes.

- **Bug fixes**: Typos, broken links, inconsistencies between docs.

## What's NOT in scope

- Adding support for non-Claude AI tools (the skill definition is Claude-specific, but the workflow and documents work with any AI coding tool)
- Rewriting the workflow to remove the GSD integration (GSD is optional; document the alternative instead)
- Adding opinionated tech stack recommendations (the framework is stack-agnostic)

## How to contribute

1. Open an issue describing what you want to change and why
2. Fork the repo
3. Make your changes
4. Submit a PR referencing the issue

For new failure modes, include:
- A clear description of what goes wrong
- The conditions that trigger it (what's missing from specs)
- Which document type prevents it
- At least one example of the wrong default AI generates
