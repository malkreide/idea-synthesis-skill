---
name: idea-synthesis
description: >
  Seed-basierte Weiterentwicklung von Ideen aus einer Notion-Wissensbasis (Idea Cockpit,
  Themen-Hub, AI-Tools, Source Library, Fragmente, People, Projekt-DBs).
  Verwende diesen Skill wenn der User (1) einen bestehenden Idea-Cockpit-Eintrag
  weiterentwickeln, schärfen oder anreichern will, (2) zu einem konkreten Problem
  Lösungskandidaten aus der eigenen Wissensbasis sucht, (3) fragt «welche Tools / Papers /
  Methoden passen zu Idee X», «was habe ich dazu schon», «kombiniere das mit …»,
  (4) eine Idee von 🌱 Funke zu 🧩 Konzept bringen will und dafür Substanz braucht, oder
  (5) Begriffe wie «Synthese», «Kombinator», «idea-synthesis», «weiterentwickeln»,
  «anreichern», «Verbindungen finden» verwendet. NICHT verwenden für reines Erfassen
  (→ idea-cockpit), Fragment-Triage (→ fragment-triage) oder freie Ideengenerierung
  ohne Seed (→ idea-serendipity).
---

# Idea Synthesis Skill (v1.1)

Pull, nicht push: Dieser Skill startet **immer von einem Seed** – einem bestehenden
Idea-Cockpit-Eintrag oder einem konkret formulierten Problem – und holt gezielt das aus
der Notion-Wissensbasis, was den Seed weiterbringt. Er generiert keine Ideen ins Blaue.
Jeder Vorschlag ist eine nachvollziehbare Kombination von Notion-Einträgen mit Provenienz.

Ergebnis: 3–5 bewertete Kandidaten, auf Wunsch zurückgeschrieben ins Idea Cockpit.

---

## Notion-Konfiguration

**Alle IDs stehen in `config.yml` im Skill-Verzeichnis** (Vorlage: `config.example.yml`).
Beim ersten Lauf einer Session diese Datei lesen; fehlt sie, den User bitten, sie aus
der Vorlage anzulegen. Nie IDs raten oder aus früheren Sessions übernehmen.

| Schlüssel in `config.yml` | Datenbank | Rolle in der Synthese |
|---|---|---|
| `notion.idea_cockpit` | 💡 Idea Cockpit | Seed-Quelle, Duplikat-Check, Ziel fürs Rückschreiben |
| `notion.themen_hub` | Themen-Hub | Brücke: von Thema zu Papers, Tools, Ideen, People |
| `notion.ai_tools` | AI-Tools | Was existiert bereits als Werkzeug |
| `notion.source_library` | 📚 Source Library | Mechanismen und Evidenz (gross – nie ganz laden) |
| `notion.fragmente` | Fragmente | Rohe Teilgedanken, die zum Seed passen (optional) |
| `notion.people` | People | Perspektiven, Stresstest (Kritiker:in, Policy) (optional) |
| `notion.optional.*` | MCP-Portfolio, Pi-Projekte, Skills-Registry, Impact Lab | nur wenn Topics, Constraints oder der Seed-Text es nahelegen |

Vor dem ersten Schreibvorgang einer Session das Schema der Ziel-DB mit `notion-fetch`
(`collection://<id>`) prüfen – Feldnamen nie raten.

