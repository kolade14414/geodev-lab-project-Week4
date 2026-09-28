# Week 4: the analysis

> **Synthetic data.** Every number below was computed by `scripts/week4_coverage_analysis.py`
> on the illustrative dataset in `data/raw/`. See `data/README.md`.

**Question.** Which planning zones are more than 100 km from any candidate CO2 storage hub?

**What I expected, written down before running anything:** most central and southern zones
covered, one or two peripheral zones with gaps, perhaps two or three zones flagged
(more than 30 percent of their area beyond 100 km).

## The operation

Buffer the storage hubs by 100 km, dissolved into one shape. Then find which zones have
area falling outside it.

1. **Vector, Geoprocessing Tools, Buffer.** Input: storage hubs. Distance: 100,000 metres.
   Dissolve result: ticked. Saved as `hub_buffer_100km.gpkg`.
2. **Vector, Geoprocessing Tools, Difference.** Input: zones. Overlay: the buffer. This leaves
   only the parts of each zone not covered.
3. Calculated the area of what remained, and the percentage of each zone.

The same steps run in Python in `scripts/week4_coverage_analysis.py`.

## The result

Three zones have more than 30 percent of their area beyond 100 km of any hub:

| Zone | Uncovered area | Uncovered share |
|---|---|---|
| Z12 | 5,994 km² | 99.9% |
| Z08 | 2,396 km² | 39.9% |
| Z11 | 2,121 km² | 35.3% |

All three are in the north-east corner of the study area. Seven zones are fully covered.
Z09 (3.5%) and Z10 (8.9%) have small gaps and are not flagged.

![Coverage map](outputs/coverage_map.png)

## The checks

*Looked at the map.* The uncovered areas sit where I expected, on the periphery, in the
corner furthest from all five hubs.

*Checked the row count.* 12 zones in, **5 out**. That is not a mistake, but it is not what
I assumed. Difference drops any feature that is completely covered by the overlay, so the
seven fully covered zones (Z01 to Z07) disappear instead of returning zero area. Reconciled:
5 returned + 7 dropped = 12. The script asserts this. If I had joined the result back to the
zone list without noticing, the covered zones would have shown as blank instead of 0%.

*Checked one by hand.* Took the worst zone, Z12, and measured from its centre to the nearest
hub. 147.6 km, well beyond 100. The south-west corner of the zone is roughly 98 km from the
nearest hub, which matches the 0.1 percent sliver that is covered.

*Looked for empty geometry.* None came back. Because Difference dropped the fully covered zones,
there were no zero-area leftovers to find. This check would matter if the tool kept them.

*Then the boundary cases.* There were no emitters within 5 km of the buffer edge, so no
facility is close to flipping in or out. The zone flags are a different story. I re-ran the
whole analysis at other buffer distances:

| Buffer | Zones flagged |
|---|---|
| 80 km | Z08, Z10, Z11, Z12 |
| 90 km | Z08, Z11, Z12 |
| **100 km** | **Z08, Z11, Z12** |
| 110 km | Z12 |
| 120 km | Z12 |

Z12 is flagged at every distance. Z08 and Z11 are not. They drop out between 100 and 110 km,
so those two flags depend on the exact distance I chose. If I had picked 110 km, the answer
would have been one zone, not three.

## What the emitters say

Of 41 emitters, one sits beyond 100 km: E18, a gas processing plant, 138.3 km from the
nearest hub, in Z12. It reports 0.72 Mt CO2 per year, which is 2.9 percent of the 24.4 Mt
reported in total. 11 of the 41 emitters report no CO2 figure at all, so that percentage is
a floor, not a measurement.

## What I would do next
- Replace straight-line distance with pipeline route distance (needs a road or right-of-way layer).
- Chase the 11 missing CO2 figures before quoting any emissions share.
- Add storage capacity to the hubs. A hub within 100 km is useless if it holds 1 Mt.

## Files
| File | What it is |
|---|---|
| `outputs/zone_coverage.csv` / `.gpkg` | Area and uncovered percentage for all 12 zones |
| `outputs/hub_buffer_100km.gpkg` | The dissolved buffer |
| `outputs/coverage_map.png` | Map above |
| `outputs/summary.json` | Every number quoted in this write-up |
