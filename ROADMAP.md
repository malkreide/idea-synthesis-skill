# Roadmap – Implementierung in Stufen

> **English summary.** This skill is rolled out in three gated stages: (1) a four-week measurement window with one scheduled synthesis run per week (Thursday) that always stops at the preview, approval in the run chat, and a one-shot evaluation task that scores the gate; (2) the same weekly run continued as regular operation, with the prompt templates released in this repo; (3) deeper infrastructure (sub-agents per database, local index) only if measured retrieval gaps or volume justify it. Each stage has a single exit metric: how many synthesis sparks the weekly review promotes. The full plan below is in German (Swiss spelling).

Stand: 05.10.2026 (Automatisierung ab 08.10.2026) · Owner: Hayal Özkan · Rhythmus: Weekly-Ritual (Montag) als einziger Takt- und Qualitätsgeber

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
| 1 | Messfenster: Referenzlauf + vier Scheduled-Task-Läufe, Gate-Auswertung per Task | 04.10.–03.11.2026 (KW 41–45) | in Betrieb | ≥ 2 Beförderungen aus ≥ 3 Läufen, Rauschen ≤ 50 % |
| 2 | Regelbetrieb des Scheduled Task «Synthese-Lauf», Weekly-Integration, Release v1.2.0 | ab 05.11.2026, Messfenster KW 46–49 | wartet auf Gate 1 | ≥ 2 Beförderungen in 4 Wochen, Freigabe-Latenz ≤ 7 Tage |
| 3 | Tiefere Infrastruktur (Subagenten, lokaler Index) | Entscheid Januar 2027 | bedingt | nur bei gemessener Recall- oder Budget-Lücke |

**Was am 05.10.2026 geändert wurde:** Die vier Läufe der Stufe 1 werden nicht manuell ausgelöst, sondern vom Scheduled Task «Synthese-Lauf» (Donnerstag 06:10 Europe/Zurich), der ursprünglich erst für Stufe 2 vorgesehen war. Der Task endet bei der Vorschau und schreibt nie selbst; die Freigabe erfolgt im Run-Chat. Die Gate-1-Auswertung übernimmt ein einmaliger Task am 03.11.2026. Stufe 2 schrumpft damit auf Regelbetrieb, Weekly-Integration und Release.

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

## Stufe 1 – Messfenster mit automatisierten Läufen (in Betrieb)

**Ziel:** Nachweisen, dass seed-basierte Synthese Funken erzeugt, die das Weekly befördert – bevor der Lauf zum Regelbetrieb wird.

**Eintrittskriterium:** Stufe 0 abgeschlossen. ✅

### Vorgehen

1. **Ein Lauf pro Woche, ausgelöst vom Scheduled Task «Synthese-Lauf»** (Donnerstag 06:10 Europe/Zurich, Push-Benachrichtigung). Der Task startet ohne Kontext, liest den Skill, wählt genau einen Seed nach der Lauf-Regel unten, führt die Schritte 0–4 des Skills aus und **endet bei der Vorschau**. Er schreibt nie selbst. Prompt-Vorlage: [`scheduled-task-prompt.md`](scheduled-task-prompt.md).
2. **Freigabe im Run-Chat** («schreib 1 und 3», «Seed anreichern», «alle», «nichts»). Erst dann werden Funken angelegt (zuerst die Funken, dann die Seed-Anreicherung, dann die Audit-Zeile). Pro Lauf höchstens drei neue Funken plus eine Seed-Anreicherung. Ohne Freigabe verfällt die Vorschau – nichts wird nachgeholt.
3. **Lauf-Protokoll:** Nach jeder Freigabe hängt der Task eine Audit-Zeile an den Registry-Eintrag `idea-synthesis` (Lauf-Nr., Seed, Retrieval-Zahlen, Aufrufe, geschriebene Einträge, Befund). Diese Zeilen sind die Datenbasis der Gate-Auswertung.
4. **Weekly:** Synthese-Funken wie alle anderen triagieren. Bei Beförderung den «Möglichen nächsten Schritt» aus dem Provenienz-Block übernehmen. Agenten-Bilanz im Bericht.
5. **Gate-Auswertung per einmaligem Scheduled Task** am Dienstag 03.11.2026 06:10 – einen Tag nach dem Weekly vom 02.11., damit dessen Triage-Entscheide bereits in Notion stehen. Prompt-Vorlage: [`gate-evaluation-prompt.md`](gate-evaluation-prompt.md).