### Erwartete Felder im Idea Cockpit
```
Titel                 TITLE
Idee                  TEXT      – Kurzbeschrieb
Reifegrad             SELECT    – 🌱 Funke | 🧩 Konzept | 🗺️ Plan | 🚀 Aktiv | ❄️ Eingefroren
Bereich               SELECT    – Optionen aus config.yml → cockpit.bereiche
Energie               SELECT    – ⚡ Hoch | 🔋 Mittel | 🪫 Niedrig
Nächster Schritt      TEXT      – Pflicht ab 🧩 Konzept, leer bei Funken
Topics                MULTI     – nur bestehende Optionen verwenden
Tags                  MULTI     – enthält die Option aus config.yml → cockpit.tag_synthese
Strategischer Wert    NUMBER    – 1–5
Machbarkeit           NUMBER    – 1–5
Lernwert              NUMBER    – 1–5
Evidenz               NUMBER    – 1–5
Priority Score        FORMULA   – read-only, aus den vier Zahlen
Relationen            AI-Tools · Papers & Ressourcen · Themen · Fragmente · Inspiriert von (People)
                      · optionale Projekt-DBs · Related Idea Cockpit · Parent item / Sub-item
```
Konvention: Synthese-Einträge erhalten das Icon aus `config.yml → cockpit.icon` (Standard 🔗).

### Werkzeuge
- **Lesen:**
  - `notion-fetch` – Seed, einzelne People-Seiten
  - `notion-ai-search` mit `data_source_url` – semantische Suche je DB; Anfrage als **Satz**, der das
    Seed-Problem beschreibt, nicht als Stichwortliste (beste Trefferquelle im Referenzlauf)
  - `notion-query-data-sources` (SQL) – **Batch-Laden** verknüpfter oder gefundener Einträge in
    einem Aufruf: `WHERE url LIKE '%<id1>' OR url LIKE '%<id2>' …`; Themen per SQL mit
    `json_array_length("Papers & Ressourcen")` etc. statt jede Themen-Seite zu fetchen
  - `notion-search` – Fallback
- **Schreiben:** `notion-create-pages` (alle Funken in einem Aufruf), `notion-update-page`
  (`update_properties` und `insert_content`), `notion-create-comment`,
  `notion-update-data-source` (nur Schema-Optionen), `notion-create-view` (nur Einrichtung)
- **Rückfragen:** Rückfrage-Widget der Umgebung (`AskUserQuestion` bzw. `ask_user_input_v0`),
  nie als Prosa-Liste. **Harte Grenzen: max. 4 Optionen pro Frage, max. 2 Fragen pro Aufruf,
  Header max. 12 Zeichen.**

---

## Workflow

### Schritt 0 – Seed klären (≤ 2 Rückfragen)

**Fall A – bestehender Cockpit-Eintrag** (User nennt Titel oder URL):
1. Eintrag per `notion-fetch` laden (liefert Properties und Seiteninhalt).
2. Lesen: Titel, Idee, Reifegrad, Bereich, Topics, Nächster Schritt, Seiteninhalt, gesetzte
   Relationen.
3. **Bereits verknüpfte Einträge** (Tools, Papers, Skills, Ideen) per SQL-Batch laden – ein Aufruf
   pro DB. Sie gelten als «bekannt»: sie liefern Mechanismen für die Operatoren, zählen aber
   nicht zum Retrieval-Budget und werden nicht als «neue Kandidaten» präsentiert.
4. Keine Rückfrage, wenn Idee + Bereich vorhanden sind. Sonst eine Frage: «Welches Problem
   soll die Idee lösen – in einem Satz?»

**Fall B – Problemstatement ohne Eintrag:**
1. Maximal zwei Fragen aufs Mal: **Bereich** (Optionen aus `config.yml`) und **harte
   Constraints** (Multi-Select, Optionen aus `config.yml → constraints`, max. 4; «keine» über
   «Other»).
2. Nicht fragen, ob ein Cockpit-Eintrag erstellt werden soll – das kommt in Schritt 4.

**Seed-Steckbrief** (intern, 6 Zeilen):
```
Problem (1 Satz):
Zielgruppe:
Constraints:
Bereich:
Schlüsselbegriffe (3–6):
Nahe Topics (aus der Topics-Liste des Cockpit):
```

### Schritt 1 – Retrieval mit Budget

