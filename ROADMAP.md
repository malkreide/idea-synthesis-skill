# Roadmap – Implementierung in Stufen

> **English summary.** This skill is rolled out in three gated stages: (1) manual, seed-based synthesis runs with a four-week measurement window; (2) a weekly scheduled run that prepares one synthesis preview per week and never writes without approval; (3) deeper infrastructure (sub-agents per database, local index) only if measured retrieval gaps or volume justify it. Each stage has a single exit metric: how many synthesis sparks the weekly review promotes. The full plan below is in German (Swiss spelling).

Stand: 05.10.2026 · Owner: Hayal Özkan · Rhythmus: Weekly-Ritual (Montag) als einziger Takt- und Qualitätsgeber

---

## Grundsätze, die für alle Stufen gelten

1. **Pull vor Push.** Der Skill startet von einem Seed. Freie Ideengenerierung ist Sache von `idea-serendipity`; beide Agenten bleiben getrennt und messen sich getrennt (Tags «Synthese» und «Serendipity»).
2. **Das Weekly ist der Engpass, nicht die Ideenmenge.** Jede Stufe darf höchstens so viele Funken erzeugen, wie ein 15-Minuten-Weekly triagieren kann. Mehr Output ist kein Fortschritt.
3. **Eine Kennzahl entscheidet über den Stufenwechsel:** Anzahl Synthese-Funken, die im Weekly von 🌱 Funke auf 🧩 Konzept (oder höher) befördert werden – gemessen über die View «🔗 Synthese», im Vergleich zu Funken ohne Tag im selben Zeitraum.
4. **Keine Stufe ohne Gate.** Jede Stufe hat ein Eintritts- und ein Austrittskriterium. Wird das Austrittskriterium verfehlt, wird die Stufe überarbeitet oder pausiert – nicht übersprungen.
5. **Infrastruktur erst, wenn sie nachweislich fehlt.** Lokaler Index, Agent SDK oder Edge-Betrieb rechtfertigen sich nur durch gemessene Lücken (Recall, Budget, Volumen), nicht durch Interesse an der Technik.
6. **Schema-Treue.** Vor jedem Schreibvorgang wird das Notion-Schema geprüft; IDs leben in `config.yml`, nie im Skill-Text.

---

## Übersicht

| Stufe | Inhalt | Zeitraum | Status | Austrittskriterium |
|---|---|---|---|---|
| 0 | Datenmodell und Infrastruktur | 04.–05.10.2026 | erledigt | Tags, View, Themen-Backfill, Registry-Einträge vorhanden |
| 1 | Manueller Skill-Betrieb, Referenzlauf, Messfenster | 04.10.–02.11.2026 (KW 41–45) | in Betrieb | ≥ 2 Beförderungen aus ≥ 3 Läufen |
| 2 | Wöchentlicher Scheduled Task «Synthese-Lauf» | Aufbau KW 45–46, Messfenster KW 47–50 | geplant | ≥ 2 Beförderungen aus automatisierten Läufen, Freigabe-Latenz ≤ 7 Tage |
| 3 | Tiefere Infrastruktur (Subagenten, lokaler Index) | Entscheid Januar 2027 | bedingt | nur bei gemessener Recall- oder Budget-Lücke |

Querschnittsaufgaben laufen parallel (Abschnitt «Querschnitt»).

---

## Stufe 0 – Datenmodell und Infrastruktur (erledigt)

**Ziel:** Die Wissensbasis so verdrahten, dass Kombination über abstrahierte Themen funktioniert, nicht über Titel-Ähnlichkeit.

| Arbeitspaket | Ergebnis | Datum |
|---|---|---|
| Tags-Optionen «Synthese» und «Serendipity» im Idea Cockpit | vorhanden | 04.10.2026 |
| Mess-View «🔗 Synthese» (Filter Tags = Synthese, Erstellt absteigend) | vorhanden | 05.10.2026 |
| Themen-Backfill im Cockpit: 201 Einträge über die Themen ihrer verknüpften Papers/Tools (Mehrheitsregel, max. 3 Themen) | 258 von 496 Einträgen mit Themen-Relation (vorher 54) | 05.10.2026 |
| Skill v1.1 nach Referenzlauf; Repo mit `config.yml`-Muster; Registry-Einträge für die Ideen-Agenten-Familie | vorhanden | 05.10.2026 |

**Offen geblieben, bewusst:** 238 Cockpit-Einträge ohne Tools oder Papers haben keine Ableitungsbasis für Themen. Sie bekommen Themen, wenn sie durch eine Synthese laufen oder im Weekly triagiert werden – kein eigener Batch.

---

## Stufe 1 – Manueller Betrieb und Messfenster (in Betrieb)

**Ziel:** Nachweisen, dass seed-basierte Synthese Funken erzeugt, die das Weekly befördert – bevor irgendetwas automatisiert wird.

