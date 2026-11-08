---
name: pairwise-feature-correlation-analysis
description: Use when a Compound Discoverer (v3.3) peak table retains adduct, dimer,
  or in-source-fragment features co-eluting with identified lipids and pairwise correlation
  filtering is needed to reduce spurious unidentified peaks.
when_to_use_negative:
- Peak table has already been filtered for adducts and dimers, so applying correlation
  filtering would remove legitimate primary lipid features.
- Fewer than 10 samples are available, making pairwise correlation estimates unreliable
  under the stated requirement.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: LipiDex 2
  role: Provides the Peak Finder module that performs the pairwise correlation filtering
    between unidentified and identified features
  repo: https://github.com/coongroup/LipiDex-2
- name: Compound Discoverer
  role: Generates the input peak table (.cdResult or .csv) that Peak Finder consumes
    for correlation filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:25aaeacff9e29c077e362d29dcf20859b5909b701fa3e09b6bcb5ff006f68761
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - Compound Discoverer
  - LipiDex 2
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pairwise-feature-correlation-analysis

## Summary

Computes Pearson/Spearman correlations between unidentified features and identified lipid features within shared FWHM retention-time windows to remove adducts, dimers, and other spurious co-eluting features, as implemented in LipiDex 2 Peak Finder.

## When to use

Use when a Compound Discoverer (v3.3) peak table retains adduct, dimer, or in-source-fragment features co-eluting with identified lipids and pairwise correlation filtering is needed to reduce spurious unidentified peaks.

## When NOT to use

- Peak table has already been filtered for adducts and dimers, so applying correlation filtering would remove legitimate primary lipid features.
- Fewer than 10 samples are available, making pairwise correlation estimates unreliable under the stated requirement.

## Inputs

- Peak table from LipiDex 2 Peak Finder, sourced from a Compound Discoverer 3.3 .cdResult file or a converted .csv peak table
- Retention-time (FWHM) information for each feature
- Sample-level intensities for identified lipid features and unidentified features (at least 10 samples required)

## Outputs

- Filtered peak table retaining only features not flagged as adducts, dimers, or other spurious species (e.g. filtered_peak_features.csv)
- Post-filter feature counts for unidentified and identified features

## How to apply

Load the peak table into LipiDex 2 Peak Finder (reading Compound Discoverer .cdResult or .csv input). Configure pairwise correlation between unidentified and identified features using Pearson or Spearman correlation computed within a shared FWHM retention-time window. Require at least 10 samples for correlation and apply a 0.99 correlation cutoff to flag features correlated with primary lipid features as adducts, dimers, or other spurious species. Run the filtering step and extract post-filter counts; in the reprocessed Nile rat plasma dataset (MSV000091033) this reduced 14826 unidentified features and 779 lipid identifications to 13898 unidentified features and 719 identified features.

## Related tools

- **LipiDex 2** (Provides the Peak Finder module that performs the pairwise correlation filtering between unidentified and identified features) — https://github.com/coongroup/LipiDex-2
- **Compound Discoverer** (Generates the input peak table (.cdResult or .csv) that Peak Finder consumes for correlation filtering)

## Examples

```
Run LipiDex2.exe → Peak Finder → load .cdResult peak table → set pairwise correlation (Pearson/Spearman, 0.99 cutoff, ≥10 samples) → export filtered_peak_features.csv
```

## Evaluation signals

- Post-filter unidentified feature count and identified lipid count match the reported reduction (e.g. 14826→13898 unidentified; 779→719 identified).
- Every removed feature co-elutes with an identified lipid feature within the same FWHM retention-time window and exceeds the 0.99 correlation cutoff.
- At least 10 samples contributed intensity values to each pairwise correlation used for filtering.
- Output table contains only features classified as primary lipid features or uncorrelated unidentified peaks.

## Limitations

- Requires a Compound Discoverer 3.3 .cdResult file or equivalent .csv peak table as input to Peak Finder.
- Correlation estimates require at least 10 samples per pairwise comparison.
- The 0.99 Pearson/Spearman cutoff is strict; features with high but sub-threshold co-elution will be retained as spurious peaks.
- LipiDex 1 peak tables and mzMine 2 inputs are not supported by the LipiDex 2 Peak Finder correlation-filtering workflow.

## Evidence

- [results] Applying Peak Finder filtering reduced 14826 unidentified features and 779 lipid identifications to 13898 unidentified features and 719 identified features by removing adducts, dimers, and other spurious features.: [evidence span withheld — anchor status: not_found]
- [readme] LipiDex automatically filters peak tables for adduct peaks, in-source fragments, and dimers: [evidence span withheld — anchor status: external_doc]