Budget: **~40 neue Einträge** im Kontext und **~15 Tool-Aufrufe** bis zur Präsentation. Bereits
am Seed verknüpfte Einträge zählen nicht mit. Kein Volltext-Dump, keine ganze DB.
Referenzlauf 04.10.2026 (Seed «Maieutic», Bildungskontext): 45 Einträge, 14 Aufrufe, 5 Kandidaten.

1. **Themen-Hub zuerst** – Einstieg in dieser Reihenfolge:
   - (a) Themen-Relation des Seeds;
   - (b) falls leer – im Cockpit häufig – die **Themen-Relationen der am Seed verknüpften
     Papers und Tools** (Quellen-DBs sind meist besser mit dem Hub verdrahtet als das Cockpit);
   - (c) erst dann `notion-ai-search` im Themen-Hub.
   Bis zu 3 Themen per SQL laden (Name, Fokus, Beschreibung, Anzahl Relationen). Fokus
   «Kernthema» bevorzugen, «Archiv» ignorieren.
2. **Source Library** – max. 10 neue. `notion-ai-search` mit Problem-Satz, dann die besten
   5–6 per SQL-Batch laden (Title, Summary, Typ, Source Quality, Reading Status, Keep as
   Reference, Notes). Eine Zusammenfassung genügt als Mechanismus-Quelle; Reading Status
   «To Read» ist kein Ausschluss, sondern senkt den Evidenz-Score. Fehlt die Summary →
   «Mechanismus unklar – ungelesen», nie aus dem Titel erraten.
3. **AI-Tools** – max. 8 neue. `notion-ai-search`, dann SQL-Batch (ToolName, Description,
   Features, Hosting-Modus, Lizenz, Status, Limits, Alternatives). Status Using/Mastered
   zuerst, dann Exploring; Discontinued ausschliessen. Hosting-Modus gegen die Constraints
   prüfen (Edge-only → nur Edge/Self-hosted/On-Prem).
4. **Idea Cockpit** – max. 5 Nachbarn via `notion-ai-search`. Zweck: Duplikat-Check,
   «Related Idea Cockpit»-Kandidaten und fehlende Nachbar-Links des Seeds. Eingefrorene
   Einträge mitnehmen – oft liegt dort das fehlende Teil.
5. **Fragmente** – max. 5, ein Aufruf. Leer ist häufig und kostet nichts weiter.
6. **People** – max. 3 über Themen-Relation (SQL: `"Themen" LIKE '%<themen-id>%'`),
   mindestens eine Person der Kategorie Kritiker:in oder Policy. **Seiten laden** (Rolle /
   Organisation, Notes, Seiteninhalt). Nur **dokumentierte Positionen** zählen als
   Wissensbasis. Ist nichts dokumentiert, darf ein Einwand aus der Rolle abgeleitet werden –
   gekennzeichnet als «aus Rolle abgeleitet – ausserhalb der Wissensbasis» und **ohne**
   Relation «Inspiriert von».
7. **Optionale Projekt-DBs** – nur wenn Topics, Constraints oder der Seed-Text es
   nahelegen, je max. 3. Skills-Registry per SQL auf `Skill Name` filtern.

Pro Kandidat festhalten: Titel, URL, DB, **Kern-Mechanismus in einem Satz** (domänenfrei,
aus Description/Summary abgeleitet), Status «bekannt» oder «neu». Dieser Satz ist der
Rohstoff der Kombination – Titel und Tags reichen nicht.

### Schritt 2 – Kombinations-Operatoren

Alle sechs Operatoren der Reihe nach anwenden. Jeder liefert 0–2 Kandidaten; ein leerer
Operator ist ein normales Ergebnis, kein Grund zum Erfinden.

**A · Mechanismus-Transfer** (Paper → Seed)
> Paper P beschreibt Mechanismus M. Angewendet auf das Seed-Problem heisst das: … Was
> wird dadurch möglich, was vorher nicht ging?

**B · Tool-Substitution** (AI-Tools → Seed)
> Tool T (Status, Hosting, Lizenz) implementiert M oder den Kern des Seeds bereits.
> Was entfällt (Eigenbau), was kommt dazu (Lizenz, Limits, Datenschutz)? Gibt es unter
> «Alternatives» eine Self-hosted-Variante?

