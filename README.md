# SBB Stationboard

A minimal, single-file departure board for Swiss public transport — designed for wall-mounted displays or lightweight browser setups. Pulls live data from the [Transport Opendata API](https://transport.opendata.ch) and refreshes automatically.

---

## Features

* Monitor multiple stations simultaneously
* Fine-grained filtering for specific lines and directions
* Real-time countdown (minutes until departure) + delay display
* Configurable “time-to-station” filter (hide unreachable connections)
* Active time window (e.g. disable overnight updates)
* Zero build step — single HTML file using [Petite Vue](https://github.com/vuejs/petite-vue)
* Automatic full page reload every 24h (prevents long-term drift / memory issues)

---

## Getting Started

1. Download or clone `index.html`
2. Open it in any browser
3. Adjust configuration at the top of the `<script>` block

No server, no bundler, no install required.

---

## Configuration

All configuration is defined inside `index.html`.

---

### `BASE`

Set this depending on where the file is hosted (only relevant for PWA/manifest):

```html
<base href="/sbb/">
```

---

### `STATIONS`

Defines which stations are queried.

```js
const STATIONS = {
    "8503001": "Alt",
    "8591167": "Gr",
    "8591402": "Tü"
};
```

* **Key**: Station ID
* **Value**: Short label displayed on screen (keep it short)

---

### `CONNECTIONS`

Filters which departures are shown.

Leave empty (`[]`) to show everything.

```js
const CONNECTIONS = [
    ["8591167","T",17,"8591402"],
    ["8591402","B",80,"8591434"],
    ["8503001","S",5,"8503020"],
    ["8503001","S",14,"8503000"],
];
```

Structure:

```js
[stationId, category, number, nextStationId]
```

This allows **direction filtering** via the immediate next stop (`passList[1]`).

---

### `ABBREVIATIONS`

Used to shorten destination names.

```js
const ABBREVIATIONS = {
  "Bahnhof": "Bhf",
  "Strasse": "Str",
  "Zentrum": "Zentr",
  "Hauptbahnhof": "HB"
};
```

Additional automatic constraints:

* max 2 words
* max 4 characters per word (truncated with `.`)

---

### `TTS` — Time to Station

Filters out departures that are too soon to realistically catch.

```js
const TTS = 4;
```

* Uses **planned departure + delay**
* Set to `0` to disable filtering

---

### `ACTIVE_TIME_INTERVAL`

Controls when the board is active.

```js
const ACTIVE_TIME_INTERVAL = [5, 24];
```

* Format: `[startHour, endHour)` (24h)
* Uses local system time (`Date.getHours()`)

Behavior:

* `hour >= start && hour < end` → active → data is fetched
* otherwise → display is cleared (`departures = []`)

Notes:

* `24` effectively means *until midnight*
* Set to `null` to keep updates always active

---

### `REFRESH_INTERVAL`

Polling interval in seconds.

```js
const REFRESH_INTERVAL = 30;
```

* Controls how often the API is queried
* Lower values increase load and may trigger **IP-based rate limiting**

---

## Runtime Behavior

* Data is fetched per station and merged
* Sorted by departure time
* Filtered by:

  * `TTS`
  * `CONNECTIONS`
* Limited to 18 entries
* Updated via `setInterval()`
* Hard reload every 24h:

```js
location.reload();
```

---

## URL Overrides

Quick overrides via query params:

```
?station=8503000
?tts=2
```

* `station` → show only a single station
* `tts` → override time-to-station

---

## Hash override (iOS PWA workaround)
```
#8503000
```
If a hash is present:

the hash value is used as the station ID
tts is forced to 0
connections are disabled

This is mainly useful for iOS home screen PWAs, because Safari may strip query parameters from saved home screen links.

---

## Data Source

```
https://transport.opendata.ch/v1/stationboard?id={stationId}&limit=100
```

* No API key required
* Returns real-time + planned data
* Includes delay + full stop sequence (`passList`)

---

## Notes / Limitations

* Depends on system clock accuracy
* No caching → each refresh hits API directly
* Direction filtering assumes `passList[1]` exists
* Layout is optimized for fixed-width / large-display usage

---

## License

MIT