**Eintrittskriterium:** Stufe 0 abgeschlossen. ✅

### Vorgehen

1. **Ein Lauf pro Woche, manuell ausgelöst**, idealerweise am Tag vor dem Weekly. Seed-Auswahl nach Priorität: zuerst 🧩 Konzept-Einträge mit vielen Relationen, dann Funken mit hohem Priority Score, dann ein Problemstatement ohne Eintrag (Fall B), damit beide Eintrittspfade getestet sind.
2. **Freigabe im selben Chat.** Pro Lauf höchstens vier neue Funken plus eine Seed-Anreicherung.
3. **Lauf-Protokoll** in einer Zeile am Ende jedes Laufs (Kurzbericht, Schritt 6 des Skills): Retrieval-Zahlen, Aufrufe, Befund zur Wissensbasis. Die Zeile wird in den Registry-Eintrag (`Audit`) übernommen.
4. **Weekly:** Synthese-Funken wie alle anderen triagieren. Bei Beförderung den «Möglichen nächsten Schritt» aus dem Provenienz-Block übernehmen. Agenten-Bilanz im Bericht.

### Arbeitspakete

| Nr. | Paket | Aufwand | Fällig |
|---|---|---|---|
| 1.1 | Referenzlauf Seed «Maieutic» – 4 Funken, Seed angereichert | erledigt | 04.10.2026 |
| 1.2 | Zweiter Lauf mit einem Seed aus einem anderen Bereich (sormena oder KI-Fachgruppe), um Bereichs-Bias der Operatoren zu prüfen | 1 Abend | KW 42 |
| 1.3 | Dritter Lauf über Fall B (Problemstatement ohne Eintrag) | 1 Abend | KW 43 |
| 1.4 | Vierter Lauf mit einem ❄️ eingefrorenen Seed – prüft, ob der Skill totes Material reaktivieren kann | 1 Abend | KW 44 |
| 1.5 | Auswertung im Weekly vom 02.11.2026 (KW 45): View «🔗 Synthese» nach Reifegrad, Vergleich mit Funken ohne Tag seit 04.10. | 15 Min | 02.11.2026 |

### Messgrössen

| Kennzahl | Quelle | Zielwert nach 4 Wochen |
|---|---|---|
| Beförderungen 🌱 → 🧩 (primär) | View «🔗 Synthese» | ≥ 2 |
| Seed-Anreicherungen, deren Nächster Schritt danach umgesetzt wurde | Seed-Einträge, Abschnitt «Synthese <Datum>» | ≥ 1 |
| Eingefroren oder gelöscht (Rauschen) | View «🔗 Synthese» | ≤ 50 % der Funken |
| Aufrufe pro Lauf | Kurzbericht | ≤ 20 |

### Austrittskriterium (Gate 1 → 2)

- ≥ 2 Beförderungen aus ≥ 3 Läufen **und** Rauschen ≤ 50 % → Stufe 2 starten.
- 1 Beförderung → Operatoren und Retrieval-Budget überarbeiten (v1.2), Messfenster um 4 Wochen verlängern.
- 0 Beförderungen → Stufe 1 pausieren. Erst prüfen, ob das Weekly überhaupt stattfand (Prozessproblem), dann ob die Themen-Verdrahtung reicht (Datenproblem), zuletzt die Operatoren (Methodenproblem). In dieser Reihenfolge.

---

## Stufe 2 – Wöchentlicher Scheduled Task «Synthese-Lauf» (geplant)

**Ziel:** Den Lauf ohne manuellen Anstoss stattfinden lassen, ohne die Kontrolle über das Schreiben abzugeben. Vorbild ist der bestehende Scheduled Task «Serendipity-Lauf»: der Lauf endet immer bei der Vorschau, schreibt nie selbst, die Freigabe erfolgt im Chat oder im Weekly.

**Eintrittskriterium:** Gate 1 bestanden.

### Design

