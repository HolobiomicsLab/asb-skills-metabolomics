---
name: metabolomics-table-export
description: Use when a Metaboseek comparative analysis (e.g. hacl-1 vs WT exo-metabolome)
  has produced a filtered feature table with fold-change, intensity, and significance
  columns that must be persisted as a .csv deliverable.
when_to_use_negative:
- The input is raw LC/MS data (mzML/NetCDF) that still requires xcms feature detection
  and alignment — export precedes from an existing feature table, not raw spectra
- The table has not yet been filtered — exporting the unfiltered table would not reproduce
  the >1000 enriched-feature result and conflates export with differential analysis
- The analysis requires p-value adjustment or molecular formula annotation steps (SIRIUS,
  Seven Golden Rules filtering) that are separate Metaboseek modules
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Metaboseek
  role: Provides the Feature Table Actions and Save Table functions used to compute
    comparative statistics, apply filters, and export the filtered feature table as
    .csv
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Detects and aligns the molecular features from LC/MS data that constitute
    the feature table loaded into Metaboseek for export
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used to load xcms results into Metaboseek as a Feature Table prior to filtering
    and saving
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:46a6b8f0e88dfe2fb19eab60d8d66d67dc8e6b969e090d6a076a65837d0fa931
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metabolomics-table-export

## Summary

Export a filtered xcms/Metaboseek feature table (with m/z, retention time, intensity, fold-change, and significance columns) to a .csv file using Metaboseek's Save Table function, so downstream differential-abundance results from untargeted comparative metabolomics can be validated and reused.

## When to use

Use when a Metaboseek comparative analysis (e.g. hacl-1 vs WT exo-metabolome) has produced a filtered feature table with fold-change, intensity, and significance columns that must be persisted as a .csv deliverable.

## When NOT to use

- The input is raw LC/MS data (mzML/NetCDF) that still requires xcms feature detection and alignment — export precedes from an existing feature table, not raw spectra
- The table has not yet been filtered — exporting the unfiltered table would not reproduce the >1000 enriched-feature result and conflates export with differential analysis
- The analysis requires p-value adjustment or molecular formula annotation steps (SIRIUS, Seven Golden Rules filtering) that are separate Metaboseek modules

## Inputs

- xcms-derived Metaboseek feature table with `mz` and `rt` columns and group-defined intensity columns
- feature table with computed group-wise mean intensities, fold changes (hacl-1/WT), and unadjusted p-values
- filtered feature subset (top 12% intensity, unadjusted p < 0.05, >=5-fold enriched in hacl-1)

## Outputs

- comma-separated .csv feature table (e.g. hacl1_op50_enriched_features.csv) containing fold-change, intensity, and significance columns plus m/z and retention time identifiers

## How to apply

Load the xcms-derived feature table into Metaboseek, ensuring the required `mz` (m/z values) and `rt` (retention time in seconds) columns plus group-defined intensity columns are present before any export. After running the FTAnalysisParam-style comparative statistics (controlGroup = WT, unadjusted p-values) and applying the study's filters (top 12% intensity, unadjusted p < 0.05, >= 5-fold enrichment in hacl-1 relative to WT), use Metaboseek's Save Table function to write the retained features to .csv format. The exported file should preserve the computed fold-change (hacl-1/WT), mean intensity, and significance columns so that row counts and threshold compliance can be re-verified independently of the GUI. Rationale: persisting the filtered table makes the >1000-feature hacl-1 enrichment result auditable against the source feature table.

## Related tools

- **Metaboseek** (Provides the Feature Table Actions and Save Table functions used to compute comparative statistics, apply filters, and export the filtered feature table as .csv) — https://github.com/mjhelf/Metaboseek
- **xcms** (Detects and aligns the molecular features from LC/MS data that constitute the feature table loaded into Metaboseek for export) — https://github.com/sneumann/xcms
- **MSnbase** (Used to load xcms results into Metaboseek as a Feature Table prior to filtering and saving)

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); # then in the Metaboseek shiny app: load feature table (mz, rt, intensity columns) -> Feature Table Actions -> apply filters (top 12% intensity, p < 0.05, FC >= 5) -> Save Table as hacl1_op50_enriched_features.csv
```

## Evaluation signals

- The exported hacl1_op50_enriched_features.csv exists and parses as a comma-separated table
- Fold-change, intensity, and significance fields are present as columns in the exported file
- The number of retained feature rows exceeds 1000, consistent with the reported >1000 features enriched in the hacl-1 adult exo-metabolome
- Every exported row satisfies all three filters: top 12% intensity, unadjusted p < 0.05, and >=5-fold hacl-1/WT enrichment
- The exported table retains `mz` and `rt` identifier columns matching the source xcms feature table row-for-row

## Limitations

- Export is only as sound as the upstream comparative statistics; the workflow deliberately uses unadjusted p-values (p < 0.05), so exported significance reflects no multiple-testing correction
- Metabolite annotations in this workflow are limited to confidence Level 3 (tentative structure or compound class assignments), so the exported table identifies features by m/z and retention time rather than confirmed structures
- Metaboseek is under active development with new features constantly added, so GUI module behavior may change between versions

## Evidence

- [other] Load the xcms-derived feature table ... ensuring the required `mz` (m/z values) and `rt` (retention time in seconds) columns plus group-defined intensity columns are present: [evidence span withheld — anchor status: not_found]
- [other] Apply the fold-change filter, retaining features at least 5-fold enriched in hacl-1 relative to WT: [evidence span withheld — anchor status: not_found]
- [other] Use Metaboseek's Save Table function to export the filtered feature table with fold-change, intensity, and significance columns in .csv format: [evidence span withheld — anchor status: not_found]
- [readme] Metaboseek is an interactive tool to analyze and browse your mass spectrometry data, built on functions from the xcms package: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
- [methods] You can then load xcms results into the app as a "Feature Table" (using `xcms` and `MSnbase` packages): "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [abstract] Metaboseek uncovered more than 200 previously uncharacterized alpha-oxidation-dependent metabolites in a comparison of wildtype and hacl-1 mutants: "which uncovers more than 200 previously uncharacterized α-oxidation-dependent metabolites in an untargeted comparison of wildtype and α-oxidation-defective hacl-1 mutants"
