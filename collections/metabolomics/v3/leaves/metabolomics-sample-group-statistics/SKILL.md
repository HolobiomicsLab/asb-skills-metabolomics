---
name: metabolomics-sample-group-statistics
description: Use when an xcms or .mskFT feature table with mz, rt and group-defined
  intensity columns must be statistically compared across biological groups to find
  enriched features.
when_to_use_negative:
- Data are raw centroided MS runs not yet processed by xcms feature detection and
  alignment — run feature detection first
- The question requires validated compound identification rather than unadjusted group
  statistics — this skill yields Level 3 tentative annotations at best
- Multiple-testing-corrected significance (e.g. FDR-adjusted p-values) is required,
  since this workflow explicitly uses no p-value adjustment
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Metaboseek
  role: GUI platform providing Feature Table Actions for group-wise statistics (mean
    intensities, fold change, p-values), Regroup Table for sample labeling, Grouped
    EICs for validation, and Save Table for CSV export
  repo: github.com/mjhelf/Metaboseek
- name: xcms
  role: Detects and aligns molecular features from LC/MS data to build the feature
    table that the group statistics are computed on
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used to load xcms results into Metaboseek as a Feature Table
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:fcf67d6623329c83c507499cbd01e5f2d04ecd0d607cf421ec54925ee335a363
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - Metaboseek
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# metabolomics-sample-group-statistics

## Summary

Applies group-wise comparative statistics (mean intensities, fold changes, p-values) to an xcms-derived LC/MS feature table in Metaboseek to identify metabolites enriched between sample groups, e.g. hacl-1 vs WT C. elegans exo-metabolomes.

## When to use

Use when an xcms or .mskFT feature table with mz, rt and group-defined intensity columns must be statistically compared across biological groups to find enriched features.

## When NOT to use

- Data are raw centroided MS runs not yet processed by xcms feature detection and alignment — run feature detection first
- The question requires validated compound identification rather than unadjusted group statistics — this skill yields Level 3 tentative annotations at best
- Multiple-testing-corrected significance (e.g. FDR-adjusted p-values) is required, since this workflow explicitly uses no p-value adjustment

## Inputs

- xcms-derived feature table (.mskFT or .csv) with mz, rt, and group-defined sample intensity columns
- centroided LC/MS data files (mzML/mzXML) with an xcms project folder carrying sample grouping
- Sample metadata assigning biological group labels (e.g. WT, hacl-1, diet strain) to replicate injections

## Outputs

- Filtered feature table (.csv) with per-feature fold-change, group mean intensity, and significance columns
- List of features passing intensity, p-value, and fold-change filters (e.g. >1000 hacl-1-enriched features)
- Grouped EIC plots confirming group-dependent features at specific m/z values (e.g. m/z 496.2317, m/z 570.2674)

## How to apply

Load the xcms-derived feature table (ensuring `mz` and `rt` columns plus group-defined intensity columns are present) into Metaboseek and assign group labels by genotype and/or diet in the Regroup Table tab. In Feature Table Actions, compute group-wise mean intensities, fold changes (e.g. hacl-1/WT), and unadjusted p-values across groups using a comparative statistics analysis with the control group defined (e.g. controlGroup = WT, no p-value adjustment). Apply intensity filtering (e.g. retain only features in the top 12% of remaining feature intensities to remove low-abundance noise), significance filtering (unadjusted p < 0.05), and a fold-change cutoff (e.g. at least 5-fold enrichment). Export the filtered table with fold-change, intensity and significance columns via Save Table in .csv format; verify group-difference claims by EIC inspection (RT window, ppm tolerance) before exporting.

## Related tools

- **Metaboseek** (GUI platform providing Feature Table Actions for group-wise statistics (mean intensities, fold change, p-values), Regroup Table for sample labeling, Grouped EICs for validation, and Save Table for CSV export) — github.com/mjhelf/Metaboseek
- **xcms** (Detects and aligns molecular features from LC/MS data to build the feature table that the group statistics are computed on) — https://github.com/sneumann/xcms
- **MSnbase** (Used to load xcms results into Metaboseek as a Feature Table) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- Exported CSV parses as a comma-separated table containing fold-change, intensity, and significance columns for every retained row
- Number of retained features exceeds 1000 after applying the top 12% intensity, unadjusted p < 0.05, and 5-fold-change filters (task_004 benchmark)
- Diet-dependent features such as m/z 496.2317 (cyclopropane GPE 13:1) and m/z 570.2674 (cyclopropane GLEA) show near-zero or absent mean intensities in the JW1653-1-fed hacl-1 group
- Regrouped table correctly assigns every replicate to a genotype/diet group before statistics are computed
- Grouped EICs confirm each reported group-dependent feature within the RT window and ppm tolerance used for inspection

## Limitations

- Comparative statistics use unadjusted p-values; multiple-testing burden on thousands of features is not controlled
- Metabolite annotations from this workflow are limited to confidence Level 3 (tentative structure or compound class assignments only)
- Fold-change and intensity thresholds are dataset-specific (top 12% intensity, 5-fold) and must be re-justified for other experiments

## Evidence

- [abstract] Modules and statistics: "Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis"
- [intro] Comparative workflow: "Processing of metabolomics data involves three major steps – feature detection, comparative statistical analysis, and structural characterization"
- [methods] Feature detection: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [readme] README tool basis: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
- [results] Cyclopropane finding: [evidence span withheld — anchor status: not_found]