### Lauf-Regel (Seed-Auswahl, je Lauf fest vorgegeben)

Die vier Läufe testen bewusst verschiedene Seed-Typen, damit das Gate nicht an einem einzigen Pfad hängt. Ab Lauf 5 gilt die Standardregel aus Stufe 2.

| Lauf | Datum | Seed-Regel | Prüft |
|---|---|---|---|
| 1 | Do 08.10.2026 | 🧩 Konzept aus einem Bereich ausserhalb des Hauptbereichs, höchste Score-Summe | Bereichs-Bias der Operatoren |
| 2 | Do 15.10.2026 | 🧩 Konzept aus dem Hauptbereich mit den meisten Relationen | Reichhaltiger Seed, Tool-/Paper-Kombinationen |
| 3 | Do 22.10.2026 | ❄️ Eingefroren mit ≥ 2 Relationen, ältester zuerst | Reaktivierung von totem Material |
| 4 | Do 29.10.2026 | 🌱 Funke ohne Tags mit ≥ 2 Relationen, ältester zuerst | Funke → Konzept als Hauptanwendungsfall |

Ausgeschlossen sind in allen Läufen: Einträge mit Tag «Synthese» oder «Serendipity» (noch nicht triagiert), Einträge mit einem Abschnitt «## Synthese» (schon gelaufen) und der Seed des Referenzlaufs.

### Arbeitspakete

| Nr. | Paket | Aufwand | Fällig |
|---|---|---|---|
| 1.1 | Referenzlauf Seed «Maieutic» – 4 Funken, Seed angereichert | erledigt | 04.10.2026 |
| 1.2 | Scheduled Task «Synthese-Lauf» angelegt (Donnerstag 06:10 Europe/Zurich, Push, Lauf-Regel für Lauf 1–4 im Prompt) | erledigt | 05.10.2026 |
| 1.3 | Einmaliger Scheduled Task «Synthese Gate-1-Auswertung» angelegt (03.11.2026 06:10) | erledigt | 05.10.2026 |
| 1.4 | Lauf 1–4: Vorschau sichten, im Run-Chat freigeben; Freigabe-Latenz beobachten | 15 Min/Woche | 08./15./22./29.10.2026 |
| 1.5 | Weekly vom 02.11.2026: letzte Triage vor der Auswertung | 15 Min | 02.11.2026 |
| 1.6 | Gate-Bericht lesen (Kommentar am Registry-Eintrag), Entscheid nach Austrittskriterium | 15 Min | 03.11.2026 |

### Messgrössen

| Kennzahl | Quelle | Zielwert nach 4 Wochen |
|---|---|---|
| Beförderungen 🌱 → 🧩 (primär) | View «🔗 Synthese», Reifegrad der Synthese-Funken | ≥ 2 |
| Seed-Anreicherungen, deren Nächster Schritt danach umgesetzt wurde | Seed-Einträge, Abschnitt «Synthese <Datum>» | ≥ 1 |
| Eingefroren oder gelöscht (Rauschen) | View «🔗 Synthese» | ≤ 50 % der Funken |
| Aufrufe pro Lauf | Audit-Zeilen | ≤ 20 |
| Freigabe-Latenz (Vorschau → Entscheid) | Audit-Zeilen vs. Lauf-Datum | Median ≤ 7 Tage |
| Läufe ohne Freigabe (verfallen) | Audit-Zeilen | ≤ 1 von 4 |

### Austrittskriterium (Gate 1 → 2)

Die Gate-Auswertung liefert eine von drei Empfehlungen:

