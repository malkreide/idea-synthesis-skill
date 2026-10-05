# Security Policy

## Scope

This repository contains a Claude Skill (a Markdown instruction file) and a configuration template. It ships no executable code and runs entirely inside Claude with the Notion MCP connector the user has authorised. The skill writes to the user's own Notion workspace only after an explicit approval step.

## What never belongs in this repository

- Notion data-source IDs, database IDs or view IDs of a real workspace — they belong in the local, gitignored `config.yml`
- Notion integration tokens or any other credentials
- Content of Notion pages, personal data, or names of people from a real knowledge base

If you find any of these in the repository or its history, please report it as described below.

## Reporting a vulnerability

Use GitHub's private vulnerability reporting on this repository (Security → Report a vulnerability). If that is not available to you, open an issue with the title «Security» and no sensitive details; you will receive a way to share them privately.

Reports are acknowledged within seven days. Confirmed issues are fixed in a new release and documented in the changelog.

## Supported versions

Only the latest release receives fixes.
