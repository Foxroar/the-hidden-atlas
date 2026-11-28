# Hidden Atlas development readiness

Audit date: 2026-11-28 UTC

Baseline: `a8e93015bb83e8b27a8131d17e021a02aafbe579` (`main`)

Branch: `development/readiness-audit`

The app runs from a fresh checkout with a static HTTP server and is ready for branch-based development. Normal browsing and map layers work. Fix the notebook state and startup errors before relying on notebook persistence. The map needs a clearer failure state for unavailable tiles.

## Repository and local setup

Read [AGENTS.md](AGENTS.md) and inventoried all four tracked files, the visible branch history and both release archives. [index.html](index.html) contains the entire UI, 26 records and all application CSS and JavaScript. Both archives pass `unzip -t` and each contains a single `index.html`. The v0.4.1 archive matches the checked-out HTML byte for byte. The older archive differs in the Biltmore record. The runtime heading still reads Field Edition 0.4.

From the repository root:

```sh
python3 -m http.server 4186 --bind 127.0.0.1
```

Open `http://127.0.0.1:4186/`. No install, compile, environment variables, secrets, backend or database are required. The HTML and Leaflet assets served successfully, and the extracted inline JavaScript passed `node --check`. Leaflet 1.9.4 CSS and JavaScript load from unpkg. OpenStreetMap and Esri supply the map images. There is no package manifest, repository test command, CI workflow or Vercel configuration file. Neither a missing package nor a missing tracked runtime asset was found.

## Browser checks

Live layers were tested in shared Chrome using the repository's HTTP server and real provider requests. Playwright Chromium used the same HTML and pinned Leaflet assets with a local image fixture for repeatable interaction and outage tests. Those fixture results establish application behavior independently of tile-provider network availability. Mobile results are viewport and touch emulation. The WebKit binary downloaded, but its launch was blocked by missing Linux libraries, including GTK 4. Physical iPad/iPhone and Safari checks remain outstanding.

| Area | Result |
| --- | --- |
| Initial map and controls | The default dossier and 26 markers loaded. Antique and Street loaded OpenStreetMap imagery, Satellite loaded Esri World Imagery, and Terrain relief loaded Esri World Hillshade. Antique toggled the sepia class. All four switched their active button and explanatory text. Satellite and Terrain preserved zoom and the visible map center within tile pixel rounding. |
| Search and categories | Mixed-case name search, combined search and category, and the empty state worked. Category totals in list and Leaflet pins agreed: 11 expositions, 2 forts, 10 mounds, 2 civic and rail, 1 estate, and 26 total. Whitespace around a query can suppress a valid match. |
| Dossiers and buttons | List selection, Leaflet marker selection, Surprise me, dossier tabs, and Connections navigation worked. The calculator updated with ordinary inputs on records with a floor area. Zero workers displayed infinite duration. |
| Bookmark persistence | Bookmarking, reloading, viewing the stored entry, removing the bookmark, and reloading again confirmed add/remove persistence on one origin. Selecting the saved entry exposed the state error below. |
| Leaflet outage | Blocking unpkg rendered the selectable SVG map. Its pins worked, but search and category filtering affected the list only. Selecting Antique replaced the offline message. |
| Tile outage | Blocking OpenStreetMap tile requests while leaving Leaflet available kept the markers and controls, with no basemap or warning. The SVG fallback did not appear. |
| Responsive | iPhone and iPad portrait and landscape tests covered map width, horizontal overflow, and touch selection of an entry and dossier tab. At 375, 390, 768, 810, 844, 1024 and 1080 CSS pixels there was no page-wide horizontal overflow. The 768 px iPad portrait layout stacked, while 810 px iPad portrait and the 1024 and 1080 px tablet landscape widths showed a sidebar. At 844 × 390, the landscape phone used the desktop sidebar and a 600 px map, so the map extended below the fold. Dossier tabs often needed horizontal scrolling. |

## Reproduced issues

