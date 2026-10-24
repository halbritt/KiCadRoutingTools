# AGENTEE-513: stock v0.23.7 esp_prog baseline

The stock grouped-placement and routing flow completed, but its board failed native DRC and the frozen connector intent. Seed 5's final board has **0 native unconnected items, 7 DRC errors and 303 warnings**. The errors are three clearances against U2's unnetted copper artwork and four solder-mask bridges. Its placement also leaves USB1 nearest the north edge, despite the declared west edge. Preserve this as a failed candidate.

Source is stock tag v0.23.7, commit `83646c8b56c3571067c359cf5524167abfef4c0f`. The assigned branch started at empty worker-marker commit `42181060587dff660bd3bb08baf16274515cc510`; both have source tree `5343c603ab68eb18c1bf7d50010ea28d4d088749`. No KRT code, installed runtime or consumer pin was changed. This commit adds only this report.

The input is `tests/fixtures/959/run29_pile.kicad_pcb` with its own `.kicad_pro`: 21 footprints, 77 pads, 24 populated nets, two copper layers and a valid outline. Its project retains 0.20 mm tracks, 0.60/0.30 mm vias and 0.15 mm clearance. The selected functional intent is the tagged `959/asbuilt/esp_prog.design-brief.json`: a west-facing Micro-B receptacle and east-side target headers. The older direct-USB-A briefs and zone-plan experiments are retained separately and explicitly reconciled in the evidence; their contradictory declarations are not merged.

Stock `place_simple.py` ran seeds 0-5 scattered and 6-11 clustered, serially, with its default objective. Stock `judge_seeds.py` quick-routed seeds 5, 6, 7, 9, 0 and 11 and selected 5. The full route starts from its original placement: GND pour on B.Cu, `/D_P`/`/D_N` differential routing, then all remaining nets. Every routing step uses `--no-fix-drc-settings --escalation board`. All output projects remain byte-identical to the corresponding original project. Routing preserves the placed footprint poses, locks, outline and pad-net assignments.

| Phase | Wall seconds | CPU seconds (user + system) |
|---|---:|---:|
| Placement, 12 seeds | 152.25 | 147.92 |
| Judging, six seeds | 46.02 | 45.72 |
| Final flow, including placed checks | 9.48 | 9.32 |
| Separate routing-only control | 7.23 | 7.06 |

The control uses `run25/esp_prog_placed` with its own different project rules. It has 0 native opens, 3 DRC errors and 310 warnings. It is separate from the unplaced baseline and does not isolate the effect of placement. The end-to-end selected placement already had one clearance error and three mask bridges before routing; the control placement already had one clearance error. The routed candidates have additional findings, not a clean DRC result.

The final end-to-end board SHA-256 is `915045d554f402df95c7a4512c32b4b60e427721bbd30ad2087e9c6cdf4ce7a1`. The routing-only board SHA-256 is `3200b0035defe3dafc15ba2d1fd54aa32d14a5723d0553c178f97515a25dc73a`.

All computation was local, admitted at weight 1, with three-CPU affinity and a 12 GiB per-process address-space limit. An aggregate memory cgroup cap was not established; the run stayed single-worker. Python 3.12.3, KiCad/pcbnew 10.0.7 and an offline build of the tagged Rust crate were used. Setup failures and a cancelled placement launch that lacked visible Shapely are retained separately from the measured baseline.

Validation ran through native admission:

```text
python3 tests/run_all.py -j 1 --timeout 120 1327_judge_seeds 1296_judge_floorplan 1300_place_simple_estimated_courtyards
3 passed, 0 failed, 0 timed out, 0 skipped (+0 self-skipped) in 14.8s

python3 -m pytest tests/test_1327_judge_seeds.py tests/test_1296_judge_floorplan.py -q
10 passed, 1 warning in 8.25s
```

Full-suite pytest was not run under the assignment's no-broad-corpus limit. The stock tree has no Makefile, so the wrapper's `make determinism` and `make smoke` targets are unavailable. Byte determinism was not measured. Native DRC used `--refill-zones`; no schematic parity or manufacturing acceptance is claimed. Four via-in-pad sites reported by the differential stage require separate fabrication review.

The sealed evidence is at `/home/halbritt/.cache/agent-ee-evidence/agentee-513-esp-prog-baseline/`: `RESULT.md`, frozen originals and manifests, commands, timings, native reports, every candidate board, logs, compiled tool identity and `SHA256SUMS` without a self-entry. Coordinator agent-530 arranges the requested Fable HIGH 20 review. This baseline is not a pod acceptance substitute or a superiority claim.
