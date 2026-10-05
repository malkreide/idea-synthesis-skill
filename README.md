# idea-synthesis-skill

![Version](https://img.shields.io/badge/version-1.1.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude Skill](https://img.shields.io/badge/Claude-Skill-orange)

> A Claude Skill that develops an existing idea by combining papers, tools, topics and people from your Notion knowledge base — every candidate carries full provenance.

🇩🇪 [Deutsche Version](README.de.md)

## Overview

Most idea pipelines fail at the opposite end from where people expect: not for lack of ideas, but because loose sparks never get connected to what you already know. This skill takes one **seed** — an entry in your Notion Idea Cockpit or a problem statement — and pulls in the papers, tools, topics and people from your own knowledge base that move it forward. It then applies six fixed combination operators, scores the candidates, and writes the selected ones back to Notion as traceable sparks.

It deliberately does **not** generate ideas from nothing (that is the job of a separate serendipity agent). Pull, not push: a seed is always required, and every candidate names the Notion entries it was built from.

## Features

- **Seed-based** — starts from a Cockpit entry (title or URL) or a one-sentence problem
- **Budgeted retrieval** — about 40 new entries and 15 tool calls per run; Themen-Hub first, then semantic search per database, SQL batch loading for linked entries
- **Six combination operators** — mechanism transfer, tool substitution, constraint tightening, morphological box, neighbour analogy, stress test against documented positions
- **Provenance-first** — a candidate without Notion URLs is never shown
- **Cockpit-native scoring** — strategic value, feasibility, learning value, evidence (1–5), plus privacy flag and effort
- **Two write-back modes** — new sparks with a provenance block, or enrichment of the seed with relations, scores and a dated synthesis section
- **Measurable** — a tag and a filtered view make the only metric that matters countable: how many synthesis sparks get promoted
- **Configuration outside the skill** — all Notion IDs live in a local, gitignored `config.yml`

## Prerequisites

- Claude with the **Notion MCP connector** (read and write access to your workspace); tested in Claude Code and claude.ai
- A Notion workspace with at least these databases: an Idea Cockpit (ideas with maturity levels), a Themen-Hub (topics as the bridge between databases), an AI-Tools database and a Source Library (papers and resources). Fragmente, People and project databases are optional.
- The Idea Cockpit schema described in [SKILL.md](SKILL.md) — the four score fields, the maturity select and the relations to the other databases

## Installation

Claude Code — clone into your skills folder:

```bash
git clone https://github.com/malkreide/idea-synthesis-skill.git ~/.claude/skills/idea-synthesis
cp ~/.claude/skills/idea-synthesis/config.example.yml ~/.claude/skills/idea-synthesis/config.yml
```

Then fill `config.yml` with the data-source IDs of your workspace (`config.yml` is gitignored).

claude.ai — upload `SKILL.md` as a custom skill and keep `config.yml` next to it, or paste its values into the skill's configuration section.

## Usage

Address the skill with a seed:

```text
Develop «Local RAG knowledge system» further — what do I already have on this?
```

```text
Our team leads keep asking the same questions about internal directives. What could I build from my knowledge base?
```

```text
Turn the spark «Letter template generator» into a concept.
```

The skill loads the seed and its linked entries, retrieves candidates from each database, applies the operators, presents 3–5 scored candidates with provenance, and asks two questions: which candidates to create as new sparks, and whether to enrich the seed. Nothing is written to Notion before that approval.

## How it works

| Step | What happens |
|---|---|
| 0 · Seed | Load the Cockpit entry or ask for area and constraints; already-linked entries count as known |
| 1 · Retrieval | Themen-Hub first (via the seed's or its sources' topic relations), then semantic search and SQL batch per database, within budget |
| 2 · Operators | A mechanism transfer · B tool substitution · C constraint tightening · D morphological box · E neighbour analogy · F stress test |
| 3 · Scoring | Duplicate check, target (new entry or into the seed), four Cockpit scores, privacy flag, effort |
| 4 · Approval | Compact table plus one block per candidate; two widget questions, max. four options each |
| 5 · Write-back | New sparks first (tag, icon, provenance block, relations), then seed enrichment |
| 6 · Report | Four lines, including one finding about the knowledge base itself |

## Configuration

Copy `config.example.yml` to `config.yml` and set:

| Key | Purpose |
|---|---|
| `notion.idea_cockpit.*` | Database ID, data-source ID and (optional) the measuring view |
| `notion.themen_hub`, `notion.ai_tools`, `notion.source_library` | Required data sources |
| `notion.fragmente`, `notion.people`, `notion.optional.*` | Optional data sources — leave empty if absent |
| `cockpit.bereiche` | Options of the Cockpit's «Bereich» select, max. four |
| `cockpit.tag_synthese`, `cockpit.icon` | How synthesis entries are marked |
| `constraints` | Options for operator C and the intake question, max. four |
| `scoring_context` | What «strategic value» should be measured against |

One-time setup in Notion: add the synthesis tag to the Cockpit's Tags field and create the filtered view. The skill describes both, plus an optional topic back-fill for Cockpit entries that have no topic relations yet.

## Project Structure

```
idea-synthesis-skill/
├── SKILL.md              ← the skill (German, Swiss spelling)
├── config.example.yml    ← template for the local, gitignored config.yml
├── README.md             ← this file
├── README.de.md          ← German version
├── CHANGELOG.md
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE
└── .github/
    └── repo-meta.yml     ← repository metadata
```

## Related skills

The skill is one of three that share the same Idea Cockpit: `idea-cockpit` captures and triages, `fragment-triage` collects scattered fragments, and `idea-serendipity` combines entries freely without a seed. All three write only sparks — the weekly review is the quality gate.

## Changelog

See [CHANGELOG.md](CHANGELOG.md)

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

## Security

Please report vulnerabilities as described in [SECURITY.md](SECURITY.md).

## License

MIT License — see [LICENSE](LICENSE)

## Author

Hayal Özkan · [malkreide](https://github.com/malkreide)
