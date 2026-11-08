---
name: lcms-feature-table-filtering
description: Use when a feature table with mz, rt, and group-defined intensity columns
  is available and features enriched between two conditions (e.g. hacl-1 vs WT) must
  be isolated with intensity, significance, and fold-change cutoffs.
when_to_use_negative:
- Input is raw vendor LC/MS data that has not yet undergone xcms feature detection
  and alignment — use the xcms module first
- The analysis goal is molecular formula or fragment-annotation work (e.g. SIRIUS
  prediction, Find Patterns GLEA motif search) rather than abundance-based feature
  enrichment
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Metaboseek
  role: Provides the shiny GUI feature table loading, FTAnalysis comparative statistics,
    MultiFilterModule filtering, cameraWrapper isotope/adduct grouping, and Save Table
    export
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Detects and aligns the molecular features that populate the feature table
    prior to filtering
  repo: https://github.com/sneumann/xcms
- name: CAMERA
  role: Assigns features into isotope/adduct pseudospectra so co-grouped features
    collapse into metabolites after filtering
  repo: https://github.com/mjhelf/Metaboseek
- name: MSnbase
  role: Supports loading xcms results into Metaboseek as a Feature Table and MS2 spectrum
    access for validation
  repo: https://github.com/mjhelf/Metaboseek
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:b9f77d883740707cf8b45d94e7c9fa51722441382da9ea11011af242231b087d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - CAMERA
  - FindPatternsModule
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# lcms-feature-table-filtering

## Summary

Filter xcms/Metaboseek-derived LC/MS molecular feature tables by intensity percentile, unadjusted p-value, and genotype fold-change thresholds to isolate metabolites enriched in a mutant condition, as done in the hacl-1 vs WT C. elegans exo-metabolome comparison.

## When to use

Use when a feature table with mz, rt, and group-defined intensity columns is available and features enriched between two conditions (e.g. hacl-1 vs WT) must be isolated with intensity, significance, and fold-change cutoffs.

## When NOT to use

- Input is raw vendor LC/MS data that has not yet undergone xcms feature detection and alignment — use the xcms module first
- The analysis goal is molecular formula or fragment-annotation work (e.g. SIRIUS prediction, Find Patterns GLEA motif search) rather than abundance-based feature enrichment

## Inputs

- xcms-derived feature table (.mskFT or .csv with 'mz' and 'rt' columns and group-defined intensity columns)
- LC/MS data files (.mzML or other supported centroided formats) for MS2 validation
- Sample grouping information from the xcms analysis (e.g. hacl-1 vs WT)

## Outputs

- Filtered feature table (.csv) with fold-change, intensity, and unadjusted p-value columns (e.g. hacl1_op50_enriched_features.csv)
- Metabolite-level curated table with CAMERA isotope/adduct annotations (e.g. fig3b_hacl1_metabolites.csv with mz, rt, isotope/adduct group, and p-value columns)

## How to apply

Load the xcms-derived feature table into Metaboseek's Feature Table module, ensuring required 'mz' (m/z) and 'rt' (retention time in seconds) columns plus group-defined intensity columns are present. Run group-wise comparative statistics (FTAnalysisParam/TableAnalysisModule-style two-sample t-tests with controlGroup = WT and no p-value adjustment) to compute fold changes (mutant/WT) and unadjusted p-values. Apply MultiFilterModule cutoffs successively: retain features in the top intensity percentile of the table (e.g. top 12% for adult OP50 exo-metabolome, top 1.25% for L1-starved larvae), unadjusted p < 0.05, and a fold-change threshold (≥5-fold for adults, ≥10-fold for L1). Collapse co-grouped isotope/adduct features via Metaboseek's cameraWrapper() (xcms/CAMERA) before or after filtering when metabolite-level counts are needed. Export the filtered table with SaveTableModule as .csv, retaining fold-change, intensity, and significance columns.

## Related tools

- **Metaboseek** (Provides the shiny GUI feature table loading, FTAnalysis comparative statistics, MultiFilterModule filtering, cameraWrapper isotope/adduct grouping, and Save Table export) — https://github.com/mjhelf/Metaboseek
- **xcms** (Detects and aligns the molecular features that populate the feature table prior to filtering) — https://github.com/sneumann/xcms
- **CAMERA** (Assigns features into isotope/adduct pseudospectra so co-grouped features collapse into metabolites after filtering) — https://github.com/mjhelf/Metaboseek
- **MSnbase** (Supports loading xcms results into Metaboseek as a Feature Table and MS2 spectrum access for validation) — https://github.com/mjhelf/Metaboseek

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- Exported filtered table parses as a comma-separated file and contains fold-change, intensity, and significance fields
- Row counts match published expectations exactly where stated (e.g. 57 hacl-1-enriched L1 features; 14 curated hacl-1-dependent metabolites; >1000 adult exo-metabolome features)
- All retained features satisfy all three filters simultaneously (top intensity percentile, unadjusted p < 0.05, ≥5- or ≥10-fold enrichment)
- Each curated metabolite carries a CAMERA isotope/adduct annotation and diagnostic MS2 evidence (e.g. glyoxylate product ion at m/z 72.993)
- Required 'mz' and 'rt' columns are present and retention times are in seconds

## Limitations

- p-values are applied unadjusted (no multiple-testing correction), so filtered hit counts depend on chosen intensity and fold-change cutoffs
- Many detected features remain unidentified; annotations are limited to confidence Level 3 (tentative structure or compound class assignments only)
- Exact expected feature counts (57, 14, >1000) are tied to the hacl-1(tm6725) vs WT datasets and may not transfer to other comparisons

## Evidence

- [abstract] Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis: "Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis"
- [methods] Noise level peak filter: "peaks at an intensity below a set percentage of the maximum peak 
intensity in a scan (`Noise level in %`) will be excluded"
- [methods] Load xcms results as feature table: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [readme] README Metaboseek description: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