**C · Constraint-Verschärfung**
> Eine der Constraints hart setzen – ohne Cloud-LLM / ohne Personendaten / Edge-only /
> null Budget / Prototyp in 2 Wochen / Zielgruppe wechseln (z. B. Leitung statt Fachperson).
> Was vom Seed überlebt, was muss anders gelöst werden, welches Tool passt dann noch?

**D · Morphologischer Kasten**
> Dimensionen: Datenquelle × Mechanismus × Deployment (SaaS | Self-hosted | Edge |
> Skill-Layer) × Zielgruppe × Bereich. Zellen mit den Retrieval-Kandidaten füllen. Welche
> Nachbarzelle des Seeds ist leer und mit vorhandenen Bausteinen erreichbar?

**E · Nachbar-Analogie** (Themen-Hub, zwei Hops)
> Seed → Thema → verwandtes Thema → dort vorhandene Lösung. Welches Muster lässt sich
> zurück auf den Seed übertragen?

**F · Stresstest** (People)
> Grundlage sind die **dokumentierten Positionen** aus den geladenen People-Seiten. Was
> würde Person X am Seed zuerst bemängeln? Welcher Kandidat aus A–E hält dem stand,
> welcher nicht? Rollen-Ableitungen sind erlaubt, aber als solche gekennzeichnet. Dieser
> Operator erzeugt keinen Kandidaten – er streicht oder schärft.

### Schritt 3 – Bewertung und Auswahl

1. **Duplikat-Check**: Jeden Kandidaten gegen die Cockpit-Nachbarn aus Schritt 1 halten.
   Identisch → verwerfen. Nahe verwandt → behalten, aber als «Weiterentwicklung von …»
   kennzeichnen.
2. **Ziel bestimmen** – jeder Kandidat ist entweder
   - **neuer Eintrag** (eigene Idee mit eigener Lösungsrichtung) oder
   - **in den Seed** (präzisiert den bestehenden Nächsten Schritt – z. B. ein Mess- oder
     Evaluationsdesign, eine Architekturentscheidung, eine Partnerliste).
3. **Scoring** mit den vier Cockpit-Zahlen (1–5), damit das Rückschreiben direkt passt:
   - Strategischer Wert – Nutzen für den Bereich; Hebel gemäss `config.yml → scoring_context`
   - Machbarkeit – mit vorhandenen Tools (Status Using/Mastered) und Constraints
   - Lernwert – was lernt der User, auch wenn es scheitert
   - Evidenz – gelesene Primärquellen hoch, «To Read» und Praxispositionen tiefer
4. Zusätzlich: **Datenschutz** 🟢 / 🟡 / 🔴 (Personendaten, Cloud-Verarbeitung,
   besonders schützenswerte Daten) und **Aufwand** S / M / L.
5. Top 3–5 behalten, davon **höchstens 4 mit Ziel «neuer Eintrag»** (Widget-Grenze). Jeder
   Kandidat braucht einen **Provenienz-Block**: welche Einträge (URLs) mit welchem Operator
   kombiniert wurden. Ohne Provenienz kein Kandidat.

### Schritt 4 – Präsentation und Freigabe

Kompakt, scanbar, dann Entscheidung abholen:

