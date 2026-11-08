---
name: feature-table-matching-across-acquisition-modes
description: Decision support for quantifying coverage overlap between full-scan,
  RegFilter-scheduled precursor lists, and DDA MS/MS fragmented features in LC-MS/MS
  metabolomics datasets.
when_to_use_negative:
- Input is already a matched/aligned feature table where SPL-vs-DDA overlap classification
  has been computed
- When the goal is de novo annotation or molecular networking of MS2 spectra (use
  GNPS FBMN instead), not acquisition coverage auditing
- When full-scan and MS/MS features were not preprocessed with the same MZmine settings
  (e.g. matching Join aligner tolerances), making m/z/RT matching unreliable
tools:
- name: MZmine
  role: Supplies the full-scan and MS/MS feature tables and defines the alignment
    tolerances (0.001 Da or 15 ppm; RT 0.2 min) used for matching SPL and DDA features
- name: RegFilter
  role: Produces the filtered scheduled precursor list (135/219/150 features for BRA006/BRA010/BRA177)
    whose overlap with DDA MS/MS is being quantified
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:0415a1c06b75dd642cc9b288d34f1637481a2f29ad728a5fdf4c2b2d19e1f0c0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - MZmine
  - RegFilter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# feature-table-matching-across-acquisition-modes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns MZmine-preprocessed full-scan MS1 feature tables against scheduled precursor list (SPL) and DDA MS/MS feature tables using Join aligner tolerances to classify features as common, list-only, or DDA-only, quantifying what the stochastic DDA acquisition missed.

## When to use

Decision support for quantifying coverage overlap between full-scan, RegFilter-scheduled precursor lists, and DDA MS/MS fragmented features in LC-MS/MS metabolomics datasets.

## When NOT to use

- Input is already a matched/aligned feature table where SPL-vs-DDA overlap classification has been computed
- When the goal is de novo annotation or molecular networking of MS2 spectra (use GNPS FBMN instead), not acquisition coverage auditing
- When full-scan and MS/MS features were not preprocessed with the same MZmine settings (e.g. matching Join aligner tolerances), making m/z/RT matching unreliable

## Inputs

- MZmine full-scan feature table (.csv with row ID, row m/z, row retention time, peak area)
- MZmine DDA MS/MS feature table
- RegFilter-filtered scheduled precursor list (SPL) per strain

## Outputs

- Overlap counts: common features, list-only features, per strain
- list_only_ion_fractions.csv with per-strain fraction of list-only ions that were full-scan base peaks

## How to apply

Load the MZmine-exported full-scan .csv feature tables (row ID, row m/z, row retention time, peak area) and the DDA MS/MS feature tables for each strain. Match RegFilter SPL precursors against the DDA fragmented precursor set using MZmine Join aligner tolerances (m/z tolerance 0.001 Da or 15 ppm; retention time tolerance 0.2 min) to recover the common features, then flag list-only ions — SPL features present in DDA MS1 but lacking MS2 fragmentation. Export the overlap counts and per-strain list-only fractions to a CSV such as list_only_ion_fractions.csv; the rationale is to expose the stochastic nature of DDA, where ions of similar intensities may or may not be fragmented.

## Related tools

- **MZmine** (Supplies the full-scan and MS/MS feature tables and defines the alignment tolerances (0.001 Da or 15 ppm; RT 0.2 min) used for matching SPL and DDA features)
- **RegFilter** (Produces the filtered scheduled precursor list (135/219/150 features for BRA006/BRA010/BRA177) whose overlap with DDA MS/MS is being quantified) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
MZmine batch: Join aligner (m/z tolerance 0.001 or 15 ppm, RT tolerance 0.2) on full-scan vs DDA tables; export list_only_ion_fractions.csv
```

## Evaluation signals

- Common feature counts equal 76 (BRA006), 135 (BRA010), and 77 (BRA177) when SPL is joined against DDA fragmented precursors
- List-only feature counts equal 59 (BRA006), 84 (BRA010), and 73 (BRA177)
- Computed per-strain base-peak fractions of list-only ions equal 33.9% (BRA006), 22.4% (BRA010), 29.7% (BRA177)
- Matched features fall within Join aligner tolerances (m/z 0.001 Da or 15 ppm; RT 0.2 min)
- SPL feature totals match RegFilter output: 135 (BRA006), 219 (BRA010), 150 (BRA177)

## Limitations

- DDA fragmentation is stochastic: ions with similar intensities may or may not be fragmented, so list-only status is not a detection failure of the feature itself
- Ions with low m/z are more frequently captured by the SPL than fragmented by DDA, biasing overlap interpretation by mass range
- Matching depends on consistent MZmine preprocessing parameters across full-scan and MS/MS tables

## Evidence
