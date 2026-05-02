# SBB Stationboard

A minimal, single-file departure board for Swiss public transport — designed to run on a wall-mounted screen or any browser. Pulls live data from the [transport.opendata.ch](https://transport.opendata.ch) API and auto-refreshes every 15 seconds.

## Features

- Monitors multiple stations simultaneously
- Filters for only the connections you care about
- Shows minutes until departure + real-time delay
- Hides connections you can no longer reach (configurable walk time)
- Zero dependencies beyond [petite-vue](https://github.com/vuejs/petite-vue) — just open the HTML file
- Auto-reloads every 24h to stay fresh

## Getting Started

1. Download or clone `index.html`
2. Open it in any browser — no build step, no server needed
3. Configure your stations and connections (see below)

## Configuration

### BASE

First of all change this according to where this file sits in your website for the manifest to work.
The manifest does not need to work for the tool to work.
```html
<base href="/sbb/">
```


All config lives at the top of the `<script>` block in `index.html`.

### `STATIONS`

The stations to fetch departures from. Find a station ID by searching on [transport.opendata.ch](https://transport.opendata.ch/v1/stationboard?id=Zürich).

```js
const STATIONS = {
    "8503001": "Alt",  // Altstetten
    "8591167": "Gr",   // Grünaustrasse
    "8591402": "Tü"    // Tüffenwies
};
```

The value is a short abbreviation shown on screen — keep it to 2–3 characters.

Some useful Zürich station IDs:

| ID | Station |
|----|---------|
| `8503000` | Zürich HB |
| `8503001` | Altstetten |
| `8503020` | Hardbrücke |
| `8591167` | Grünaustrasse |
| `8591402` | Tüffenwies |
| `8591434` | Winzerhalde |

---

### `CONNECTIONS`

Filters which departures to show. Leave as `[]` to show everything.

Each entry is an array of four values:

```js
[stationId, category, number, nextStationId]
```

| Field | Type | Description |
|---|---|---|
| `stationId` | string | Departure station ID |
| `category` | string | Line type: `S`, `IR`, `T`, `B`, etc. |
| `number` | number | Line number (e.g. `5` for S5, `17` for tram 17) |
| `nextStationId` | string | ID of the **immediate next stop** — used to determine direction |

```js
const CONNECTIONS = [
    ["8591167", "T", 17, "8591402"], // Tram 17 from Grünaustrasse toward HB
    ["8591402", "B", 80, "8591434"], // Bus 80 from Tüffenwies toward Hönggerberg
    ["8503001", "S",  5, "8503020"], // S5 from Altstetten toward Hardbrücke
    ["8503001", "S", 14, "8503000"], // S14 from Altstetten toward HB
];
```

> The next station ID is how direction is determined. You can find it by looking at the `passList` in the API response.

---

### `ABBREVIATIONS`

Shortens long destination names to fit the display.

```js
const ABBREVIATIONS = {
    "Bahnhof":      "Bhf",
    "Strasse":      "Str",
    "Zentrum":      "Zentr",
    "Hauptbahnhof": "HB"
};
```

Destinations are also capped at 2 words and 4 characters per word automatically.

---

### `TTS` — Time to Station

Minimum minutes until departure. Connections leaving sooner than this are hidden (accounts for delay).

```js
const TTS = 5  // hide anything leaving in less than 5 minutes
```

Set to `0` to show all upcoming departures.

## Data Source

Live departure data comes from the open Swiss transport API:

```
https://transport.opendata.ch/v1/stationboard?id={stationId}&limit=100
```

No API key required. See the [full API docs](https://transport.opendata.ch) for more details.

## License

MIT