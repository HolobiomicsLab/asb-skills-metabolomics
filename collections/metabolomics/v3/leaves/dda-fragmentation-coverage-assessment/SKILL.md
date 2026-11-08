---
name: dda-fragmentation-coverage-assessment
description: Use when you have MZmine feature tables from a full-scan run and a DDA
  run of the same strains, plus a RegFilter-generated SPL, and you need to quantify
  which SPL ions were not fragmented by DDA.
when_to_use_negative:
- The input is a raw instrument file (.raw or .mzXML) that has not yet been preprocessed
  by MZmine into aligned feature tables with row ID, row m/z, row retention time,
  and peak area columns.
- No RegFilter SPL or other targeted precursor list exists — without a candidate list
  there is no 'list-only' set to compare against the DDA fragmented precursors.
- The goal is annotation or molecular networking (FBMN, SIRIUS, ChemWalker) rather
  than assessing DDA fragmentation coverage of a precursor list.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine
  role: Preprocesses full-scan and MS/MS data into feature tables (.csv with row ID,
    row m/z, row retention time, peak area) and aligns SPL precursors against DDA
    fragmented precursors using Join aligner tolerances (m/z 0.001 Da or 15 ppm; RT
    0.2 min)
- name: RegFilter
  role: Generates the scheduled precursor list (SPL) whose fragmentation coverage
    is being assessed; selects features meeting filtering requirements from full-scan
    MZmine output
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:c3c9facf7b5fb8ebf756f4cc8e6e01a60ce7585ec19cc0b3cfad7b2a54e142a0
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
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# dda-fragmentation-coverage-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well a targeted scheduled precursor list (SPL) from RegFilter overlaps with what data-dependent acquisition (DDA) actually fragmented, by aligning MZmine full-scan and MS/MS feature tables and computing per-strain fractions of list-only ions, including how many were full-scan base peaks that DDA missed.

## When to use

Use when you have MZmine feature tables from a full-scan run and a DDA run of the same strains, plus a RegFilter-generated SPL, and you need to quantify which SPL ions were not fragmented by DDA.

## When NOT to use

- The input is a raw instrument file (.raw or .mzXML) that has not yet been preprocessed by MZmine into aligned feature tables with row ID, row m/z, row retention time, and peak area columns.
- No RegFilter SPL or other targeted precursor list exists — without a candidate list there is no 'list-only' set to compare against the DDA fragmented precursors.
- The goal is annotation or molecular networking (FBMN, SIRIUS, ChemWalker) rather than assessing DDA fragmentation coverage of a precursor list.

## Inputs

- MZmine-preprocessed full-scan feature table (.csv with row ID, row m/z, row retention time, peak area) per strain
- MZmine-preprocessed DDA MS/MS feature table (.csv) per strain
- RegFilter-filtered scheduled precursor list (SPL) per strain (BRA006: 135, BRA010: 219, BRA177: 150 features)

## Outputs

- Per-strain count of common features between SPL and DDA fragmented precursors (e.g. 76 BRA006, 135 BRA010, 77 BRA177)
- Per-strain count of list-only ions (59 BRA006, 84 BRA010, 73 BRA177)
- list_only_ion_fractions.csv with per-strain percentages of list-only ions that were full-scan base peaks missed by DDA (33.9% BRA006, 22.4% BRA010, 29.7% BRA177)

## How to apply

Load the MZmine-preprocessed full-scan feature tables (.csv with row ID, row m/z, row retention time, peak area) and the DDA MS/MS feature tables for each strain, together with the RegFilter-filtered features defining the SPL (135 for BRA006, 219 for BRA010, 150 for BRA177). Align SPL precursors against the DDA fragmented precursor set using Join aligner tolerances (m/z tolerance 0.001 Da or 15 ppm; retention time tolerance 0.2 min) to recover the common features (76 BRA006, 135 BRA010, 77 BRA177). Flag the list-only ions present in DDA MS1 but lacking MS2 fragmentation (59 BRA006, 84 BRA010, 73 BRA177). For each list-only ion, query its full-scan peak area to determine whether it was the base peak (highest-intensity ion) at its retention point. Compute and report per-strain fractions of list-only ions that were full-scan base peaks missed by DDA (33.9% BRA006, 22.4% BRA010, 29.7% BRA177). The rationale is that DDA's stochastic nature leaves many detected ions unfragmented, and this assessment exposes that coverage gap.

## Related tools

- **MZmine** (Preprocesses full-scan and MS/MS data into feature tables (.csv with row ID, row m/z, row retention time, peak area) and aligns SPL precursors against DDA fragmented precursors using Join aligner tolerances (m/z 0.001 Da or 15 ppm; RT 0.2 min))
- **RegFilter** (Generates the scheduled precursor list (SPL) whose fragmentation coverage is being assessed; selects features meeting filtering requirements from full-scan MZmine output) — https://github.com/computational-chemical-biology/regression_filter

## Evaluation signals

- The computed per-strain fractions of list-only ions that were base peaks missed by DDA must equal 33.9% (BRA006), 22.4% (BRA010), and 29.7% (BRA177).
- The count of common features between SPL and DDA fragmented precursors must match 76 (BRA006), 135 (BRA010), and 77 (BRA177) when using Join aligner tolerances of m/z 0.001 Da or 15 ppm and RT 0.2 min.
- The count of list-only ions must match 59 (BRA006), 84 (BRA010), and 73 (BRA177) for each strain.
- list_only_ion_fractions.csv must be produced with per-strain rows and percentage values consistent with the reported fractions.
- All list-only ions must be present in the DDA MS1 full-scan table but absent from the DDA fragmented precursor set (i.e. no MS2 spectra).

## Limitations

- DDA fragmentation is stochastic — ions with similar intensities may or may not be fragmented — so the list-only set reflects acquisition mode behavior, not just feature properties.
- Ions with low m/z were more frequently captured by the SPL than fragmented by DDA, so the assessment may be skewed toward low-mass ions.
- The assessment assumes full-scan feature tables were preprocessed separately from MS/MS tables in MZmine and aligned with the specified Join aligner tolerances; different tolerances change the common and list-only sets.

## Evidence