| Priority | Finding | Reproduction and likely cause |
| --- | --- | --- |
| High | Opening a saved entry throws a JavaScript error and leaves the Notebook toolbar state inconsistent. | Save an entry, open Notebook, then select the entry. `select()` exits notebook mode and re-renders while the toolbar still displays “Show all locations.” Changing the button's `textContent` has removed the `#count` span, so `render()` throws `Cannot set properties of null (setting 'textContent')`. The underlying saved entry persists across reload. See [selection and toolbar handlers](index.html#L44-L46). |
| High | Malformed or inaccessible notebook storage prevents startup. | Put malformed JSON such as `{broken` in `hiddenAtlasSaved02` and reload. An uncaught `SyntaxError` leaves the list and map uninitialized. Simulating an unavailable storage API also prevents startup. The storage read and JSON parse run before map initialization. See [startup](index.html#L33). |
| High | All ten mound dossiers generate unrelated archive queries. | Open Cahokia Mounds and choose Photos & plans. All five visible links contain `undefined` in their search queries because mound records lack `terms`. The destinations can return HTTP 200 while showing irrelevant results. See [mound data and archive links](index.html#L22-L40). |
| Medium | Tile outages leave a blank basemap with a normal layer note. | Block OpenStreetMap while allowing unpkg. The fallback only checks whether Leaflet initialized. There is no tile-error handling for an initialized map. See [map startup and layer switching](index.html#L34-L39). |
| Medium | The fallback map ignores filters and can imply that a tile layer is active. | Block unpkg, then search Fort Jefferson. The list showed one result while the SVG retained all 26 pins. Switching to Terrain and back to Antique changed the note without changing the SVG. See [fallback pins and layer switching](index.html#L35-L39). |
| Medium | Calculator accepts a zero denominator. | In Build estimate, set Workers on site to `0`. The result shows `Infinity working days · Infinity years`. HTML input bounds do not gate the arithmetic. All ten mound records also lack `size`, so Build estimate opens with an empty floor area and reports zero labor and material until the user enters an area. See [calculator](index.html#L42-L43). |
| Low | Space-padded search can miss an exact entry name. | Search for ` Fort Jefferson ` with one space on each side. The unpadded mixed-case query returns one location, while the padded query returns zero. See [search](index.html#L45). |

## Links and deployment

- The USGS viewer and Leaflet attribution URL returned HTTP 200. A hidden Library of Congress placeholder at `/resource/sn83045462/?fo=json` returned HTTP 404. The other hidden newspaper endpoint timed out during the first probe. Neither hidden anchor is part of the visible navigation. See [archive link generation](index.html#L40).
- Checked the generated URLs for all 130 visible discovery links across 26 dossiers. Chicago and Cahokia samples were probed over HTTP. Wikimedia and Google responded for the checked searches. Both visible Library of Congress sample requests timed out in the audit environment, so their reachability is unverified. The mound links failed at the query level. These links point to search results and depend on third-party site behavior. An HTTP success confirms endpoint reachability only.
- The former GitHub Pages candidate at `https://foxroar.github.io/the-hidden-atlas/` returned HTTP 404. GitHub Pages is not needed for the Vercel workflow.
- Vercel shows two projects linked to this repository, `the-hidden-atlas` and `the-hidden-atlas-btm9`, with prior ready production and preview deployments. Check the PR deployment status by project and commit to distinguish the previews. Both projects report no framework preset and Vercel Authentication protection for generated URLs. This audit uses the existing Git integration.
- The project can be served from the repository root as static files. For portability, verify the Vercel project still points to that root and does not require an install or framework build. Assess whether the release ZIPs should be directly downloadable when publishing the repository root.
- Notebook data is browser- and origin-specific. Localhost, Vercel preview and production each have their own notebook. Preview deployment protection can also require authorized Vercel access.
- External dependency risks include unpkg reachability or integrity, provider rate limits or outages, and missing basemap failure feedback. Consider vendoring or integrity-pinning Leaflet, confirming provider usage policies before increasing traffic, and adding fallback coverage to a future smoke suite.

## Suggested order for follow-up

1. Make notebook storage tolerant of invalid or inaccessible data and keep the toolbar label and count in sync.
2. Supply valid archive terms and explicitly handle unknown calculator defaults for mound records.
3. Add a visible degraded map state for tile failures and apply list filters to fallback pins.
4. Validate calculator inputs and trim search queries. Confirm touch and tab scrolling on physical Safari devices.
