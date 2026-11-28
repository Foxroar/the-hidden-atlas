# Hidden Atlas preview readiness

Audited November 28, 2026 UTC. This report covers `899e31e81d4730985530a63f217020b06b991366` on `feature/collection-focus`, which includes the redesign, hiking overlay, and subject-first collection. Production `main` is still `a8e9301`, the baseline covered in the [original audit PR](https://github.com/Foxroar/the-hidden-atlas/pull/2). This report is the only change introduced by the audit branch.

## Repository and setup

Read [AGENTS.md](AGENTS.md), inspected the full tracked tree, reviewed the branch history and the single application file, and checked both ZIPs. [index.html](index.html) contains the entire interface, the 26 site records, and the inline JavaScript and CSS. Both ZIPs contain one `index.html` and pass `unzip -t`. The fixed v0.4.1 archive matches production `main` byte for byte. Both archives contain older interfaces than the audited preview. The preview header still says Field Edition 0.4.

From the repository root, serve the checkout over HTTP.

```sh
python3 -m http.server 4190 --bind 127.0.0.1
```

Open `http://127.0.0.1:4190/`. Local responses and inline JavaScript syntax were checked. There is no package install, build step, database, environment variable, or missing tracked runtime asset. Leaflet 1.9.4 loads from unpkg. OpenStreetMap, Esri and Waymarked Trails supply map tiles. There is no repository-owned test suite, CI workflow, or Vercel configuration file.

## Checks run

Chromium ran against the exact local preview checkout. Actual provider responses were fetched through the environment's HTTP proxy and fulfilled in Chromium for live tile checks. Separate runs used tile fixtures for repeatable interaction and fault injection. Phone and tablet checks used touch and viewport emulation. Physical iPhone, iPad, and Safari testing remain open.

| Area | Result |
| --- | --- |
| Initial map and basemaps | Street opened with 26 markers. Verified rendered imagery for Street, Antique, Satellite and Terrain relief, with matching active controls. |
| Hiking overlay | Verified a live Waymarked Trails tile, keyboard activation, attribution, persistence when switching basemaps, and removal on toggle-off. The control is disabled in the Leaflet fallback. |
| Collection and search | Initial results and search are collapsed. Subject buttons show site counts and update list and pins together. Verified category counts of 11 expositions, 2 forts, 10 mound sites, 2 civic and rail sites, and 1 estate. Browse all showed 26 sites. Search opened with focus, handled whitespace and empty results, and restored the map when closed. |
| Buttons and dossiers | Verified Surprise me with a deterministic selection, site selection and coordinates, dossier tabs, Connections navigation, and calculator recalculation. |
| Notebook | Saved a site, reloaded, reopened it from Notebook, removed it, and reloaded again. The prior saved-entry `textContent` error did not recur. Normal test paths emitted no uncaught JavaScript exceptions. Exceptional storage still fails as shown below. |
| Responsive layout | Tested iPhone portrait 390 × 844 and landscape 844 × 390, and tablet portrait 810 × 1080 and landscape 1080 × 810. No horizontal document overflow. Touch selection opened the right dossier. |
| Fallback | Blocking Leaflet activated the SVG map and subject filtering reduced its visible pins. Blocking tile requests while leaving Leaflet available produced the blank-basemap case below. |

## Remaining issues

| Priority | Finding | How to reproduce |
| --- | --- | --- |
| High | Malformed or inaccessible notebook storage stops startup. | Store malformed JSON such as `{broken` in `hiddenAtlasSaved02` and reload. An uncaught `SyntaxError` leaves zero markers. Simulating an inaccessible `localStorage` raises a `SecurityError` with the same result. The storage read and parse happen before map initialization. |
| High | All ten mound dossiers generate unrelated archive searches. | Open Photos & plans for any mound site. All five visible URLs search for `undefined` because the records have no archive terms. Checked every mound dossier. |
| Medium | Tile-only outages have no visible failure state. | Block tile requests while allowing Leaflet. The default map retains 26 pins, displays the normal Street note, and never activates the SVG fallback. This can also leave the hiking overlay looking empty during a provider outage. |
| Medium | The calculator accepts invalid or missing inputs. | Set Workers on site to zero. The duration becomes `Infinity`. The ten mound records also open Build estimate with an empty floor area because they lack a size value. |
| Low | A hidden Library of Congress link returns HTTP 404. | Inspect the generated Photos & plans markup for `/resource/sn83045462/?fo=json`. A visible Chicago Library of Congress search timed out in this environment, so its reachability remains unverified. USGS, Leaflet attribution, and the tested Wikimedia search returned HTTP 200. |

The earlier whitespace-search issue, notebook selection exception, and fallback filtering mismatch were fixed in the feature stack and passed the retest. The original GitHub Pages URL still returns HTTP 404. Vercel is the active preview path.

## Deployment

Two Vercel projects have previously built this repository, `the-hidden-atlas` and `the-hidden-atlas-btm9`. Use the branch commit to identify the correct preview. The generated preview URLs use Vercel Authentication. Browser checks in this report used the local static server. Notebook entries are browser- and origin-specific, so they do not follow a user from one preview hostname to another.

The feature stack and release archives represent different versions of the app. Use the preview tied to the audited commit for review, and update release archives explicitly when releasing. Before a production merge, address the notebook startup failure and archive query defects, then do a device check in Safari. Confirm external tile and CDN availability in the target environment.
