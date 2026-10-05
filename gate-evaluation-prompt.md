# Gate-Auswertung – Prompt für den einmaligen Scheduled Task (Vorlage)

> Vorlage für die Auswertung nach einem vierwöchigen Messfenster (Gate 1 → 2 in
> `ROADMAP.md`). Platzhalter `{{…}}` aus `config.yml` und dem Lauf-Setup ersetzen. Der Task
> misst und empfiehlt; er ändert keine Cockpit-Einträge und keinen Scheduled Task. Einzige
> Schreibvorgänge: der Bericht als Kommentar und eine Audit-Zeile am Registry-Eintrag.

---

# Gate-1-Auswertung idea-synthesis (einmalig, {{gate_datum}}) – Standalone-Prompt

Du wertest das vierwöchige Messfenster des Skills `idea-synthesis` aus und empfiehlst nach der Regel aus `ROADMAP.md`, wie es weitergeht. Du startest ohne Kontext. Schweizer Rechtschreibung. Deutsch. Du änderst keine Cockpit-Einträge, keinen Reifegrad, keinen Scheduled Task.

## Hintergrund
- Seit {{messfenster_start}} läuft der Skill: ein Referenzlauf und vier automatisierte Läufe des Scheduled Task «Synthese-Lauf» (`{{trigger_id}}`). Jeder Lauf endet bei einer Vorschau; der User gibt im Run-Chat frei. Geschriebene Einträge tragen im Idea Cockpit den Tag «{{cockpit.tag_synthese}}».
- Vergleichsgruppen: Funken des Serendipity-Agenten (Tag «Serendipity») und eigene Funken ohne Tag im selben Fenster.
- Das Weekly (Montag) ist die Qualitätskontrolle: Funken werden befördert, belassen oder eingefroren.

## Notion
- 💡 Idea Cockpit: `collection://{{notion.idea_cockpit.data_source_id}}` – `Titel`, `Reifegrad`, `Bereich`, `Tags`, vier Scores, `Erstellt`, `Zuletzt bewegt`, `Nächster Schritt`, `Related Idea Cockpit`, `Themen`.
- Skills-Registry, Eintrag `idea-synthesis`: Seite `{{registry_page_id}}` (Felder `Audit`, `Known Issues`, `Changelog`).
- SQL mit `notion-query-data-sources`; Schema vorher mit `notion-fetch` prüfen. Datumsvergleiche über `"Erstellt" >= '{{messfenster_start}}'`.

## Schritt 1 – Synthese-Gruppe
Alle Einträge mit `"Tags" LIKE '%{{cockpit.tag_synthese}}%'`: Titel, URL, Reifegrad, Bereich, Erstellt, Zuletzt bewegt, Score-Summe. Zählen nach Reifegrad. **Beförderungen** = 🧩 Konzept, 🗺️ Plan oder 🚀 Aktiv. **Rauschen** = ❄️ Eingefroren (Gelöschte sind unsichtbar – vermerken). **Unentschieden** = 🌱 Funke. Rauschen-Quote = Rauschen / Gesamt.

## Schritt 2 – Angereicherte Seeds
Einträge ohne Synthese-Tag, die in `Related Idea Cockpit` eines Synthese-Funkens stehen und deren Seite einen Abschnitt «## Synthese» enthält (max. 6). Pro Seed: Reifegrad heute, Zuletzt bewegt, ob der Nächste Schritt seit der Synthese verändert wurde. Zählen, bei wie vielen Reifegrad gestiegen oder Nächster Schritt umgesetzt wurde.

## Schritt 3 – Vergleichsgruppen (gleiches Fenster)
Eigene Funken (`Erstellt` im Fenster, Tags ohne Synthese/Serendipity) und Serendipity-Funken: je nach Reifegrad zählen, Beförderungsquote. Weekly-Aktivität: Anzahl Einträge mit `Zuletzt bewegt` im Fenster (Indikator, ob Weeklys stattfanden).

## Schritt 4 – Lauf-Protokoll
Registry-Eintrag lesen, Feld `Audit`: Zeilen «Lauf <Nr> …» zählen und zusammenfassen (Freigaben, Funken je Lauf, Aufrufe je Lauf). Fehlende Läufe vermerken.

## Schritt 5 – Gate-Entscheid nach ROADMAP.md
- **Bestanden:** ≥ 2 Beförderungen aus ≥ 3 Läufen und Rauschen ≤ 50 % → Task weiterlaufen lassen (er ist das Stufe-2-Setup), Release v1.2.0, zweites Messfenster mit zusätzlichen Kennzahlen (Freigabe-Latenz, verfallene Vorschauen, verworfene Seed-Auswahl); Recall-, Budget- und Volumen-Lücke für Gate 2 → 3 beobachten.
- **Teilweise (genau 1):** Operatoren und Retrieval-Budget überarbeiten (Skill v1.2), Messfenster um 4 Wochen verlängern. Benennen, welcher Operator die beförderte Idee geliefert hat und welche Bereiche Rauschen erzeugten.
- **Verfehlt (0):** Task pausieren empfehlen (Trigger-ID `{{trigger_id}}`). Diagnose in dieser Reihenfolge: (1) Prozess – Weeklys stattgefunden, Vorschauen freigegeben? (2) Daten – Themen-Relationen der Seeds und Quellen? (3) Methode – Operatoren.
Zusätzlich die Beförderungsquoten der drei Gruppen vergleichen und in einem Satz sagen, welche Quelle die beförderten Funken liefert.

## Schritt 6 – Bericht
Struktur: (1) Verdikt in einer Zeile «Gate 1: bestanden | teilweise | verfehlt – <n> Beförderungen aus <n> Läufen, Rauschen <x> %»; (2) Tabelle Synthese-Gruppe nach Reifegrad mit Links; (3) angereicherte Seeds; (4) Vergleich der drei Gruppen; (5) Lauf-Protokoll; (6) Empfehlung mit den nächsten drei Schritten; (7) Befunde zur Wissensbasis.
Bericht als Abschlussnachricht ausgeben. Dann `notion-create-comment` mit dem Bericht auf der Registry-Seite und das Feld `Audit` um die Zeile `Gate-1-Auswertung <Datum>: <Verdikt-Zeile>` ergänzen (bestehenden Text behalten). Sonst nichts schreiben.

Falls Notion nicht erreichbar ist oder das Schema abweicht: Fehler klar melden, nichts erfinden, keinen Bericht mit geschätzten Zahlen abgeben.