```
## Synthese: <Seed-Titel>
Seed: <Reifegrad> · <Bereich> · Problem: <1 Satz>
Retrieval: <n> Themen · <n> Papers · <n> Tools · <n> Ideen · <n> Fragmente · <n> People · <n> Skills
<1–3 Beobachtungen zum Retrieval: dünne DBs, fehlende Relationen, Tool-Status>

| # | Kandidat | Kombination | SW | MB | LW | EV | DS | Aufwand |
|---|---|---|---|---|---|---|---|---|
| 1 | … | Paper X × Tool Y (A+B) | 4 | 3 | 5 | 3 | 🟡 | M |

### 1 · <Kandidat-Titel>
Kern: <2 Sätze>
Kombination: [Paper X](url) × [Tool Y](url) – Operator A, dann B
Warum jetzt: <1 Satz>
Risiko / Stresstest: <1 Satz, Bezug auf Operator F>
Ziel: neuer Eintrag | in den Seed
Nächster Schritt (bei Beförderung zu 🧩): <1 konkreter Schritt>

**Verworfen:** <Einträge und Grund, 1 Zeile>
**Kennzeichnung:** <was ausserhalb der Wissensbasis stammt>
```

Danach **zwei Fragen in einem Widget-Aufruf**, je max. 4 Optionen:

- **Frage 1 (Multi-Select): «Welche Kandidaten als neue Funken ins Cockpit?»** – nur die
  Kandidaten mit Ziel «neuer Eintrag» (max. 4). Kandidaten mit Ziel «in den Seed» sind hier
  keine Option.
- **Frage 2 (Single), Fall A: «Seed anreichern?»** – *Ja, vollständig* (Synthese-Abschnitt
  inkl. Seed-Kandidaten, Relationen, Scores) · *Nur Relationen* · *Nein*.
  **Fall B: «Seed anlegen?»** – *als 🧩 Konzept mit Nächstem Schritt* · *als 🌱 Funke* ·
  *nicht anlegen*.

### Schritt 5 – Rückschreiben

Nie ohne Freigabe aus Schritt 4. **Reihenfolge: zuerst Modus 2 (Funken anlegen), dann
Modus 1 (Seed anreichern)** – so kann der Seed auf die neuen URLs verlinken.

**Modus 2 – neuer Eintrag pro gewähltem Kandidaten** (`notion-create-pages`, alle in einem Aufruf)
- Icon aus `config.yml` · Reifegrad **🌱 Funke** (🧩 nur, wenn Problem + Lösungsrichtung klar
  sind **und** der User den Nächsten Schritt bestätigt; nie höher)
- Titel · Idee (3–5 Sätze mit den Kernquellen) · Bereich · Topics (nur bestehende Optionen)
- Tags: Option aus `config.yml → cockpit.tag_synthese` (immer)
- Zahlen: Strategischer Wert, Machbarkeit, Lernwert, Evidenz aus Schritt 3
- Related Idea Cockpit → Seed (Fall A) + Nachbarn aus dem Duplikat-Check
- Relationen zu allen kombinierten Quellen: Papers & Ressourcen, AI-Tools, Themen,
  Fragmente, optionale Projekt-DBs; **Inspiriert von nur bei dokumentierter Position**
- **Kein** «Nächster Schritt» als Property (Funke). Stattdessen im Seiteninhalt:
  `## Provenienz (idea-synthesis, <Datum>)` – Seed, Operator(en), kombinierte Einträge mit
  Links und Mechanismus-Satz, Warum jetzt, Stresstest, Datenschutz-Flag, Bewertung – und
  `## Möglicher nächster Schritt (bei Beförderung zu 🧩 Konzept)`. So ist der Funke im
  Weekly ohne Chat-Kontext prüfbar und beförderbar.

**Modus 1 – bestehenden Seed anreichern** (`notion-update-page`)
- `insert_content` am Ende: Abschnitt `## Synthese <YYYY-MM-DD> (idea-synthesis)` mit
  Retrieval-Zeile, den Kandidaten mit Ziel «in den Seed» (Kern, Quellen, Protokoll,
  Bewertung), Liste der abgeleiteten Funken mit Links, ergänzten Relationen, Verworfenem.
  Bestehenden Inhalt nie überschreiben.
- `update_properties`: Relationen **nur hinzufügen, nie entfernen** – bestehende Werte
  mitgeben. Immer auch **Themen** setzen (Cockpit-Einträge haben sie oft nicht) und
  **Related Idea Cockpit** auf alle neuen Funken und gefundenen Nachbarn (Relation von
  beiden Seiten setzen).
