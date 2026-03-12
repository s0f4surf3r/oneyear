# OneYear Kalender — Projekt-Gedächtnis

## Was ist das?
Digitaler Jahreskalender für Jochen & Levi. Zeigt Papa-/Mama-Zeiten, Ferien, Geburtstage.
Gebaut als Single-File React App (kein Build-Step, alles in `index.html`).

## Stack
- React 18 (via CDN, kein npm)
- Tailwind CSS (via CDN)
- Babel Standalone (JSX im Browser)
- Alles in einer Datei: `oneyear-jahreskalender/index.html`

## Event-Typen (relevant für Unterhaltsrechner)
```js
papazeit:  { label:'Papa-Zeit', color:'#4A90E2', emoji:'👨🏼', hasStripe:true }
mamazeit:  { label:'Mama-Zeit', color:'#FF69B4', emoji:'👩🏻', hasStripe:true }
kita:      { label:'Kita',      color:'#FFC0CB', emoji:'🎨' }
```

Events haben immer: `{ type, startDate: 'YYYY-MM-DD', endDate: 'YYYY-MM-DD', title }`

## Nächste Aufgabe: Unterhaltsrechner einbauen

### Was gebraucht wird
Am Ende jedes Monats soll automatisch angezeigt werden:
- Anzahl Übernachtungen bei Papa in diesem Monat
- Berechneter Unterhalt den Jochen an Katja zahlt

### Die Funktion (von ChatGPT generiert, getestet)
```js
function berechneUnterhalt(events, year, month) {

  // KONSTANTEN — bei Bedarf anpassen
  const EINKOMMEN_VATER = 1959.60;
  const EINKOMMEN_MUTTER = 2700;
  const TABELLENBEDARF = 554; // Düsseldorfer Tabelle, Altersstufe 6–11
  const KINDERGELD = 255;
  const GEWICHT_EINKOMMEN = 0.5;
  const GEWICHT_BETREUUNG = 0.5;

  function toDate(d) { return new Date(d + "T00:00:00"); }
  function daysBetween(a, b) { return Math.round((b - a) / 86400000); }

  const startOfMonth = new Date(year, month - 1, 1);
  const endOfMonth = new Date(year, month, 0);

  let uebernachtungen = 0;
  events.filter(e => e.type === "papazeit").forEach(event => {
    let start = toDate(event.startDate);
    let end = toDate(event.endDate);
    if (start < startOfMonth) start = startOfMonth;
    if (end > endOfMonth) end = endOfMonth;
    const nights = daysBetween(start, end);
    if (nights > 0) uebernachtungen += nights;
  });

  const tageImMonat = endOfMonth.getDate();
  const betreuungsanteilPapa = uebernachtungen / tageImMonat;
  const gesamtEinkommen = EINKOMMEN_VATER + EINKOMMEN_MUTTER;
  const einkommenPapa = EINKOMMEN_VATER / gesamtEinkommen;
  const quotePapa = einkommenPapa * GEWICHT_EINKOMMEN + betreuungsanteilPapa * GEWICHT_BETREUUNG;
  const restbedarf = TABELLENBEDARF - KINDERGELD;
  const unterhalt = Math.max(0, Math.round(restbedarf * quotePapa));

  return { uebernachtungen, unterhalt, grundlage: `Restbedarf ${restbedarf}€ × Quote ${(quotePapa*100).toFixed(1)}%` };
}
```

### Wo einbauen
In `index.html` — in der Monatsansicht/Statistik-Bereich. Pro Monat eine Zeile:
```
👨🏼 12 Nächte · Unterhalt: 134 €
```

Kann per Klick expandieren (Grundlage anzeigen).

### Prompt für diese Session
Direkt kopieren und in Claude Code einfügen:

---

Ich möchte in den OneYear Kalender (`oneyear-jahreskalender/index.html`) den Unterhaltsrechner einbauen.

**Kontext:**
- React 18 Single-File App (kein Build-Step, alles inline)
- Events haben `{ type, startDate: 'YYYY-MM-DD', endDate: 'YYYY-MM-DD' }`
- `papazeit`-Events = Levi bei Papa
- Die fertige `berechneUnterhalt(events, year, month)`-Funktion liegt in dieser CLAUDE.md

**Ziel:**
In der Monatsansicht (Monatsstatistik / Footer jedes Monats) soll pro Monat automatisch erscheinen:
- Anzahl Übernachtungen bei Papa
- Berechneter Unterhaltsbetrag (€)
- Per Klick: Berechnungsgrundlage einblenden

**Stil:** Dezent, kleine Zeile unter dem Monat. Passt sich ans bestehende Design an (Tailwind, gleiche Farben).

Lies zuerst die `index.html` komplett, verstehe die Monatsstruktur, dann bau es ein.

---

## Hardware-Vision (Langfristig)
- Native Mac App (SwiftUI) auf 32" 4K Monitor (Minimum)
- Dauerinstallation, läuft immer
- KI-Integration: man kann mit dem Kalender sprechen (Claude API)
- Später: 5K/6K möglich, nie unter 32"
- Basis: diese Web-App → als SwiftUI-App neu bauen
