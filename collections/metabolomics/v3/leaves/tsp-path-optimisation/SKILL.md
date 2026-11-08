---
name: tsp-path-optimisation
description: Use when a detected/filtering-surviving list of sample coordinates (e.g.
  from macroMS sample finding over colonies or droplets) must be ordered into a visit
  sequence to minimise total stage travel before generating a device geometry file.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective in that
  regime, so use the microMS cell-targeting workflow instead of the macroMS macroscopic
  ordering pipeline
- Coordinates that have not yet passed target filtering — ordering debris and merged/broken
  regions wastes stage travel and inflates the target count beyond the ~10,000 limit
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: macroMS
  role: Web platform whose device coordinate generation step consumes the ordered
    target coordinates to produce geometry files for MALDI-TOF acquisition
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Source of the optically guided targeting workflow and scikit-image feature
    finding from which macroMS's target list is derived
- name: scikit-image
  role: Python library used upstream to detect connected feature regions and compute
    the morphological descriptors feeding the coordinate list
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:76147f56926fb7003cb96164e59929e9fd3ae37119753faa29e85cb96a23f6dd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  - scikit-image
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tsp-path-optimisation

## Summary

Orders detected sample-target coordinates by solving a traveling salesman problem over pairwise stage-travel distances, so that optically guided MALDI-TOF MS acquisitions (macroMS workflows) traverse all targets with minimal stage movement.

## When to use

Use when a detected/filtering-surviving list of sample coordinates (e.g. from macroMS sample finding over colonies or droplets) must be ordered into a visit sequence to minimise total stage travel before generating a device geometry file.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective in that regime, so use the microMS cell-targeting workflow instead of the macroMS macroscopic ordering pipeline
- Coordinates that have not yet passed target filtering — ordering debris and merged/broken regions wastes stage travel and inflates the target count beyond the ~10,000 limit

## Inputs

- List of detected, filter-passing target coordinates (pixel and/or stage mm positions with per-target morphological measurements such as area, circularity, nearest-neighbor distance)
- Upper bound on target count (macroMS-reported ~10,000 targets maximum)

## Outputs

- TSP-optimised visit ordering (valid permutation of the detected target coordinates)
- Custom geometry/target file consumed by the MS device for spectrum acquisition at the ordered locations (JSON target list plus ordered coordinates in the macroMS workflow)

## How to apply

Take the surviving target coordinates that passed macroMS's population filtering rules (accepted size range, acceptable circularity, and expected nearest-neighbor spacing). Compute a pairwise stage-travel distance matrix over those coordinates and feed it to a standard TSP solver to obtain the visiting order that minimises total stage travel; this ordering is what macroMS's device coordinate generation step consumes to emit the custom geometry files loaded into the MS device. Sanity-check that the returned ordering is a valid permutation of all detected targets — no targets dropped or duplicated. Compare total travel of the TSP order against an unsorted ordering to confirm an actual reduction. Keep the ordering step downstream of target filtering, since removing debris/merged regions before optimisation avoids wasting stage travel on invalid targets.

## Related tools

- **macroMS** (Web platform whose device coordinate generation step consumes the ordered target coordinates to produce geometry files for MALDI-TOF acquisition) — github.com/macroMS/macroMS_web
- **microMS** (Source of the optically guided targeting workflow and scikit-image feature finding from which macroMS's target list is derived)
- **scikit-image** (Python library used upstream to detect connected feature regions and compute the morphological descriptors feeding the coordinate list)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(xy, xy, metric='euclidean')  # pairwise stage-travel distances over filter-passing target coords, then solve TSP for visit order
```

## Evaluation signals

- Returned ordering is a valid permutation: it contains every filter-passing target exactly once (no drops, no duplicates)
- Total path length of the TSP order is reduced relative to the unsorted (original detection) ordering over the same coordinates
- Detected target list matches the expected target count/locations on the calibration/sample image before ordering
- Ordering is computed only over targets surviving the size, circularity, and nearest-neighbor-spacing filters
- Target count stays within the reported ~10,000 upper limit for the platform

## Limitations

- Platform upper limit of ~10,000 targets is reported without measurement details or uncertainty; larger target sets are untested in the article
- Comparison to microMS is qualitative — no quantitative benchmark of travel-time gains is provided
- Optical distortion/imperfect perspective in scanner or phone-camera images must first be corrected (macroMS optical correction tools) or ordered coordinates will not map accurately to device stage positions

## Evidence
