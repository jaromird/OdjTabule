<!-- Generated guidance for AI coding agents working on this repo -->
# Copilot instructions for OdjTabule

This repo is a small static web app that renders a virtual departure board from local GTFS CSV files (in `data/`). The UI is a single HTML file with embedded JavaScript: `tabule_stream.html`.

- **Run / debug locally:** use `startServeru.bat` or run `python -m http.server 8000` from the repo root, then open `http://localhost:8000/tabule_stream.html`.
- **Primary data source:** the app reads GTFS files in `data/` (notably `stops.txt`, `trips.txt`, `routes.txt`, `calendar.txt`, `calendar_dates.txt`, and streams `stop_times.txt`). Do not change filenames unless updating the fetch paths in `tabule_stream.html`.

- **Big picture / architecture:**
  - Single-page static UI with inline JS. No build system, frameworks, or server-side processing.
  - Initialization flow: `loadAllAndStream()` -> `loadAllMetadata()` (loads metadata CSVs) -> `loadStopTimesStream('data/stop_times.txt')` (streams stop_times.txt and incrementally updates the table).
  - Filtering and rendering are in-memory: `stopTimesByStop`, `tripsMap`, `routesMap`, and `validTripIds` drive `updateTable()`.

- **Important constants and where to change behavior:** edit these in `tabule_stream.html` near the top of the script:
  - `MAX_ROWS` — number of rows shown.
  - `REFRESH_SECONDS` — how often the DOM refreshes (calls `updateTable`).
  - `FULL_RELOAD_TIME` — daily full-reload time string (HH:MM).
  - `INITIAL_SELECTED_STOP_IDS` — fallback stop ids to preselect a stop.
  - `VEHICLE_HIGHLIGHT` — route_type -> color mapping.

- **Data handling specifics (important for changes):**
  - CSV parsing uses a custom `parseCSVLine()` to handle quoted fields and doubled quotes — reuse this parser if adding CSV reads.
  - `validTripIds` is computed from `calendar.txt` and `calendar_dates.txt` (service validity for the current date). Preserve this logic when modifying trip filtering.
  - `loadStopTimesStream()` uses the Fetch stream reader (`res.body.getReader()`) and relies on incremental parsing; it also reads `Last-Modified` header to set `stopTimesLastModified` used by the status indicator.

- **UI / DOM patterns:**
  - The app manipulates DOM directly (no frameworks). Use `document.createElement` and `innerHTML` cautiously (see `updateTable()` usage).
  - Station selection uses a `select#stationSelect` populated from `stops.txt` via `populateStationSelect()`; selected station persisted in cookie `selectedStation`.
  - Filters (lines/time) are simple DOM controls; `updateLineFilterList()` rebuilds the line filter UI based on current `stopTimesByStop` and `validTripIds`.

- **Network and hosting expectations:**
  - The app expects the CSV files to be served by a static HTTP server (to allow streaming and to provide `Last-Modified` headers). Running directly from the filesystem (file://) will break fetch() calls.

- **When editing code:**
  - Keep all CSV-to-object mapping indices consistent with the header parsing approach already present (`Object.fromEntries(...map((h,i)=>[h,i]))`).
  - If you change field names or add derived fields, update the related index lookups in `loadAllMetadata()` and `loadStopTimesStream()`.
  - Avoid changing global variable names without updating all references — the code uses many globals (e.g., `stopTimesByStop`, `routesMap`, `tripsMap`, `validTripIds`).

- **Quick examples:**
  - To add a new vehicle color for route_type `3` (bus), edit `VEHICLE_HIGHLIGHT['3']` in `tabule_stream.html`.
  - To force a full reload from the console: call `fullReload()` in the page context.
  - To change the display threshold that switches between "minutes left" and absolute time, edit `MINUTES_FORMAT_THRESHOLD`.

- **Conventions & gotchas:**
  - UI text is Czech; sorting and locale-specific behaviour uses `localeCompare(..., 'cs')` — keep this for correct ordering.
  - The project has no tests or CI configured; be conservative with changes and test locally by serving the `data/` files.
  - The streaming loader assumes `stop_times.txt` is large and may be served incrementally; if you replace that with a static JSON API, update the progress UI and remove reader-based incremental parsing.

If anything above is unclear or you want me to expand a section (examples, where to change a specific behavior, or to add quick code snippets), tell me which part and I will iterate. 
