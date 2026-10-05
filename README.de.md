# idea-synthesis-skill

![Version](https://img.shields.io/badge/version-1.1.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude Skill](https://img.shields.io/badge/Claude-Skill-orange)

> Ein Claude Skill, der eine bestehende Idee weiterentwickelt, indem er Papers, Tools, Themen und Personen aus der eigenen Notion-Wissensbasis kombiniert — jeder Kandidat mit vollständiger Provenienz.

🇬🇧 [English Version](README.md)

## Übersicht

Ideen-Pipelines scheitern meist nicht am Mangel an Ideen, sondern daran, dass lose Funken nie mit dem verbunden werden, was man schon weiss. Dieser Skill nimmt einen **Seed** — einen Eintrag im Notion Idea Cockpit oder ein Problemstatement — und holt gezielt die Papers, Tools, Themen und Personen aus der eigenen Wissensbasis, die ihn weiterbringen. Dann wendet er sechs feste Kombinations-Operatoren an, bewertet die Kandidaten und schreibt die ausgewählten als nachvollziehbare Funken zurück nach Notion.

Er generiert bewusst **keine** Ideen ins Blaue (das ist Aufgabe eines separaten Serendipity-Agenten). Pull, nicht push: Ein Seed ist immer Voraussetzung, und jeder Kandidat nennt die Notion-Einträge, aus denen er gebaut wurde.

## Funktionen

- **Seed-basiert** — startet von einem Cockpit-Eintrag (Titel oder URL) oder einem Ein-Satz-Problem
- **Retrieval mit Budget** — rund 40 neue Einträge und 15 Tool-Aufrufe pro Lauf; Themen-Hub zuerst, dann semantische Suche je Datenbank, SQL-Batch für verknüpfte Einträge
- **Sechs Kombinations-Operatoren** — Mechanismus-Transfer, Tool-Substitution, Constraint-Verschärfung, morphologischer Kasten, Nachbar-Analogie, Stresstest gegen dokumentierte Positionen
- **Provenienz zuerst** — ein Kandidat ohne Notion-URLs wird nie präsentiert
- **Cockpit-natives Scoring** — Strategischer Wert, Machbarkeit, Lernwert, Evidenz (1–5), dazu Datenschutz-Flag und Aufwand
- **Zwei Rückschreib-Modi** — neue Funken mit Provenienz-Block oder Anreicherung des Seeds mit Relationen, Scores und datiertem Synthese-Abschnitt
- **Messbar** — Tag und gefilterte View machen die einzige Kennzahl zählbar, die zählt: wie viele Synthese-Funken befördert werden
- **Konfiguration ausserhalb des Skills** — alle Notion-IDs liegen in einer lokalen, gitignorierten `config.yml`

## Voraussetzungen

- Claude mit dem **Notion-MCP-Connector** (Lese- und Schreibzugriff auf den Workspace); getestet in Claude Code und claude.ai
- Ein Notion-Workspace mit mindestens diesen Datenbanken: Idea Cockpit (Ideen mit Reifegrad), Themen-Hub (Themen als Brücke zwischen den Datenbanken), AI-Tools und Source Library (Papers und Ressourcen). Fragmente, People und Projekt-Datenbanken sind optional.
- Das in [SKILL.md](SKILL.md) beschriebene Cockpit-Schema — die vier Score-Felder, das Reifegrad-Select und die Relationen zu den anderen Datenbanken

## Installation

Claude Code — in den Skills-Ordner klonen:

```bash
git clone https://github.com/malkreide/idea-synthesis-skill.git ~/.claude/skills/idea-synthesis
cp ~/.claude/skills/idea-synthesis/config.example.yml ~/.claude/skills/idea-synthesis/config.yml
```

Danach `config.yml` mit den Data-Source-IDs des eigenen Workspace füllen (`config.yml` ist gitignored).

claude.ai — `SKILL.md` als eigenen Skill hochladen und `config.yml` daneben ablegen oder die Werte in den Konfigurationsabschnitt des Skills übernehmen.

## Verwendung

Den Skill mit einem Seed ansprechen:

```text
Entwickle «Lokales RAG-Wissenssystem» weiter — was habe ich dazu schon?
```

```text
Die Leitungen fragen dauernd dasselbe zu internen Weisungen. Was lässt sich aus meiner Wissensbasis dafür bauen?
```

```text
Mach aus dem Funken «Briefvorlagen-Generator» ein Konzept.
```

Der Skill lädt den Seed und seine verknüpften Einträge, holt Kandidaten aus jeder Datenbank, wendet die Operatoren an, präsentiert 3–5 bewertete Kandidaten mit Provenienz und stellt zwei Fragen: welche Kandidaten als neue Funken angelegt werden sollen und ob der Seed angereichert werden soll. Vor dieser Freigabe wird nichts nach Notion geschrieben.

## Funktionsweise

| Schritt | Was passiert |
|---|---|
| 0 · Seed | Cockpit-Eintrag laden oder Bereich und Constraints abfragen; bereits verknüpfte Einträge gelten als bekannt |
| 1 · Retrieval | Themen-Hub zuerst (über die Themen des Seeds oder seiner Quellen), dann semantische Suche und SQL-Batch je Datenbank, im Budget |
| 2 · Operatoren | A Mechanismus-Transfer · B Tool-Substitution · C Constraint-Verschärfung · D morphologischer Kasten · E Nachbar-Analogie · F Stresstest |
| 3 · Bewertung | Duplikat-Check, Ziel (neuer Eintrag oder in den Seed), vier Cockpit-Scores, Datenschutz-Flag, Aufwand |
| 4 · Freigabe | Kompakte Tabelle plus ein Block je Kandidat; zwei Widget-Fragen mit max. vier Optionen |
| 5 · Rückschreiben | Zuerst neue Funken (Tag, Icon, Provenienz-Block, Relationen), dann Seed-Anreicherung |
| 6 · Bericht | Vier Zeilen, darunter ein Befund zur Wissensbasis selbst |

## Konfiguration

`config.example.yml` nach `config.yml` kopieren und setzen:

| Schlüssel | Zweck |
|---|---|
| `notion.idea_cockpit.*` | Datenbank-ID, Data-Source-ID und (optional) die Mess-View |
| `notion.themen_hub`, `notion.ai_tools`, `notion.source_library` | Pflicht-Datenquellen |
| `notion.fragmente`, `notion.people`, `notion.optional.*` | Optionale Datenquellen — leer lassen, wenn nicht vorhanden |
| `cockpit.bereiche` | Optionen des Cockpit-Selects «Bereich», max. vier |
| `cockpit.tag_synthese`, `cockpit.icon` | Kennzeichnung der Synthese-Einträge |
| `constraints` | Optionen für Operator C und die Intake-Frage, max. vier |
| `scoring_context` | Woran «Strategischer Wert» gemessen wird |

Einmalige Einrichtung in Notion: die Synthese-Option im Tags-Feld des Cockpit anlegen und die gefilterte View erstellen. Der Skill beschreibt beides, dazu einen optionalen Themen-Backfill für Cockpit-Einträge ohne Themen-Relation.

## Projektstruktur

```
idea-synthesis-skill/
├── SKILL.md              ← der Skill (Deutsch, Schweizer Rechtschreibung)
├── config.example.yml    ← Vorlage für die lokale, gitignorierte config.yml
├── README.md             ← englische Version
├── README.de.md          ← diese Datei
├── ROADMAP.md            ← Stufenplan mit Gates und Kennzahlen
├── scheduled-task-prompt.md   ← Vorlage für den wöchentlichen Scheduled Task (endet bei der Vorschau)
├── gate-evaluation-prompt.md  ← Vorlage für die einmalige Gate-Auswertung nach vier Wochen
├── CHANGELOG.md
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE
└── .github/
    └── repo-meta.yml     ← Repository-Metadaten
```

## Verwandte Skills

Der Skill ist einer von dreien, die dasselbe Idea Cockpit teilen: `idea-cockpit` erfasst und triagiert, `fragment-triage` sammelt verstreute Fragmente ein, `idea-serendipity` kombiniert Einträge frei und ohne Seed. Alle drei schreiben ausschliesslich Funken — das Weekly ist die Qualitätskontrolle.

## Roadmap

Die Einführung erfolgt in drei Stufen mit Gates — ein vierwöchiges Messfenster mit einem Scheduled Task pro Woche, der bei der Vorschau endet, und einem einmaligen Auswertungs-Task für das Gate; derselbe Lauf als Regelbetrieb; tiefere Infrastruktur nur bei einer gemessenen Lücke. Stufen, Gates, Kennzahlen und Risiken stehen in [ROADMAP.md](ROADMAP.md).

## Automatisierter Betrieb

Zwei Prompt-Vorlagen machen aus dem Skill einen unbeaufsichtigten Wochenlauf, ohne die Kontrolle über das Schreiben abzugeben:

- [`scheduled-task-prompt.md`](scheduled-task-prompt.md) — der wöchentliche Lauf. Er wählt nach fester Regel einen Seed, führt die Schritte 0–4 des Skills aus und **endet immer bei der Vorschau**; Funken werden erst nach Freigabe im Run-Chat geschrieben, danach folgt eine Audit-Zeile am Registry-Eintrag des Skills.
- [`gate-evaluation-prompt.md`](gate-evaluation-prompt.md) — die einmalige Auswertung nach vier Wochen. Sie zählt Beförderungen, Rauschen und Freigabe-Latenz gegen Vergleichsgruppen und empfiehlt eines von drei Ergebnissen (bestanden, teilweise, verfehlt). Sie ändert nichts im Cockpit.

Beide sind ID-freie Vorlagen: die Platzhalter `{{…}}` mit den Werten aus der eigenen `config.yml` ersetzen, dann die Scheduled Tasks in Claude anlegen (wöchentlich an einem anderen Tag als andere Agenten-Läufe, die dasselbe Weekly bedienen; die Auswertung am Tag nach dem letzten Weekly des Fensters).

## Changelog

Siehe [CHANGELOG.md](CHANGELOG.md)

## Mitwirken

Beiträge sind willkommen — siehe [CONTRIBUTING.md](CONTRIBUTING.md).

## Sicherheit

Schwachstellen bitte wie in [SECURITY.md](SECURITY.md) beschrieben melden.

## Lizenz

MIT-Lizenz — siehe [LICENSE](LICENSE)

## Autor

Hayal Özkan · [malkreide](https://github.com/malkreide)