- Leere Zahlenfelder mit den Scores füllen; befüllte Felder nicht überschreiben.
- Reifegrad **nicht** anheben. Ausnahme: User wählt einen Kandidaten explizit als Richtung
  → Nächster Schritt eintragen und 🧩 Konzept **vorschlagen**, Wechsel nur auf Bestätigung.

Relationen als JSON-Array von Page-URLs; Multi-Selects als Array; Zahlen als Zahl. Bei
Fehlern das Schema erneut laden, nicht raten.

### Schritt 6 – Kurzbericht

Vier Zeilen: was angelegt / angereichert wurde (mit Links); was verworfen wurde und warum;
was im nächsten Weekly zu prüfen ist; **ein Lauf-Befund** zur Wissensbasis (dünne DB,
fehlende Relationen, Tool-Status) – das kalibriert den Skill. Keine Wiederholung der
Kandidaten.

---

## Interaktionsprinzipien

1. **Seed vor allem.** Ohne Seed keine Synthese – dann zu `idea-serendipity` oder
   `idea-cockpit` verweisen.
2. **Provenienz ist Pflicht.** Ein Kandidat ohne Notion-URLs wird nicht präsentiert.
3. **Budget zählt Neues.** ~40 neue Einträge, ~15 Aufrufe, 3–5 Kandidaten. Bereits
   verknüpfte Einträge sind Rohstoff, keine Kandidaten.
4. **Wissensbasis vor Weltwissen.** Ergänzungen aus eigenem Wissen sind erlaubt, aber als
   «ausserhalb der Wissensbasis» markiert und ohne erfundene Quellen. Gilt auch für
   Stresstest-Einwände, die nur aus einer Rolle abgeleitet sind.
5. **Leer ist ein Ergebnis.** Liefert eine DB nichts Brauchbares, das sagen – das ist
   selbst eine Erkenntnis fürs Weekly.
6. **Widget-Grenzen:** max. 2 Fragen pro Aufruf, max. 4 Optionen pro Frage, immer per Widget.
7. **Cockpit-Regeln gelten weiter:** 3-Aktiv-Limit, kein Nächster Schritt bei Funken,
   kein Löschen ohne Bestätigung.
8. **Schweizer Rechtschreibung**, kein ß.

---

## Anti-Patterns

- ❌ Ganze Datenbanken laden («alle Papers zu RAG») statt über Themen-Hub, ai-search und SQL-Batch
- ❌ Jede Themen- oder Tool-Seite einzeln fetchen, wenn ein SQL-Batch reicht
- ❌ Bereits am Seed verknüpfte Einträge als «neue Kandidaten» präsentieren
- ❌ Kandidaten ohne Provenienz oder mit Provenienz, die nur Titel-Ähnlichkeit ist
- ❌ Paper-Inhalte aus dem Titel erraten, wenn die Summary fehlt
- ❌ Stresstest-Einwände aus Rollen als dokumentierte Positionen ausgeben; «Inspiriert von»
  ohne gelesene People-Seite setzen
- ❌ Mehr als 5 Kandidaten präsentieren; mehr als 4 Optionen in eine Widget-Frage packen
- ❌ Ohne Freigabe schreiben; bestehenden Seiteninhalt überschreiben; Relationen entfernen
- ❌ Reifegrad eigenmächtig anheben oder einen vierten 🚀 Aktiv-Eintrag erzeugen
- ❌ «Nächster Schritt» als Property bei einem Funken (gehört als «Möglicher nächster
  Schritt» in den Seiteninhalt)
- ❌ Tag oder Icon vergessen – damit ist der Erfolg nicht mehr messbar
- ❌ Ein Weiterentwicklungs-Vorschlag als neuer Eintrag, obwohl er in den Seed gehört
- ❌ IDs im SKILL.md hart codieren – sie gehören in `config.yml`

