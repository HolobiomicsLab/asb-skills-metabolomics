---
name: morphological-population-filtering
description: Use when macroMS has detected candidate targets in an optical image (flatbed
  scanner or cell phone camera) and the raw region list must be filtered by per-region
  area, circularity, and nearest-neighbor distance before MS targeting.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective for these
- The target list is already curated/validated and no debris or merged regions are
  present in the detected regions
- Feature finding itself (thresholding/inversion/connected-region detection) has not
  yet been run — filtering operates on detected regions, not raw images
tools:
- name: macroMS
  role: Web platform exposing the target-filtering functionality via a simple web
    interface
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Source code reused by macroMS to implement the feature-finding/filtering image
    analysis
- name: scikit-image
  role: Python imaging library implementing thresholding, inversion, connected-region
    detection, and morphological descriptors used by the filter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:a13d4a7cb6c48313ee06683232fd4a27e5aee9bbc8404ad3112a5b15fed209d2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  - scikit-image
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# morphological-population-filtering

## Summary

Filters the detected target list from macroMS's scikit-image-based feature finder by applying population-level morphological rules — size, circularity, and nearest-neighbor spacing — so debris and merged/broken regions are discarded before device coordinate generation.

## When to use

Use when macroMS has detected candidate targets in an optical image (flatbed scanner or cell phone camera) and the raw region list must be filtered by per-region area, circularity, and nearest-neighbor distance before MS targeting.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective for these
- The target list is already curated/validated and no debris or merged regions are present in the detected regions
- Feature finding itself (thresholding/inversion/connected-region detection) has not yet been run — filtering operates on detected regions, not raw images

## Inputs

- Detected connected-region list from scikit-image feature finding on the sample optical image (regions plus area/size, circularity, nearest-neighbor distance descriptors)
- Expected sample population parameters (accepted size range, acceptable circularity, expected nearest-neighbor spacing)

## Outputs

- Filtered target list with coordinates and per-target morphological measurements (JSON)
- Discarded region set (debris, merged/broken regions) excluded from downstream TSP ordering and device coordinate generation

## How to apply

After scikit-image thresholding, inversion, and connected-region detection produce candidate regions, compute per-region morphological descriptors (area/size, circularity, and nearest-neighbor distances) using the microMS source code that macroMS reuses. Apply the paper's population filtering rules: retain only regions within the accepted size range, acceptably circular, and consistent with the expected nearest-neighbor spacing of the sample population, discarding debris and merged or broken regions. Thresholds are set relative to the expected population of objects (e.g. yeast colonies or nL droplets from E. coli / P. fluorescens arrays) rather than fixed constants. Emit the surviving targets with their coordinates and per-target morphological measurements as JSON, which downstream TSP ordering and device-coordinate generation consume.

## Related tools

- **macroMS** (Web platform exposing the target-filtering functionality via a simple web interface) — github.com/macroMS/macroMS_web
- **microMS** (Source code reused by macroMS to implement the feature-finding/filtering image analysis)
- **scikit-image** (Python imaging library implementing thresholding, inversion, connected-region detection, and morphological descriptors used by the filter)

## Evaluation signals

- Detected-and-filtered target list matches the expected target count and locations on the sample image
- Retained regions all fall within the accepted size range, circularity, and expected nearest-neighbor spacing
- Known debris and merged/broken regions from the raw detection are absent from the filtered output
- Each surviving target carries its per-target morphological measurements alongside coordinates in the JSON artifact
- Downstream TSP visit ordering is a valid permutation of the filtered targets only

## Limitations

- Upper limit of ~10,000 targets was reported without measurement details or uncertainty
- Comparison to microMS is qualitative only — no quantitative filtering benchmark is given
- Reliability demonstrated at the 300 μm sample scale; smaller samples should use microMS instead
- No macroMS version information is stated for the filtering functionality

## Evidence
