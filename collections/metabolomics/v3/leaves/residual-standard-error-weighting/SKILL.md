---
name: residual-standard-error-weighting
description: Use when aligning two same-method nontargeted LC-MS datasets (Source→Target)
  whose RT/mz drift must be modeled per feature dimension, and candidate matches must
  be filtered with a dataset-specific RSE-scaled tolerance rather than fixed ppm/RT
  windows.
when_to_use_negative:
- Datasets were acquired with different LC-MS methods — Eclipse explicitly targets
  'two or more same-method nontargeted LCMS datasets'
- Features have not been drift-corrected; Blueshift drift correction via pooled technical
  replicates and internal standards is expected to precede alignment
- You need redundant-feature clustering rather than cross-dataset alignment — Gravity
  clusters redundant LCMS features based on RT and correlation
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: Eclipse
  role: 'Aligns the datasets: fits LOWESS RT/mz scalers, computes RSE, applies the
    ±6 RSE cutoff, and performs penalty-score best-match assignment'
  repo: https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA with Compound_ID, RT, MZ, Intensity)
    and Feature Abundances pivot consumed by the RSE-weighted matching
  repo: https://github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:c1d5a452b8f1631f16ec2a7d14df564b00fce7b9f785d6045d94e775d6d18e6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# residual-standard-error-weighting

## Summary

Weight inter-dataset retention-time, m/z, and intensity distances by the Residual Standard Error (RSE) of survey-based LOWESS scalers so that a ±6 RSE cutoff identifies candidate feature matches between LC-MS datasets during Eclipse subalignment.

## When to use

Use when aligning two same-method nontargeted LC-MS datasets (Source→Target) whose RT/mz drift must be modeled per feature dimension, and candidate matches must be filtered with a dataset-specific RSE-scaled tolerance rather than fixed ppm/RT windows.

## When NOT to use

- Datasets were acquired with different LC-MS methods — Eclipse explicitly targets 'two or more same-method nontargeted LCMS datasets'
- Features have not been drift-corrected; Blueshift drift correction via pooled technical replicates and internal standards is expected to precede alignment
- You need redundant-feature clustering rather than cross-dataset alignment — Gravity clusters redundant LCMS features based on RT and correlation

## Inputs

- Merged Feature Metadata + Feature Abundances table (bmxp.FMDATA joined with Compound_ID x injection_id pivot) for each of the DS1–4 plasma datasets
- Shared features between the Source and Target datasets (used to fit the LOWESS RT and m/z scalers)

## Outputs

- Per-pair subalignment match tables, e.g. subalignment_matches_<source>_vs_<target>.csv, with source and target Compound_ID, RT, MZ, RSE-scaled distance, and penalty rank
- One-to-one best-match assignments between Source and Target Compound_IDs within the ±6 RSE cutoff

## How to apply

For each Source→Target dataset pair, merge Feature Metadata (bmxp.FMDATA) with the Feature Abundances pivot (Compound_ID x injection_id), then fit survey-based inter-dataset RT and m/z scalers via LOWESS on the shared features between the merged tables. Compute the Residual Standard Error for each LOWESS fit and use it to normalize the residual distances so the tolerance adapts to per-dataset scaling quality. Apply the ±6 RSE cutoff to retain candidate Target features whose RSE-weighted RT, m/z, and Intensity distances fall within tolerance of the Source feature. Rank retained candidates per Source feature with the best-match penalty ranking and assign one-to-one Source↔Target Compound_ID matches, writing each pairwise subalignment match table to CSV with bmxp shared column headers (Compound_ID, RT, MZ) so headers can be remapped globally if needed.

## Related tools

- **Eclipse** (Aligns the datasets: fits LOWESS RT/mz scalers, computes RSE, applies the ±6 RSE cutoff, and performs penalty-score best-match assignment) — https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- **bmxp** (Provides the shared schema (bmxp.FMDATA with Compound_ID, RT, MZ, Intensity) and Feature Abundances pivot consumed by the RSE-weighted matching) — https://github.com/broadinstitute/bmxp

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner; aligner = MSAligner(); matches = aligner.align(source_merged, target_merged)  # LOWESS RT/mz scalers -> RSE-weighted ±6 RSE cutoff -> penalty-ranked one-to-one matches
```

## Evaluation signals

- Each subalignment_matches_<source>_vs_<target>.csv exists and contains the required bmxp FMDATA fields (Compound_ID, RT, MZ)
- Assigned best matches are one-to-one between Source and Target Compound_IDs (expert review of one-to-one assignment)
- All recorded matches satisfy the ±6 RSE cutoff on RSE-weighted RT, m/z, and Intensity distances
- LOWESS scalers were fit only on shared features between the two merged Source/Target tables, and each fit's RSE was computed and used to weight residuals

## Limitations

- Eclipse handles 'complex matching scenarios that arise from n > 2 datasets' via a graph-based approach; RSE-weighted pairwise subalignments alone do not resolve the global n>2 assignment
- RT in bmxp.FMDATA is 'Unitless retention time, may or may not be scaled', so RSE cutoffs are scale-dependent and must be interpreted relative to the fitted scalers
- BMXP modules are standalone; no version numbers or automated pipeline guarantee that the RSE-weighted step was run with consistent parameters across runs

## Evidence

- [abstract] Graph-based matching for n>2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] Merging FMDATA + Feature Abundances required by Eclipse: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Shared schema and header remapping: "All BMXP modules use a shared schema and file formats with our prefered columns headers."
- [readme] FMDATA defines the matched feature fields: "* `Compound_ID` - Index, Project-unique feature ID (a bit of a misnomer) * `RT` - Unitless retention time, may or may not be scaled * `MZ` - Unsigned mass-to-charge ratio"
- [readme] Eclipse scope: same-method nontargeted LCMS datasets: "Align two or more same-method nontargeted LCMS datasets"