- **bestanden:** ≥ 2 Beförderungen aus ≥ 3 Läufen **und** Rauschen ≤ 50 % → Stufe 2: Task läuft weiter, Weekly-Integration, Release v1.2.0.
- **teilweise:** 1 Beförderung → Operatoren und Retrieval-Budget überarbeiten (v1.2.x), Messfenster um 4 Wochen verlängern, Task läuft weiter.
- **verfehlt:** 0 Beförderungen → Task pausieren. Erst prüfen, ob das Weekly überhaupt stattfand (Prozessproblem), dann ob die Themen-Verdrahtung reicht (Datenproblem), zuletzt die Operatoren (Methodenproblem). In dieser Reihenfolge.

Der Task entscheidet nichts selbst – er misst und empfiehlt. Der Entscheid bleibt beim Owner.

---

## Stufe 2 – Regelbetrieb des Scheduled Task (wartet auf Gate 1)

**Ziel:** Den in Stufe 1 bewährten Lauf ohne weitere Aufmerksamkeit laufen lassen, den Freigabe-Pfad ins Weekly einbauen und den Stand im Repo veröffentlichen. Es wird nichts Neues gebaut – der Task läuft nach Lauf 4 einfach weiter, solange er nicht pausiert wird.

**Eintrittskriterium:** Gate 1 bestanden (Empfehlung «bestanden» im Gate-Bericht, bestätigt vom Owner).

### Design (gilt ab Lauf 5)

| Element | Festlegung |
|---|---|
| Takt | unverändert wöchentlich, Donnerstag 06:10 Europe/Zurich – bewusst nicht am Montag, damit das Weekly nicht zwei Agenten-Vorschauen gleichzeitig verdauen muss (Serendipity läuft Montag 06:10) |
| Seed-Auswahl (Standardregel) | genau **ein** Seed pro Lauf, Reihenfolge: (1) 🧩 Konzept ohne Synthese-Abschnitt, höchste Score-Summe; (2) 🌱 Funke mit ≥ 2 Relationen ohne Synthese-Abschnitt, ältester zuerst; ausgeschlossen: ❄️ Eingefroren, Einträge mit Synthese-Abschnitt in den letzten 60 Tagen, Synthese- und Serendipity-Funken (noch nicht triagiert) |
| Umfang | Retrieval-Budget wie im Skill; maximal 3 Kandidaten mit Ziel «neuer Eintrag» |
| Output | Vorschau im Run-Chat plus Push-Benachrichtigung; nichts wird angelegt, keine Relation gesetzt |
| Freigabe | im Run-Chat oder im nächsten Weekly; ohne Freigabe verfällt die Vorschau |
| Schreibschutz | liegt im Prompt (Lauf endet bei Vorschau), nicht in der Berechtigung – wie bei Serendipity |
| Prompt | Vorlage `scheduled-task-prompt.md` im Repo, ID-frei; die produktive Fassung lebt im Task des Accounts |
| Pflege | Prompt-Änderungen über `update_trigger`, nie löschen und neu anlegen (Run-Historie) |

### Arbeitspakete

| Nr. | Paket | Aufwand | Fällig |
|---|---|---|---|
| 2.1 | Gate-Bericht und Audit-Zeilen der Läufe 1–4 in den Prompt einarbeiten (Seed-Regel, Budget, Abbruchregeln), per `update_trigger` | 1 Abend | KW 45 |
| 2.2 | Freigabe-Pfad im Weekly verankern: `idea-cockpit` v2.1 – Schritt «Agenten-Vorschauen sichten» vor der Funken-Triage | 1 Std | KW 46 |
| 2.3 | Release v1.2.0 des Repos: überarbeitete Prompt-Vorlagen, Roadmap-Stand, Changelog | 30 Min | KW 46 |
| 2.4 | Vier Wochen Messfenster (KW 46–49), Auswertung im Weekly vom 07.12.2026 – gleicher Mess-Prompt wie Gate 1, Fenster ab 05.11. | 15 Min/Woche | 07.12.2026 |

### Messgrössen (zusätzlich zu Stufe 1)

