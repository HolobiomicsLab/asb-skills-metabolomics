---
name: fold-change-based-feature-selection
description: Use when you have an xcms-derived feature table with mz, rt, and group-defined
  intensity columns for two conditions (e.g. mutant vs WT exo-metabolome) and need
  to isolate high-abundance, significantly enriched features.
when_to_use_negative:
- Input is raw LC/MS data still requiring feature detection and alignment — run the
  xcms module in Metaboseek first to obtain a feature table
- Goal is molecular formula prediction or structural annotation (SIRIUS, Seven Golden
  Rules filters, CAMERA) rather than comparative statistics between groups
- The analysis requires multiple-testing-corrected significance (this workflow explicitly
  uses unadjusted p-values)
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Metaboseek
  role: Interactive platform providing the Feature Table Actions / FTAnalysisParam
    comparative statistics module used to compute fold changes and unadjusted p-values
    and to apply the intensity, significance, and fold-change filters
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Upstream feature detection and alignment of LC/MS data that produces the feature
    table this filtering skill consumes
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used with xcms to load xcms results into Metaboseek as a Feature Table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:8600e4405e93437f203b5b9a95c6d13bbe75431590e21b17f10d2d9d6c27fd67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fold-change-based-feature-selection

## Summary

Selects metabolomic features enriched between two sample groups from an xcms/Metaboseek feature table by stacking an intensity cutoff (top 12%), an unadjusted significance cutoff (p < 0.05), and a fold-change cutoff (≥5-fold enrichment), as used to uncover >1000 features enriched in the adult hacl-1 exo-metabolome versus WT.

## When to use

Use when you have an xcms-derived feature table with mz, rt, and group-defined intensity columns for two conditions (e.g. mutant vs WT exo-metabolome) and need to isolate high-abundance, significantly enriched features.

## When NOT to use

- Input is raw LC/MS data still requiring feature detection and alignment — run the xcms module in Metaboseek first to obtain a feature table
- Goal is molecular formula prediction or structural annotation (SIRIUS, Seven Golden Rules filters, CAMERA) rather than comparative statistics between groups
- The analysis requires multiple-testing-corrected significance (this workflow explicitly uses unadjusted p-values)

## Inputs

- xcms-derived feature table for hacl-1(tm6725) vs WT adult OP50 exo-metabolome comparison with `mz` and `rt` columns
- Group-defined intensity columns labeling hacl-1 and WT sample groups

## Outputs

- Filtered feature table (.csv, e.g. hacl1_op50_enriched_features.csv) with fold-change, intensity, and significance columns
- Count of retained features (expected >1000 for the hacl-1 adult comparison)

## How to apply

Load the xcms feature table into Metaboseek, ensuring the `mz` and `rt` columns plus group-defined intensity columns are present. In the Feature Table Actions analysis, compute group-wise mean intensities, fold changes (mutant/WT), and unadjusted p-values across the two sample groups with Metaboseek's comparative statistics module (FTAnalysisParam-style analysis, controlGroup = WT, no p-value adjustment). Sequentially apply three filters: retain only features in the top 12% of intensities to remove low-abundance noise, retain features with unadjusted p < 0.05 between genotypes, and retain features at least 5-fold enriched in the mutant relative to WT. Export the surviving features with fold-change, intensity, and significance columns as a .csv (e.g. hacl1_op50_enriched_features.csv) via Metaboseek's Save Table function.

## Related tools

- **Metaboseek** (Interactive platform providing the Feature Table Actions / FTAnalysisParam comparative statistics module used to compute fold changes and unadjusted p-values and to apply the intensity, significance, and fold-change filters) — https://github.com/mjhelf/Metaboseek
- **xcms** (Upstream feature detection and alignment of LC/MS data that produces the feature table this filtering skill consumes) — https://github.com/sneumann/xcms
- **MSnbase** (Used with xcms to load xcms results into Metaboseek as a Feature Table)

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- Exported hacl1_op50_enriched_features.csv exists and parses as a comma-separated table containing fold-change, intensity, and significance fields
- Number of retained features exceeds 1000, matching the >1000 features reported as enriched in the hacl-1 adult exo-metabolome
- Every retained feature has hacl-1/WT fold change ≥ 5 and unadjusted p < 0.05
- Retained feature intensities all fall in the top 12% of the remaining feature table
- Reported group-wise statistics match an FTAnalysisParam-style run with controlGroup = WT and no p-value adjustment

## Limitations

- Significance is unadjusted (no multiple-testing correction), so enriched-feature counts may include false positives
- The intensity cutoff (top 12%) is a noise-removal heuristic tuned to this comparison; retuning changes the feature count
- Metabolite annotation following this selection is limited to confidence Level 3 (tentative structure or compound class assignments only), and many detected features remain unidentified

## Evidence

- [other] Applying intensity (top 12% of remaining features), unadjusted significance (p < 0.05), and 5-fold-change filters uncovered more than 1000 features enriched in the hacl-1 adult exo-metabolome.: [evidence span withheld — anchor status: not_found]
- [abstract] Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis: "Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis"
- [methods] graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [readme] Metaboseek is an interactive tool to analyze and browse your mass spectrometry data.: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
