# Action plan: Blick Mead aurochs least-cost study

Draft, 4 October 2026. This plan follows `PROTOCOL-for-Claude.md` (handoff) and `PROTOCOL-draft.docx` (Draft v0.1; the text matches the handoff). It describes what I will do and in what order. **Nothing here has been run.**

## 0. Gate: what must happen before any analysis

The handoff's first rule overrides everything. I do not start Phase 2 onwards until Tim confirms **both**:

1. Part A is approved in writing.
2. Decisions D1 to D8 (A10) are settled.

Until then I may only do Phase 1 (read-only preparation that touches no data).

**Questions for Tim (Phase 0)**

| # | Ask | Why |
|---|---|---|
| Q1 | Approve Part A and answer D1 to D8. My suggested defaults are the proposals in the protocol: D1 W0/W1/W2 at 2x; D2 360 runs; D3 10 m with a 5 m subset; D4 test H4 as a separate Late Neolithic section; D5 Mesolithic targets only for H3; D6 thresholds as written; D7 Claude builds, Tim runs; D8 Tim gives the V5 verdict. | Rule 1 |
| Q2 | Write your own expectation for H1 to H4 into A6 before approval. | A6 says Tim must do this before approval |
| Q3 | Share `_analyse_sarsen_lcp_blind.py`, `SARSEN_analysis_PREREGISTRATION.md`, `SARSEN_ROUTES_data_spec_v5.md` and `SARSEN_RESULTS.md`. | Rule 2. They are on `D:\` and I cannot see them from this session. |
| Q4 | Where does this run? The handoff says Tim's laptop (`D:\Projects\aurochs-hunt\`, about 7 GB RAM). This session is a Linux cloud container with only `README.md` and `index.html` in the repo. | Decides where the code is built and who runs the sweep |
| Q5 | Is outbound access to the B1 hosts (EA, OS Data Hub, Overpass, HE ArcGIS, BGS, Internet Archaeology) allowed from wherever I run? | Needed for data download |

## Issues I noticed in the protocol (for Tim to rule on, not for me to fix silently)

- **Pilot barrier bug.** A2 says barrier variants gave identical funnel numbers, "possibly a bug". The dial-effect check (A5) and test B5.5 cover this, but I will also look for the cause in the pilot code if Tim shares it.
- **Inconsistent W0/W1/W2 note.** A3 says `w_wood` is fixed at 1.0 (doubling flat wooded ground) and D1 asks about 2x or 3x. 3x would mean `w_wood` = 2.0. D1 must fix this number before lock.
- **Compute size.** 1,080 solves on 1.69 M nodes with 16-neighbour directed edges is roughly 27 M edges per graph. Memory (4 GB cap) and time are unknown. B4 requires measuring the first three runs before the full sweep; I will do that and report before going further.
- **Cost term `(1 - p_v*V)` only applies with `p_v` > 0**, so the 180 `p_v` = 0 runs used for V1 to V4 are those with no valley discount. `V` still affects nothing in them, which is the intent.
- **Pilot paths in A2** (`/workspace/projects/aurochs-hunt/`) differ from the build path in B0 (`D:\Projects\aurochs-hunt\`). I will treat the pilot as read-only context.
- **Data licence caveat.** BGS 50K vectors need a licence; the plan uses the WMS for visual checks only, as written.
- **Pit coordinates.** Only six pits have published coordinates; the other 14 are digitised, with RMS no more than 5 m as the acceptance test.

## Phase 1. Preparation (allowed before approval; no data analysis)

1. Create the project skeleton: `src/`, `data/raw/`, `data/derived/`, `FROZEN/`, `out/page/`, `out/checks/`, `docs/`.
2. Read the sarsen solver and pre-registration once shared. Write a short mapping table: sarsen function, kept or changed, and why (per B0 items 1 to 5).
3. Draft the file layout and function signatures only. No run.

## Phase 2. After approval: lock the protocol (A7 step 1)

1. Write `params_locked.json` (the 360-run grid, fixed values, sensitivity values, thresholds, grid frame, all decisions D1 to D8 as answered).
2. Write the scoring code and the sealed-file schema. Unit-test the scorer on **synthetic data only**.
3. `git init` locally, commit, tag `protocol-v1`, record SHA-256 hashes. **No push.**

## Phase 3. Data build (B1 to B3)

1. Download the 169 EA DTM 1 km tiles (E 406000 to 419000, N 136000 to 149000). Block-average to 10 m, EPSG:27700, row 0 at N 149000. Fill nodata and report the amount. Save float32 `.npy` and a 16-bit PNG.
2. Download OS Open Rivers, OSM water and roads, OS Open Roads, NHLE polygons (1012126, 1009133, 1010140), HE AIM, OS Terrain 50 (for the 10 km margin), BGS 625k.
3. Build masks and layers: inpaint (Laplace plus two binomial passes) with the stated buffers; river mask (one connected component at 10 m); epsilon-filled surface; D8 flow and thalwegs at 0.5 km²; `WET`, `WOOD` (W0 to W2), `BAR` (30 m buffer), `V`, `A`.
4. Starts: 250 m lattice with the A4.4 rules. Targets T1, T2 (30 m around OSM 6960129398), T3 (Durrington interior clipped at N 143500).
5. Pit and post geometry go **only** into `hypotheses_SEALED.json` (Avenue centreline, elbow, ridge crest point, forced waypoints, posts, pits). Georeference IA69 Figure 1 from the six control points; report RMS and leave-one-out error; compare with pilot (RMS 1.78 m; mean LOO 2.55 m) and flag differences over 15 m.
6. Run B5 tests 1 to 3 and report the numbers.

## Phase 4. Solver (B0, B4)

1. Directed 16-neighbour graph (queen plus knight moves; knight slope from the two half-steps sampled at the midpoint). Per-edge cache of `d`, `up`, `dn`, `X`.
2. Per-run edge cost from the A3 formula (cell terms as the mean of the two end cells). Dijkstra via `scipy.sparse.csgraph` on the reversed graph, multi-source from each target set, so one solve gives every start's cost and path.
3. Corridors: trace every start's path, resample to 10 m, count cells, take the top 10% as the run's flow corridor. Corridor-frequency maps across runs. 2%, 5% and 10% cost bands for west starts to T2.
4. Tests B5.4 to B5.8 (edge costs, dial effects including the barrier transect, seal enforced by `grep`, 10 m sampling, corridor sanity).
5. **Run the first three runs only. Report time and peak RAM.** Stop and ask Tim if the full sweep will not fit in about 4 GB.

## Phase 5. Frozen sweep (A7 step 3)

1. Run all 360 runs by 3 target sets (1,080 solves), plus the sensitivity runs listed in A3 (water attraction 0 and 0.4; crossing cost 0 and 1,000), raw-DEM run, 1 km edge margin run, 24-run subset at 5 m, and full grid at 25 m and OS Terrain 50.
2. Write `FROZEN/` (run table, corridor PNGs with JSON georeference sidecars, frequency maps, bands) and `FROZEN.json` with timestamp, parameter hash and per-file SHA-256.
3. The solver never reads `hypotheses_SEALED.json`.

## Phase 6. Validity checks (A5), still sealed

1. V1 to V4 on the 180 `p_v` = 0 runs, with numbers.
2. Six V5 check images for Tim (corridor-frequency map; lowest, middle and highest climb settings, over a 1 m hillshade with thalwegs). **Tim records his verdict before anything is unsealed (D8).**
3. Dial-effect report; any inert dial reported with evidence that it is not a bug.
4. Coarse-DEM bias: six transects (1 m against 10 m gradient, flag above 3 percentage points), 5 m subset, 25 m and Terrain 50 overlap (Jaccard) and whether any verdict changes.
5. If V1 to V4 fail or Tim rejects V5: stop, write a dated amendment (stating that it followed a failed check), re-lock, rerun from Phase 5. Do not open the seal.

## Phase 7. Unseal and score (A7 steps 5 and 6)

1. `score_hypotheses.py` refuses to run unless `FROZEN.json` hashes match.
2. Score H1 to H4 per run with margins, then verdicts: robust (80% or more of relevant runs), conditional (20% to 80%, with the settings it needs), none (below 20%). H2 is reported separately for barrier-off and barrier-on runs. H3 counts T1/T2 only; T3 is context. H4 includes the 25 m and 50 m radii, the all-20-pits set, the alternative null and the 1,000 jitter draws.
3. Write `RESULTS/hypotheses.json`. Report every result, including failed predictions, against the expectations Tim recorded.

## Phase 8. Documentation and local page (B6, B7)

1. `docs/METHOD.md` and `docs/RESULTS.md` in plain British English with no em-dashes, each number traceable to a file. Check images at 1,000 px or more in `out/checks/`.
2. `index.html` dials page: plain JavaScript, no libraries or CDNs, works from `file://`, 25 MB maximum, under 300 MB browser memory. Controls: climb (5 steps), side-slope, valley floor, wet ground, woodland, spur barrier, target. Opens at climb 40 / side-slope off / valley off / wet 5 / W1 / barrier off, labelled "analyst judgement, not a model output". Banner "Draft, local only" plus all source credits. Hypothesis overlays and the H1 to H4 panel only after unsealing.
3. **Do not push, deploy or publish.** Tim tests locally and tells me when.

## Standing rules I will follow throughout

- Open data only, public URLs from B1; never invent a coordinate; approximate items flagged `"approximate": true`; unknown values `null`.
- Pilot numbers are provisional context, never results.
- Every acceptance test (B5) is reported with numbers.
- Part A is not edited after approval; changes are dated amendments.
- Plain British English, no em-dashes.

## Timeline (indicative)

The protocol estimates 1.5 to 2.5 hours of build once approved, plus compute. The download of 169 tiles, the first-three-runs benchmark and the full sweep are the uncertain parts, and I will give real figures after the benchmark.