| Kennzahl | Zielwert |
|---|---|
| Beförderungen aus Läufen 5–8 | ≥ 2 |
| Freigabe-Latenz (Vorschau → Entscheid) | Median ≤ 7 Tage |
| Verfallene Vorschauen (keine Freigabe) | ≤ 1 von 4 |
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
| Registry-Eintrag `idea-synthesis` nach jedem Lauf um eine Audit-Zeile ergänzen (macht der Task nach der Freigabe) | pro Lauf | Claude Skills Registry |
| Scheduled Task pausieren, wenn zwei Weeklys ausfallen oder zwei Vorschauen ohne Freigabe bleiben – Vorschauen ohne Triage sind Rauschen | bei Bedarf | Task-Einstellungen |
| Lauf-Befunde (dünne DBs, fehlende Relationen) sammeln und beim Stufenwechsel in den Skill einarbeiten | pro Lauf | `CHANGELOG.md`, Skill-Changelog |
| Repo-Hygiene nach `github-repo`-Skill: Validator vor jedem Push, Release pro Stufenwechsel | pro Stufe | dieses Repo |
| Serendipity und Synthese **gemeinsam** auswerten: welche Quelle liefert die beförderten Funken? Daraus folgt, welcher Agent Aufmerksamkeit verdient | monatlich | Weekly-Bericht |

---

## Versionierung des Repos entlang der Stufen

| Version | Inhalt | Auslöser |
|---|---|---|
| 1.1.0 | Skill nach Referenzlauf, `config.yml`-Muster | Stufe 0/1 (erledigt) |
| 1.1.x | Roadmap, Prompt-Vorlagen für Lauf und Gate-Auswertung, Korrekturen aus den Läufen 1–4 | laufend |
| 1.2.0 | Prompt-Vorlagen nach Gate-Bericht überarbeitet, Weekly-Integration, Roadmap-Stand | Gate 1 → 2 |
| 2.0.0 | Retrieval über neues Werkzeug (Index oder Subagenten), Fallback Notion | Gate 2 → 3 |

---

## Risiken und Gegenmassnahmen

| Risiko | Wirkung | Gegenmassnahme |
|---|---|---|
| Weekly findet nicht statt | Funken stauen sich, Messung wird wertlos | Agenten-Läufe pausieren, wenn zwei Weeklys ausfallen; Vorschauen ohne Freigabe verfallen |
| Vorschau wird nicht freigegeben (Run-Chat nicht geöffnet) | Lauf ohne Wirkung, Gate misst zu wenig | Push-Benachrichtigung; Freigabe-Latenz ist Messgrösse; ab zwei verfallenen Vorschauen Task pausieren statt weiterlaufen lassen |
| Task schreibt trotz Schreibschutz | ungeprüfte Einträge im Cockpit | Schreibschutz steht als erste harte Regel im Prompt; Audit-Zeilen machen jeden Schreibvorgang sichtbar; Gate-Auswertung prüft Einträge ohne Freigabe |
| Mehr Funken als Triage-Kapazität | Rauschen, Cockpit verliert Glaubwürdigkeit | harte Quote: 3 neue Funken pro Lauf, ein Lauf pro Woche |
| Notion-Schema ändert sich | Fehlschreibungen | Schema-Check vor jedem Schreibvorgang; Lauf bricht ab und meldet |
| Workspace-IDs geraten ins Repo | Struktur des Workspace öffentlich | `config.yml` gitignored, Secrets-Check vor jedem Push, Push Protection aktiv |
| Infrastruktur-Reiz (Stufe 3 zu früh) | Aufwand ohne gemessenen Nutzen | Gate 2 → 3 verlangt eine benannte, gemessene Lücke |
| Operatoren produzieren bereichs-einseitige Kandidaten | Verzerrung Richtung Bildungskontext | Lauf 1.2 mit Seed aus anderem Bereich; Bereichs-Verteilung der beförderten Funken im Monatsbericht |

---

## Nächste konkrete Schritte

1. Do 08.10.2026: erste Vorschau des Scheduled Task sichten und im Run-Chat freigeben (Lauf 1).
2. Montags: Synthese-Funken im Weekly triagieren; Weekly vom 02.11.2026 ist die letzte Triage vor der Auswertung.
3. Di 03.11.2026: Gate-Bericht am Registry-Eintrag lesen, Entscheid nach Austrittskriterium, danach Arbeitspaket 2.1 oder Pause.
