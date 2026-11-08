---
name: inchikey-unique-count-aggregation
description: Use when ms1-id annotations from multiple LC-MS modes must be reduced
  to unique-metabolite counts (distinct InChIKey strings) per mode and pooled, e.g.
  to benchmark annotation yield on a public cohort dataset.
when_to_use_negative:
- Input data already contain data-dependent MS/MS scans — standard MS/MS library search,
  not ms1-id pseudo-spectrum generation, is the appropriate route
- The goal is per-feature or per-sample annotation output rather than a cohort-level
  unique-metabolite tally; stop at the aligned feature table instead of deduplicating
  by InChIKey
- Data are MS imaging (imzML) — different cutoffs apply (minimum spectral usage 0.05
  instead of 0.20), so the LC-MS count aggregation rules do not transfer directly
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: Generates pseudo MS/MS spectra from MS1-only features, performs reverse spectral
    search against the indexed GNPS library, and emits annotated feature tables whose
    InChIKey column is deduplicated and counted per mode
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend (v1.0.11) used by ms1-id for feature detection and isotope peak assignment;
    isotope peaks flagged by MassCube are excluded from pseudo MS/MS generation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:e7d3c14f542b6ebfee5075a5fa995778c746016fd3df368f263eede24dc17c6f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - MassCube
  - ms1-id
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# inchikey-unique-count-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates ms1-id level 2/3 structural annotations of full-scan LC-MS data into per-mode and overall counts of unique metabolites by counting distinct InChIKey strings, used to reproduce the IBD stool cohort benchmark (3010/293/227/636; total 3802).

## When to use

Use when ms1-id annotations from multiple LC-MS modes must be reduced to unique-metabolite counts (distinct InChIKey strings) per mode and pooled, e.g. to benchmark annotation yield on a public cohort dataset.

## When NOT to use

- Input data already contain data-dependent MS/MS scans — standard MS/MS library search, not ms1-id pseudo-spectrum generation, is the appropriate route
- The goal is per-feature or per-sample annotation output rather than a cohort-level unique-metabolite tally; stop at the aligned feature table instead of deduplicating by InChIKey
- Data are MS imaging (imzML) — different cutoffs apply (minimum spectral usage 0.05 instead of 0.20), so the LC-MS count aggregation rules do not transfer directly

## Inputs

- ms1-id annotation result files per LC-MS mode (e.g. aligned_feature_table.tsv) containing InChIKey, precursor m/z, and retention time per annotated feature
- Full-scan LC-MS mzML/mzXML files from the 546-sample IBD stool dataset (Metabolomics Workbench PR000639) across four modes (HILIC positive, HILIC negative, C8 positive, C18 negative)

## Outputs

- Per-mode counts of unique InChIKey strings at level 2/3 confidence (expected: 3010 HILIC positive, 293 HILIC negative, 227 C8 positive, 636 C18 negative)
- Total unique annotated metabolites across all modes (expected: 3802)
- Counts table saved as ibd_ms1_annotation_counts.csv

## How to apply

Run ms1-id (with the MassCube v1.0.11 backend) on each LC-MS mode separately: correlate features within a ±1.5 s retention-time window requiring at least 4 consecutive shared MS1 scans with Pearson correlation ≥ 0.80, excluding MassCube-assigned isotope peaks, then perform precursor-tolerant reverse spectral search of the pseudo MS/MS spectra against the indexed GNPS MS/MS library (July 17, 2024) using square-root-transformed, mass-dependently peak-scaled reference spectra (k = 10). Apply the LC-MS annotation cutoffs — minimum reverse cosine score 0.7, minimum 3 matched peaks, minimum spectral usage 0.20 — keeping the top 1 hit per unique precursor m/z. Link the surviving annotations back to metabolic features via retention time and precursor mass, then deduplicate by InChIKey string: count unique InChIKeys within each mode (HILIC positive, HILIC negative, C8 positive, C18 negative) and across the union of all modes. Aggregate the per-mode and total counts into a table (e.g. ibd_ms1_annotation_counts.csv) for comparison against expected values.

## Related tools

- **ms1-id** (Generates pseudo MS/MS spectra from MS1-only features, performs reverse spectral search against the indexed GNPS library, and emits annotated feature tables whose InChIKey column is deduplicated and counted per mode) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend (v1.0.11) used by ms1-id for feature detection and isotope peak assignment; isotope peaks flagged by MassCube are excluded from pseudo MS/MS generation)

## Examples

```
ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- Per-mode unique InChIKey counts match the published values: 3010 (HILIC positive), 293 (HILIC negative), 227 (C8 positive), 636 (C18 negative)
- Pooled unique InChIKey count across the four modes equals 3802 (i.e. cross-mode duplicates are collapsed, not summed: 3010+293+227+636 > 3802)
- All retained annotations pass the cutoffs: reverse cosine ≥ 0.7, ≥ 3 matched peaks, spectral usage ≥ 0.20, top-1 hit per unique precursor m/z
- Every annotation carries a non-empty InChIKey string before deduplication; count of distinct InChIKey strings per mode is reproducible across reruns
- Results table ibd_ms1_annotation_counts.csv contains one row per mode plus the total, with the four LC-MS mode labels

## Limitations

- Counts depend on the specific GNPS library version indexed (July 17, 2024); a different library build or version will change the annotation counts
- Counts reflect level 2/3 confidence only (no level 1 standard-confirmed annotations) and depend on the reverse-matching cutoffs; loosening or tightening 0.7 / 3 peaks / 0.20 spectral usage changes the tally
- The ms1-id package version itself is not stated in the paper (only MassCube ver 1.0.11 and GNPS2 workflow version 2024.09.20 are given), so exact reproduction may require pinning the PyPI release

## Evidence

- [discussion] Applying ms1-id to the 546-sample IBD stool dataset in HILIC positive, HILIC negative, C8 positive, and C18 negative modes yielded 3010, 293, 227 and 636 unique metabolites: [evidence span withheld — anchor status: not_found]
- [readme] README: annotation results file name: [evidence span withheld — anchor status: external_doc]
- [readme] README: indexed GNPS library requirement and download: [evidence span withheld — anchor status: external_doc]
- [readme] README: IBD dataset accession: [evidence span withheld — anchor status: external_doc]
