# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- `ROADMAP.md` — staged implementation plan: three gated stages (manual runs with measurement window, weekly scheduled run ending at the preview, infrastructure only on a measured gap), metrics, risks and repo versioning per stage; linked from both READMEs

## [1.1.0] - 2026-10-05

First public release. Includes the changes made after the reference run of 2026-10-04.

### Added
- `config.example.yml` — all Notion IDs, area options, constraints and scoring context live in a local, gitignored `config.yml` instead of the skill text
- Candidate target «new entry | into the seed», so evaluation designs and similar refinements enrich the seed instead of creating sparks
- Provenance block with «possible next step» on every new spark, so the weekly review can promote it without chat context
- One finding about the knowledge base itself in the closing report
- Setup section with topic back-fill for Cockpit entries without topic relations

### Changed
- Retrieval budget counts only new entries; already-linked entries are loaded in one SQL batch and treated as known
- Themen-Hub entry via the topic relations of the seed's linked papers and tools before falling back to search
- Operator F reads the People pages; objections derived only from a role are flagged and never create an «Inspiriert von» relation
- Approval split into two widget questions with at most four options each
- Write-back order: new sparks first, then seed enrichment, so the seed can link to the new URLs

## [1.0.0] - 2026-10-04

### Added
- Initial version of the skill: seed intake, budgeted retrieval, six combination operators, Cockpit-native scoring, two write-back modes, success metric (not published)