---

## Erfolgsmessung

Nach vier Wochen im Weekly über die View «🔗 Synthese» (Filter: Tags enthält die
Synthese-Option) prüfen:

| Kennzahl | Lesart |
|---|---|
| Synthese-Funken, die 🌱 → 🧩 befördert wurden | die einzige Zahl, die zählt |
| Angereicherte Seeds, deren Nächster Schritt danach umgesetzt wurde | Modus 1 funktioniert |
| Synthese-Funken, die eingefroren oder gelöscht wurden | Rauschen – Operatoren prüfen |

Null Beförderungen nach vier Wochen → Operatoren oder Retrieval-Budget ändern, nicht
mehr Kandidaten erzeugen.

---

## Einrichtung

1. `config.example.yml` nach `config.yml` kopieren und die IDs des eigenen Workspace eintragen.
2. Im Idea Cockpit die Multi-Select-Option für Synthese-Einträge im Feld Tags anlegen
   (`notion-update-data-source`); Name in `config.yml` eintragen.
3. View «🔗 Synthese» im Idea Cockpit anlegen (`notion-create-view`: Filter Tags = Synthese-Option,
   Sortierung Erstellt absteigend) – für die Erfolgsmessung.
4. Empfohlen – Themen-Relationen im Cockpit nachziehen: Backfill über die Themen der
   verknüpften Papers/Tools (SQL-Join über die Data Sources, Mehrheitsregel: bei ≥3 Quellen
   nur Themen mit ≥2 Nennungen, max. 3 pro Idee, «Archiv» ausgeschlossen; Schreiben per
   `notion-update-page`). Ohne diesen Schritt läuft der Themen-Einstieg nur über die Quellen.
5. Optional – v2: Textfeld **Kern-Mechanismus** in AI-Tools und Source Library (ein Satz,
   domänenfrei). Damit entfällt die Ableitung in Schritt 1 und die Kombinationen werden
   präziser.

---

## Beispiel-Interaktionen

**Seed aus dem Cockpit:**
> «Entwickle ‹Lokales RAG-Wissenssystem› weiter – was habe ich dazu schon?»
→ Eintrag + verknüpfte Einträge per Batch laden → Themen über Papers/Tools → ai-search je DB
→ Operatoren A–F → 3–5 Kandidaten mit Ziel → 2 Fragen → Funken anlegen → Seed anreichern

**Problem ohne Eintrag:**
> «Die Leitungen fragen dauernd dasselbe zu internen Weisungen. Was lässt sich aus meiner
> Wissensbasis dafür bauen?»
→ 2 Fragen (Bereich, Constraints) → Steckbrief → Retrieval → Kandidaten → Frage 1 Funken,
Frage 2 Seed anlegen → Seed als 🧩 Konzept, Kandidaten als verbundene Funken

**Von Funke zu Konzept:**
> «Mach aus dem Funken ‹Briefvorlagen-Generator› ein Konzept.»
→ Synthese → User wählt Richtung → Nächster Schritt eintragen → 🧩 vorschlagen → Wechsel
auf Bestätigung

---

## Changelog

- **v1.1 (05.10.2026)** – nach Referenzlauf: Budget zählt nur neue Einträge; Widget-Grenzen
  (2 Fragen, 4 Optionen) und Zwei-Fragen-Freigabe; Operator F liest People-Seiten,
  Rollen-Ableitung gekennzeichnet, «Inspiriert von» nur bei dokumentierter Position;
  Themen-Einstieg über Papers/Tools des Seeds; SQL-Batch und ai-search als Standard;
  Kandidaten-Ziel «neuer Eintrag | in den Seed»; Reihenfolge Modus 2 → Modus 1; Icon;
  Provenienz-Block mit «Möglicher nächster Schritt»; Lauf-Befund im Kurzbericht;
  Mess-View; Konfiguration in `config.yml`.
- **v1.0 (04.10.2026)** – Erstfassung.
