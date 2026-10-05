# Contributing

Thanks for taking the time to improve this skill.

## What helps most

- **Findings from real runs.** The skill is calibrated on a reference run; every further run against a different knowledge base sharpens it. Open an issue with the retrieval line (how many topics, papers, tools, ideas, fragments, people) and what the operators produced — without pasting content from your Notion workspace.
- **Operator proposals.** A new combination operator needs a one-paragraph description, a prompt template in the style of operators A–F, and an example of a candidate it would produce that the existing operators would not.
- **Schema variants.** If your Idea Cockpit uses different field names or maturity levels, describe the mapping rather than changing the defaults.

## Ground rules

- Keep `SKILL.md` free of workspace-specific IDs — those go in `config.example.yml` as placeholders.
- German text uses Swiss spelling (ss, never ß). The READMEs stay bilingual: change both.
- Add an entry under `[Unreleased]` in `CHANGELOG.md` for every change that affects behaviour.
- One pull request per concern.

## Workflow

1. Fork and create a branch from `main`.
2. Make the change, update `CHANGELOG.md` and, if structure or features changed, both READMEs.
3. Open a pull request describing what changed and why.