| Element | Festlegung |
|---|---|
| Takt | wöchentlich, Donnerstag 06:10 Europe/Zurich – bewusst nicht am Montag, damit das Weekly nicht zwei Agenten-Vorschauen gleichzeitig verdauen muss (Serendipity läuft Montag 06:10) |
| Seed-Auswahl (automatisch) | genau **ein** Seed pro Lauf, Reihenfolge: (1) 🧩 Konzept ohne Synthese-Abschnitt, höchster Priority Score; (2) 🌱 Funke mit ≥ 2 Relationen ohne Synthese-Abschnitt, ältester zuerst; ausgeschlossen: ❄️ Eingefroren, Einträge mit Synthese-Abschnitt in den letzten 60 Tagen, Serendipity-Funken (die sind noch nicht triagiert) |
| Umfang | Retrieval-Budget wie im Skill; maximal 3 Kandidaten mit Ziel «neuer Eintrag» (nicht 4 – ein Lauf weniger Entscheidungsdruck fürs Weekly) |
| Output | Vorschau als Kommentar **am Seed-Eintrag** in Notion (`notion-create-comment`) plus Push-Benachrichtigung; nichts wird angelegt, keine Relation gesetzt |
| Freigabe | im Run-Chat («schreib 1 und 3», «Seed anreichern») oder im nächsten Weekly; ohne Freigabe verfällt die Vorschau nach 14 Tagen |
| Schreibschutz | liegt im Prompt (Lauf endet bei Vorschau), nicht in der Berechtigung – wie bei Serendipity. Zusätzlich: der Task bekommt nur Leserechte, falls die Plattform das erlaubt |
| Prompt | eigene Datei `scheduled-task-prompt.md` im Repo, ID-frei, liest `config.yml`; startet ohne Kontext und muss vollständig sein |
| Pflege | Prompt-Änderungen über `update_trigger`, nie löschen und neu anlegen (Run-Historie) |

### Arbeitspakete

| Nr. | Paket | Aufwand | Fällig |
|---|---|---|---|
| 2.1 | `scheduled-task-prompt.md` schreiben: Seed-Auswahl per SQL, Lauf bis Vorschau, Kommentar-Format, Abbruchregeln (Schema weicht ab, Seed-Pool leer, Budget überschritten) | 1 Abend | KW 45 |
| 2.2 | Trockenlauf des Prompts im Chat gegen drei verschiedene Seeds – prüft Seed-Auswahl und Kommentar-Format, schreibt nichts | 1 Abend | KW 45 |
| 2.3 | Scheduled Task anlegen (Donnerstag 06:10 Europe/Zurich, Push), Trigger-ID in `.github/repo-meta.yml` und Registry eintragen | 30 Min | KW 46 |
| 2.4 | Freigabe-Pfad im Weekly verankern: `idea-cockpit` v2.1 – Schritt «Agenten-Vorschauen sichten» vor der Funken-Triage | 1 Std | KW 46 |
| 2.5 | Vier Wochen Messfenster (KW 47–50), Auswertung im Weekly vom 15.12.2026 | 15 Min/Woche | 15.12.2026 |
| 2.6 | Release v1.2.0 des Repos: Prompt-Datei, Roadmap-Stand, Changelog | 30 Min | nach 2.3 |

### Messgrössen (zusätzlich zu Stufe 1)

| Kennzahl | Zielwert |
|---|---|
| Beförderungen aus automatisierten Läufen in 4 Wochen | ≥ 2 |
| Freigabe-Latenz (Vorschau → Entscheid) | Median ≤ 7 Tage |
| Verfallene Vorschauen (keine Entscheidung in 14 Tagen) | ≤ 1 von 4 |
| Seed-Auswahl vom User als «falscher Seed» verworfen | ≤ 1 von 4 |

### Austrittskriterium (Gate 2 → 3)

Stufe 3 wird **nur** geprüft, wenn mindestens eine dieser Lücken gemessen ist:

- **Recall-Lücke:** In ≥ 2 Läufen hat der User Einträge benannt, die zum Seed passen, die `notion-ai-search` aber nicht fand.
- **Budget-Lücke:** Läufe überschreiten regelmässig 20 Aufrufe oder brechen am Kontext ab.
- **Volumen-Lücke:** Es besteht ein belegter Bedarf für mehr als einen Lauf pro Woche (Weekly schafft die Triage und fordert mehr).

Ist keine Lücke messbar, bleibt Stufe 2 der Endzustand – das ist ein gutes Ergebnis, kein Scheitern.

---

## Stufe 3 – Tiefere Infrastruktur (bedingt)

**Ziel:** Die in Stufe 2 gemessene Lücke schliessen – und nur diese.

**Eintrittskriterium:** Gate 2 → 3 mit benannter Lücke.

### Optionen, je nach Lücke

| Lücke | Massnahme | Aufwand | Abhängigkeit |
|---|---|---|---|
| Recall | Lokaler semantischer Index über die Summaries der Source Library und die Descriptions der AI-Tools (Embeddings, Vektor-Store), als MCP-Tool «knowledge-index» neben Notion; Themen-Hub bleibt die Brücke | 2–3 Wochenenden | KOMKI-Stack; kann auf dem Pi 5 laufen |
| Recall | Feld **Kern-Mechanismus** (ein Satz, domänenfrei) in AI-Tools und Source Library, per Batch gefüllt und stichprobenweise geprüft – macht die Operatoren A und B präziser, ohne neue Infrastruktur | 1 Wochenende Batch + laufende Pflege | nur Notion |
| Budget | Agent SDK mit einem Subagenten pro Datenbank (Themen, Papers, Tools, People), Orchestrator fasst zusammen; Kontext pro Subagent klein | 2 Wochenenden | Python, API-Key, Betrieb lokal oder Pi |
| Volumen | Notion-eigene Agenten für Seed-Auswahl und Vorschau, Claude nur für die Kombination | 1 Wochenende | Notion-Agent-Funktionen des Workspace |

