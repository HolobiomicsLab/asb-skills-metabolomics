---
name: comparative-metabolomics-feature-filtering
description: Use when an xcms-derived feature table (csv/.mskFT with 'mz' and 'rt'
  columns and grouped intensity columns) must be reduced to features differentially
  enriched between sample groups, e.g. mutant vs. WT exo-metabolomes.
when_to_use_negative:
- Input is raw LC/MS data (.mzML, .raw) rather than an already feature-detected table
  — run the xcms feature detection module first
- The goal is spectral matching or MS/MS structural annotation (e.g. Find Patterns
  / MS2 Browser tasks), not differential enrichment between sample groups
- Statistical hypothesis testing with multiple-testing correction is required — the
  reported workflow uses unadjusted p < 0.05 thresholds
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Interactive Shiny platform hosting LoadTableModule, MultiFilterModule (Filter
    Table tab), Feature Table Actions comparative statistics, and SaveTableModule
    used to execute the filtering workflow
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Produces the feature table (feature detection and alignment from LC/MS data)
    that serves as the filtering input
  repo: https://github.com/sneumann/xcms
- name: CAMERA
  role: Annotates adduct and isotope feature groups used for post-filter deduplication
- name: MSnbase
  role: Supports loading xcms results into the app as a Feature Table
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:0fd95b68f965f0e762da86ab5a719bed312b471c6ff076d81b27f32f4de716c1
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# comparative-metabolomics-feature-filtering

## Summary

Filter untargeted LC/MS feature tables in Metaboseek by combining fold-change, intensity, and sample-group-dependent significance filters (plus adduct/isotope deduplication) to isolate metabolites enriched in one biological group, e.g. the 32 CA-dependent metabolites enriched in hacl-1 mutants or the >1000 hacl-1-enriched adult exo-metabolome features.

## When to use

Use when an xcms-derived feature table (csv/.mskFT with 'mz' and 'rt' columns and grouped intensity columns) must be reduced to features differentially enriched between sample groups, e.g. mutant vs. WT exo-metabolomes.

## When NOT to use

- Input is raw LC/MS data (.mzML, .raw) rather than an already feature-detected table — run the xcms feature detection module first
- The goal is spectral matching or MS/MS structural annotation (e.g. Find Patterns / MS2 Browser tasks), not differential enrichment between sample groups
- Statistical hypothesis testing with multiple-testing correction is required — the reported workflow uses unadjusted p < 0.05 thresholds

## Inputs

- xcms-derived feature table (.csv or .mskFT) with 'mz' (m/z values) and 'rt' (retention time in seconds) columns plus group-defined intensity columns
- Sample group definitions (e.g. WT vs. hacl-1(tm6725), CA-supplemented vs. unsupplemented)
- CAMERA-annotated feature group assignments for adduct/isotope removal

## Outputs

- Filtered feature table containing only group-enriched features with fold-change, intensity, and significance columns (e.g. hacl1_op50_enriched_features.csv)
- .mskFT file preserving the complete filtering and processing history
- Adduct/isotope-deduplicated list of compounds (e.g. 32 CA-derived metabolites)

## How to apply

Load the feature table into Metaboseek via LoadTableModule (files with 'mz' and 'rt' columns are supported as .csv, with .mskFT preferred for embedded metadata), then compute group-wise mean intensities, fold changes (e.g. hacl-1/WT), and unadjusted p-values in Feature Table Actions with controlGroup = WT. Apply stacked filters in the MultiFilterModule: an intensity filter keeping only the top ~12% most intense features to remove low-abundance noise, a significance filter at unadjusted p < 0.05, and a fold-change filter (e.g. ≥5-fold enriched in the mutant group). After filtering, remove CAMERA-annotated adduct and isotope duplicates so each compound is represented once (e.g. reducing the hacl-1 CA-supplemented hits to 32 CA-derived metabolites). Save with SaveTableModule, ideally as .mskFT which preserves the complete processing history, and cross-check the retained feature count against expected values (>1000 for adult hacl-1 exo-metabolome; 32 for CA-derived metabolites).

## Related tools

- **Metaboseek** (Interactive Shiny platform hosting LoadTableModule, MultiFilterModule (Filter Table tab), Feature Table Actions comparative statistics, and SaveTableModule used to execute the filtering workflow) — https://github.com/mjhelf/Metaboseek
- **xcms** (Produces the feature table (feature detection and alignment from LC/MS data) that serves as the filtering input) — https://github.com/sneumann/xcms
- **CAMERA** (Annotates adduct and isotope feature groups used for post-filter deduplication)
- **MSnbase** (Supports loading xcms results into the app as a Feature Table)

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- Filtered table retains the expected feature counts: >1000 features for adult hacl-1 vs. WT OP50 exo-metabolome; exactly 32 CA-derived metabolites for the CA-feeding comparison (Supplementary Fig. 3)
- Exported CSV parses as comma-separated with fold-change, intensity, and significance fields present
- All retained features satisfy the applied thresholds: unadjusted p < 0.05, ≥5-fold enrichment, intensity in top 12% of the table
- No CAMERA-annotated adduct/isotope duplicates remain — each CA-derived compound is represented once
- .mskFT output preserves the complete processing history of applied filters

## Limitations

- Filters rely on unadjusted p-values (p < 0.05), so the feature set is enriched but not corrected for multiple testing
- Fold-change and intensity cutoffs (≥5-fold, top 12%) are sample- and dataset-dependent and may need retuning for other comparisons
- Adduct/isotope deduplication depends on CAMERA annotation quality; mis-annotated groups may cause over- or under-merging of features

## Evidence

- [methods] graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [abstract] Abstract hacl-1 metabolites: "which uncovers more than 200 previously uncharacterized α-oxidation-dependent metabolites in an untargeted comparison of wildtype and α-oxidation-defective hacl-1 mutants"
- [readme] README xcms shiny: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
