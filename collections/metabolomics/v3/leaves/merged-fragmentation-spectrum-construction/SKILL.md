---
name: merged-fragmentation-spectrum-construction
description: Use when processing LC-MS/MS runs (mzML/mzXML) where a feature (precursor
  ion) was fragmented at multiple collision energies and you need a single consolidated
  MS/MS spectrum per feature before formula and structure annotation.
when_to_use_negative:
- Input LC-MS runs contain profiled-mode spectra — these are discarded in the COSMIC
  pipeline, so centroid conversion is needed first
- The analysis requires separate per-collision-energy spectra (e.g. evaluating a 20-eV-only
  condition) rather than consolidated per-feature spectra
- Fragmentation spectra contain at most one peak (or the compound yields such a spectrum
  at any collision energy/noise level) — the compound is removed from all libraries
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Collects fragmentation spectra and precursor information, detects features,
    and merges spectra assigned to the same feature via cosine-distance agglomerative
    clustering before de novo formula determination
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Consumes the merged spectra to compute confidence scores and estimate FDR;
    merged spectra yielded 1,701 correct annotations at FDR 5% versus 1 for spectral
    library search
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates using fragmentation trees computed from the merged
    spectra
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:3c17befeec9df7c0d817117b0d978e350a750d7b935619410906f112c11f77c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID 1.2.0
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# merged-fragmentation-spectrum-construction

## Summary

Construct per-feature merged fragmentation spectra by grouping all fragmentation spectra assigned to the same precursor-ion feature via agglomerative clustering on cosine distance, as implemented in SIRIUS 4 during COSMIC processing. Merging improves both annotation rates and confidence-score separation compared with individual collision-energy spectra.

## When to use

Use when processing LC-MS/MS runs (mzML/mzXML) where a feature (precursor ion) was fragmented at multiple collision energies and you need a single consolidated MS/MS spectrum per feature before formula and structure annotation.

## When NOT to use

- Input LC-MS runs contain profiled-mode spectra — these are discarded in the COSMIC pipeline, so centroid conversion is needed first
- The analysis requires separate per-collision-energy spectra (e.g. evaluating a 20-eV-only condition) rather than consolidated per-feature spectra
- Fragmentation spectra contain at most one peak (or the compound yields such a spectrum at any collision energy/noise level) — the compound is removed from all libraries

## Inputs

- LC-MS/MS runs in .mzML or .mzXML format (centroided, not profiled mode)
- Per-scan fragmentation spectra with precursor information and collision energies
- Feature tables linking precursor ions, adducts and isotope peaks to fragmentation spectra

## Outputs

- One merged fragmentation spectrum per detected feature (precursor ion)
- Merged spectra suitable for SIRIUS 4 formula determination and CSI:FingerID ranking
- Per-condition comparison counts (merged vs. individual collision energy, e.g. 20 eV / 40 eV) used for FDR 5% benchmarking

## How to apply

As the second step of SIRIUS 4 preprocessing, first collect all fragmentation spectra with their precursor information and associate features (precursor, adduct, isotope peaks). Then merge fragmentation spectra assigned to the same feature using an agglomerative clustering algorithm based on cosine distance, so spectra from different collision energies (e.g. 20/40 eV runs or merged conditions) consolidate into one spectrum per compound. Remove fragmentation spectra with at most one peak before merging, and discard compounds entirely if any collision energy/noise level yields a spectrum with at most one peak. The rationale, from the COSMIC benchmark, is that merged spectra yielded the best annotation rates and confidence-score separation (usually followed by 40-eV spectra), and in the independent QTOF evaluation at FDR 5% merged spectra gave 1,701 correct COSMIC annotations versus 1 for spectral library search. Feed the merged spectra onward into SIRIUS 4 de novo formula determination (10 ppm mass accuracy) and CSI:FingerID structure ranking.

## Related tools

- **SIRIUS 4** (Collects fragmentation spectra and precursor information, detects features, and merges spectra assigned to the same feature via cosine-distance agglomerative clustering before de novo formula determination) — https://github.com/sirius-ms/sirius
- **COSMIC** (Consumes the merged spectra to compute confidence scores and estimate FDR; merged spectra yielded 1,701 correct annotations at FDR 5% versus 1 for spectral library search) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates using fragmentation trees computed from the merged spectra) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Each merged spectrum maps 1:1 to a detected feature (precursor/adduct/isotope-peak group); no precursor should remain unmerged or duplicated
- Merged spectra must show consistent performance gains: annotation rates and confidence-score separation should exceed the best single collision energy (usually 40 eV)
- Benchmark invariant from the independent QTOF evaluation: merged-spectra COSMIC annotations at exact FDR 5% should reach 1,701 correct hits vs 1 for spectral library search
- Any fragmentation spectrum with at most one peak is removed before merging; compounds failing at any collision energy/noise level are discarded entirely
- Output spectra must be derivable from centroided .mzML/.mzXML input only

## Limitations

- Merging aggregates spectra across collision energies, so per-energy fragmentation behavior is not preserved for downstream condition-specific analyses
- Annotation quality from merged spectra improves averages but small structure modifications remain hard or impossible to tell apart using MS/MS data alone
- The merged-spectrum advantage was measured on reference/benchmark data (e.g. MassBank, GNPS, NIST 2017); FDR for biological datasets cannot be guaranteed given the inability to accurately estimate FDR from fragmentation spectra

## Evidence

- [discussion] Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance.: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra.: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] SIRIUS first collects all fragmentation spectra and their precursor information and then searches for features that are associated with those fragmentation spectra: "SIRIUS first collects all fragmentation spectra and their precursor information and then searches for features that are associated with those fragmentation spectra"
- [readme] SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments: "SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments"