**Reihenfolge, falls mehrere Lücken:** zuerst Kern-Mechanismus-Feld (billig, reversibel), dann lokaler Index, zuletzt Agent SDK. Jede Massnahme wird einzeln vier Wochen gemessen, bevor die nächste folgt.

### Arbeitspakete (nur als Skizze, wird bei Eintritt konkretisiert)

| Nr. | Paket |
|---|---|
| 3.1 | Evaluationsnotiz (2 Seiten): welche Lücke, welche Massnahme, welcher Messwert soll sich wie ändern – **vor** dem Bauen |
| 3.2 | Bau der gewählten Massnahme in einem eigenen Repo (`knowledge-index-mcp` oder `idea-synthesis-agent`), dieses Repo bleibt der Skill |
| 3.3 | Skill v2.0: Retrieval-Schritt 1 nutzt das neue Werkzeug, Fallback bleibt Notion |
| 3.4 | Vier Wochen Messfenster, Entscheid: behalten oder zurückbauen |

---

## Querschnitt – läuft parallel zu allen Stufen

| Aufgabe | Takt | Wo |
|---|---|---|
| Themen-Relation bei jeder Triage setzen (Einträge ohne Quellen bekommen so nach und nach Themen) | Weekly | `idea-cockpit` |
| 3-Aktiv-Limit wieder einhalten (Stand 05.10.2026 überschritten) – abschliessen oder einfrieren | nächstes Weekly | `idea-cockpit` |
| Registry-Eintrag `idea-synthesis` nach jedem Lauf um eine Audit-Zeile ergänzen | pro Lauf | Claude Skills Registry |
| Lauf-Befunde (dünne DBs, fehlende Relationen) sammeln und beim Stufenwechsel in den Skill einarbeiten | pro Lauf | `CHANGELOG.md`, Skill-Changelog |
| Repo-Hygiene nach `github-repo`-Skill: Validator vor jedem Push, Release pro Stufenwechsel | pro Stufe | dieses Repo |
| Serendipity und Synthese **gemeinsam** auswerten: welche Quelle liefert die beförderten Funken? Daraus folgt, welcher Agent Aufmerksamkeit verdient | monatlich | Weekly-Bericht |

---

## Versionierung des Repos entlang der Stufen

| Version | Inhalt | Auslöser |
|---|---|---|
| 1.1.0 | Skill nach Referenzlauf, `config.yml`-Muster | Stufe 0/1 (erledigt) |
| 1.1.x | Korrekturen aus den Läufen 1.2–1.4 | laufend |
| 1.2.0 | `scheduled-task-prompt.md`, Weekly-Integration, Roadmap-Stand | Gate 1 → 2 |
| 2.0.0 | Retrieval über neues Werkzeug (Index oder Subagenten), Fallback Notion | Gate 2 → 3 |

---

## Risiken und Gegenmassnahmen

| Risiko | Wirkung | Gegenmassnahme |
|---|---|---|
| Weekly findet nicht statt | Funken stauen sich, Messung wird wertlos | Agenten-Läufe pausieren, wenn zwei Weeklys ausfallen; Vorschauen verfallen nach 14 Tagen |
| Mehr Funken als Triage-Kapazität | Rauschen, Cockpit verliert Glaubwürdigkeit | harte Quoten (Stufe 1: 4, Stufe 2: 3 pro Lauf, ein Lauf pro Woche) |
| Notion-Schema ändert sich | Fehlschreibungen | Schema-Check vor jedem Schreibvorgang; Lauf bricht ab und meldet |
| Workspace-IDs geraten ins Repo | Struktur des Workspace öffentlich | `config.yml` gitignored, Secrets-Check vor jedem Push, Push Protection aktiv |
| Infrastruktur-Reiz (Stufe 3 zu früh) | Aufwand ohne gemessenen Nutzen | Gate 2 → 3 verlangt eine benannte, gemessene Lücke |
| Operatoren produzieren bereichs-einseitige Kandidaten | Verzerrung Richtung Bildungskontext | Lauf 1.2 mit Seed aus anderem Bereich; Bereichs-Verteilung der beförderten Funken im Monatsbericht |

---

## Nächste konkrete Schritte

1. Lauf 1.2 (Seed aus sormena oder KI-Fachgruppe) in KW 42.
2. Weekly am 02.11.2026: Gate-1-Auswertung über die View «🔗 Synthese».
3. Bei bestandenem Gate: Arbeitspaket 2.1 – `scheduled-task-prompt.md` schreiben.
