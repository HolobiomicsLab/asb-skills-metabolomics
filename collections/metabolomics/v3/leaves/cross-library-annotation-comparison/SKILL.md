---
name: cross-library-annotation-comparison
description: Use when you have one aligned MS/MS feature list (e.g., an MS-Dial peak
  table) and need to quantify how many features each candidate spectral library can
  annotate, to choose or justify a library.
when_to_use_negative:
- Input is already a single annotation result and no library comparison is needed
- Libraries cannot be exported to a common format compatible with the search tool
  (FragHub supports .MSP, .MGF, .JSON, .CSV, .XML)
- The feature list has not been filtered for blanks and reproducibility, so unfiltered
  noise inflates all annotation counts non-comparably
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS-Dial
  role: Runs the MS/MS spectral search of the aligned feature list against each library
    (.MSP) with default parameters and reports per-library annotation counts
- name: FragHub
  role: Produces the unified, deduplicated (.MSP/.MGF/.JSON/.CSV/.XML) library that
    serves as the union reference being benchmarked against the individual libraries
  repo: github.com/eMetaboHUB/FragHub
- name: MS-CleanR
  role: Applied before benchmarking to filter the raw feature list (blank ratio, mass
    defect, RSD) so that per-library annotation counts are computed on a clean, comparable
    feature set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:b60a94c65aa76a96ff8d93a74b4118a0dd0f04b5a32b2e5e27469a55d1493589
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - MS-CleanR
  - MS-Dial
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Cross-library annotation comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for feature-level MS/MS annotation in which an identical aligned feature list is searched against each candidate spectral library independently and per-library annotation counts are tabulated to quantify coverage differences. In the FragHub paper this benchmark compared FragHub's unified library against individual libraries (MassBank, GNPS, MoNA, MS-Dial-VS17) on the 435-feature A. thaliana PI-mode feature list.

## When to use

Use when you have one aligned MS/MS feature list (e.g., an MS-Dial peak table) and need to quantify how many features each candidate spectral library can annotate, to choose or justify a library.

## When NOT to use

- Input is already a single annotation result and no library comparison is needed
- Libraries cannot be exported to a common format compatible with the search tool (FragHub supports .MSP, .MGF, .JSON, .CSV, .XML)
- The feature list has not been filtered for blanks and reproducibility, so unfiltered noise inflates all annotation counts non-comparably

## Inputs

- Aligned MS/MS feature list from MS-Dial feature detection and alignment (e.g., 435 PI-mode A. thaliana features after MS-CleanR filtering with blank ratio 0.8, RSD 40%, relative mass defect 50–3500 ppm)
- Spectral libraries to compare, each exported in a common search format (FragHub unified library, MassBank, GNPS, MoNA, MS-Dial-VS17)
- Chromatography/ionization context for each library (FragHub separates spectra by polarity — positive/negative — and LC/GC mode)

## Outputs

- Per-library annotated-feature counts (integer number of features each library annotates)
- Single aggregated annotation-result table (CSV) comparing the number of annotated features across libraries
- Per-feature library hit metadata (compound identity from the matched library entry)

## How to apply

Export each processed spectral library in a common, search-tool-compatible format (the paper used .MSP for FragHub-processed OMSLs). Run spectral search of the same aligned feature list against each library with identical search parameters — in the paper, MS/MS-based annotation in MS-Dial v5.23 with default MS-Dial parameters for every library. Record per-library counts of annotated features, not annotated spectra, to make results comparable across libraries of different sizes (e.g., FragHub unified library vs. individual GNPS/MoNA/MassBank/MS-Dial-VS17). Aggregate per-library counts into a single comparison table (CSV). Validate by checking that library-specific counts match expected or reported values (FragHub = 102 features, MassBank = 41, others in the 55–81 range) and that the union library annotates ≥ any individual library, since it contains all their spectra after deduplication via SPLASH keys.

## Related tools

- **MS-Dial** (Runs the MS/MS spectral search of the aligned feature list against each library (.MSP) with default parameters and reports per-library annotation counts)
- **FragHub** (Produces the unified, deduplicated (.MSP/.MGF/.JSON/.CSV/.XML) library that serves as the union reference being benchmarked against the individual libraries) — github.com/eMetaboHUB/FragHub
- **MS-CleanR** (Applied before benchmarking to filter the raw feature list (blank ratio, mass defect, RSD) so that per-library annotation counts are computed on a clean, comparable feature set)

## Examples

```
MS-Dial MS/MS search: import aligned PI-mode feature table (.txt), select library = FragHub_unified.msp, run MS/MS annotation with default parameters; repeat for MassBank.msp, GNPS.mgf, MoNA.msp, MS-Dial-VS17.msp; collect counts into annotation_comparison.csv
```

## Evaluation signals

- Per-library annotated-feature counts should be reproducible: the paper reports FragHub = 102 (24% of 435 features), MassBank = 41, and GNPS/MoNA/MS-Dial-VS17 in the 55–81 range
- The unified (FragHub) library's annotation count must be ≥ the count of any single constituent library, since it contains all their spectra after deduplication
- Annotated-feature counts (not annotated-spectra counts) should be compared across libraries, since library sizes differ (e.g., MoNA: 21,839 compounds / 190,359 spectra; MassBank: 8,358 compounds / 164,261 spectra)
- The output must be a single CSV table with one row (or column) per library, using identical search parameters across all searches
- Each library must be pre-split/filtered by ionization mode and chromatography type (e.g., PI-mode spectra for a positive-mode feature list) before searching

## Limitations

- Annotation percentage is low relative to natural product diversity: 32,193 unique compounds in the integrated library vs. millions of estimated natural products, so a library may under-annotate even on a curated feature list
- Redundancy across OMSLs can cause inconsistent scores with dot-product / reverse dot-product metrics that are sensitive to fragment number and intensity, biasing cross-library comparisons
- Collision energy harmonization is not addressed (about 70 different nonstandardized formats), which may cause unequal annotation success across libraries
- Chemical class disparity between OMSLs and natural product catalogues means some features remain unannotated regardless of library choice; orthogonal strategies are needed

## Evidence

- [other] Run MS/MS-based spectral annotation of the aligned feature list using each FragHub-processed OMSL exported in .MSP format with default MS-Dial parameters.: [evidence span withheld — anchor status: not_found]
- [other] Per-library annotated-feature counts must reproduce the paper's reported values (FragHub = 102, MassBank = 41, GNPS/MoNA/MS-Dial-VS17 in the 55-81 range).: [evidence span withheld — anchor status: not_found]
