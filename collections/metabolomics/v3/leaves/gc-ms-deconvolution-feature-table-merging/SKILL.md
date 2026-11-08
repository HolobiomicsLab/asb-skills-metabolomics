---
name: gc-ms-deconvolution-feature-table-merging
description: Use when you have deconvoluted GC-MS feature tables from MSHub/GNPS (e.g.
  GNPS_GC release) from parallel acquisition modes that must be combined and filtered
  for reliable annotations before spatial or comparative analysis.
when_to_use_negative:
- Input is already a single merged feature table — re-merging headspace and liquid-injection
  tables would duplicate features keyed on the same deconvoluted identity.
- The analysis is untargeted raw GC-MS spectral processing rather than post-deconvolution
  feature-table curation — use the MSHub/GNPS auto-deconvolution workflow instead.
- Features have not been deconvoluted (no balance score or cosine similarity columns
  exist); the 50% balance / >0.9 cosine filters cannot be applied to raw spectra.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Source of the deconvolution/molecular networking workflows and the deconvoluted
    feature tables that are merged and filtered
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSHub
  role: Machine-learning auto-deconvolution engine that produced the feature tables
    (via unsupervised non-negative matrix factorization) with balance scores and cosine
    similarities
- name: GNPS_GC companion repository
  role: Hosts the headspace and liquid-injection deconvolution feature tables with
    body-location coordinates used as inputs
  repo: https://github.com/bittremieux/GNPS_GC
- name: pandas
  role: DataFrame loading, merging, and threshold filtering of the feature tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:5109f134d064af5158c3c64e87d4d20cb3c9999c13191c968e487e1d1970d94b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - '''ili'
  - GNPS
  - GNPS_GC companion repository
  - MSHub
  - pandas
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gc-ms-deconvolution-feature-table-merging

## Summary

Merge GNPS/MSHub auto-deconvoluted GC-MS feature tables (e.g. headspace and liquid-injection analyses of body-location samples) into one table, then filter by balance score ≥50% and cosine similarity >0.9 to retain reproducible deconvoluted features for downstream visualization.

## When to use

Use when you have deconvoluted GC-MS feature tables from MSHub/GNPS (e.g. GNPS_GC release) from parallel acquisition modes that must be combined and filtered for reliable annotations before spatial or comparative analysis.

## When NOT to use

- Input is already a single merged feature table — re-merging headspace and liquid-injection tables would duplicate features keyed on the same deconvoluted identity.
- The analysis is untargeted raw GC-MS spectral processing rather than post-deconvolution feature-table curation — use the MSHub/GNPS auto-deconvolution workflow instead.
- Features have not been deconvoluted (no balance score or cosine similarity columns exist); the 50% balance / >0.9 cosine filters cannot be applied to raw spectra.

## Inputs

- Headspaces GNPS deconvolution feature table with body-location coordinates (from github.com/bittremieux/GNPS_GC release)
- Liquid-injection GNPS deconvolution feature table (from github.com/bittremieux/GNPS_GC release)
- Deconvoluted feature identity keys shared between the two tables

## Outputs

- filtered_feature_table.csv — merged table retaining only features with balance ≥50% and cosine >0.9
- Per-body-location intensity profiles for target compounds (e.g. squalene, hexanoic acid, globulol, phenylene dibenzoate) ready for 'ili rendering

## How to apply

Load the headspace and liquid-injection GNPS deconvolution feature tables from the GNPS_GC release (github.com/bittremieux/GNPS_GC) into pandas DataFrames, keyed on deconvoluted feature identity, then merge them into a single combined feature table. Filter merged rows to those with a balance score of at least 50% — the article's metric for reproducibility of fragmentation patterns across all samples — and spectral cosine similarity above 0.9, which together retain only features whose deconvolution was consistent across acquisition modes. The rationale is that MSHub performs auto-deconvolution via unsupervised non-negative matrix factorization, so strict balance/cosine thresholds guard against unreliable compound fragment patterns before intensity profiles (e.g. for squalene, hexanoic acid, globulol, phenylene dibenzoate per body location) are extracted for mapping in 'ili. Export the filtered table (filtered_feature_table.csv) as the handoff artifact for downstream spatial mapping.

## Related tools

- **GNPS** (Source of the deconvolution/molecular networking workflows and the deconvoluted feature tables that are merged and filtered) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSHub** (Machine-learning auto-deconvolution engine that produced the feature tables (via unsupervised non-negative matrix factorization) with balance scores and cosine similarities)
- **GNPS_GC companion repository** (Hosts the headspace and liquid-injection deconvolution feature tables with body-location coordinates used as inputs) — https://github.com/bittremieux/GNPS_GC
- **pandas** (DataFrame loading, merging, and threshold filtering of the feature tables)

## Examples

```
import pandas as pd; feat = pd.merge(pd.read_csv('headspace_features.csv'), pd.read_csv('liquid_features.csv'), on='feature_id'); filt = feat[(feat['balance_score']>=50) & (feat['cosine']>0.9)]; filt.to_csv('filtered_feature_table.csv', index=False)
```

## Evaluation signals

- Merged table row count equals the union of deconvoluted feature identities from both headspace and liquid-injection tables (no dropped or duplicated feature keys).
- Every row in the output satisfies balance score ≥ 50% AND cosine similarity > 0.9 — verify with a single invariant check across all filtered rows.
- Spot-check known volatiles: squalene should show intensity on all body locations but less on the feet, matching the reported spatial distribution.
- Intensity profiles for squalene, hexanoic acid, globulol, and phenylene dibenzoate are extractable per body location, sufficient to rebuild Figure 2f–i in 'ili.
- filtered_feature_table.csv exports without NaN balance/cosine columns, confirming filter columns survived the merge.

## Limitations

- The balance score quantifies reproducibility of fragmentation patterns across all samples, so features absent in one acquisition mode (headspace vs liquid injection) may be filtered out even if genuinely present.
- Cosine >0.9 and balance ≥50% are thresholds demonstrated for the human volatilome body-mapping use case; other datasets may need recalibrated cutoffs.
- The Fast Fourier Transform-based strategy addresses scalability limits of deconvolution, but very large raw GC-MS datasets must still be deconvolved upstream before this merging step.

## Evidence

- [readme] GNPS_GC repository provenance for input tables: [evidence span withheld — anchor status: not_found]
