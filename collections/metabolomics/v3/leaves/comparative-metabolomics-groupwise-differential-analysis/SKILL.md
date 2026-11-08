---
name: comparative-metabolomics-groupwise-differential-analysis
description: Use when comparing untargeted LC/MS feature intensities between two sample
  groups (e.g., hacl-1 mutant vs wild-type) to identify enriched molecular features
  and collapse them into metabolites.
when_to_use_negative:
- The input is already a curated metabolite-level table rather than a raw xcms/MSnbase
  feature table requiring statistical comparison
- The goal is molecular formula filtering (Seven Golden Rules) or spectral similarity
  networking rather than group-wise differential enrichment testing
- The analysis requires multiple-testing-corrected p-values rather than the unadjusted
  p < 0.05 screening used in this discovery workflow
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Metaboseek
  role: Provides the GUI modules (LoadDataModule, TableAnalysisModule, MultiFilterModule,
    MS2 Browser/Specmodule, SaveTableModule) and the cameraWrapper() function used
    to load the feature table, run t-tests, filter enriched features, group isotopes/adducts,
    and export the curated metabolite table
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Underlying feature detection and alignment engine whose results are loaded
    into Metaboseek as a Feature Table and which supplies CAMERA grouping
  repo: https://github.com/sneumann/xcms
- name: CAMERA
  role: Performs isotope/adduct grouping (polarity, maxcharge, maxiso, ppm, mzabs,
    minfrac) to collapse co-eluting features into pseudospectra prior to manual metabolite
    curation
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used with xcms to load xcms results into the app as a Feature Table and to
    access MS2 spectra for fragment verification
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:f8e54ac490212506754cd03aa1b417d03ae0764c93d3a640d394fafb7b05a4e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - CAMERA
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# comparative-metabolomics-groupwise-differential-analysis

## Summary

Group-wise differential analysis of LC/MS molecular features in Metaboseek: load an xcms/MSnbase feature table (.mskFT) with sample grouping, run two-sample t-tests between groups, and apply stringent enrichment filters (≥10-fold change, top 1.25% intensity, unadjusted p < 0.05), then collapse isotope/adduct features via CAMERA into metabolites.

## When to use

Use when comparing untargeted LC/MS feature intensities between two sample groups (e.g., hacl-1 mutant vs wild-type) to identify enriched molecular features and collapse them into metabolites.

## When NOT to use

- The input is already a curated metabolite-level table rather than a raw xcms/MSnbase feature table requiring statistical comparison
- The goal is molecular formula filtering (Seven Golden Rules) or spectral similarity networking rather than group-wise differential enrichment testing
- The analysis requires multiple-testing-corrected p-values rather than the unadjusted p < 0.05 screening used in this discovery workflow

## Inputs

- Negative-ion LC/MS feature table (.mskFT) exported from an xcms analysis with sample grouping information
- Associated MS and MS2 raw data for wild-type and hacl-1 L1-starved samples
- Sample grouping/metadata columns defining WT and hacl-1 groups

## Outputs

- Filtered table of ≥10-fold-enriched, top 1.25% intensity, unadjusted p < 0.05 molecular features (57 hacl-1-enriched features)
- Curated metabolite-level CSV (e.g., fig3b_hacl1_metabolites.csv) with mz, rt, CAMERA isotope/adduct group, and p-value columns (14 metabolites)
- MS2 evidence per metabolite confirming the diagnostic glyoxylate fragment at m/z 72.993

## How to apply

Load the negative-ion feature table (.mskFT) and associated MS data with Metaboseek's LoadDataModule, retaining the sample grouping from the xcms analysis. Run group-wise comparative statistics in the TableAnalysisModule using two-sample t-tests on unadjusted p-values comparing the mutant (hacl-1) intensity columns against wild-type. Apply the MultiFilterModule to keep only features enriched ≥10-fold in the mutant, ranked in the top 1.25% by intensity, with unadjusted p < 0.05 (this yielded 57 hacl-1-enriched features). Then call Metaboseek's exported cameraWrapper() (xcms/CAMERA) with isotope/adduct grouping parameters (polarity, maxcharge, maxiso, ppm, mzabs, minfrac) to assign features to pseudospectra, and manually curate groups so co-grouped isotope/adduct features collapse into metabolites (14 hacl-1-dependent metabolites). Verify each metabolite by checking for the diagnostic glyoxylate product ion at m/z 72.993 in its MS2 spectra via the MS2 Browser / Specmodule before exporting the curated table with SaveTableModule.

## Related tools

- **Metaboseek** (Provides the GUI modules (LoadDataModule, TableAnalysisModule, MultiFilterModule, MS2 Browser/Specmodule, SaveTableModule) and the cameraWrapper() function used to load the feature table, run t-tests, filter enriched features, group isotopes/adducts, and export the curated metabolite table) — https://github.com/mjhelf/Metaboseek
- **xcms** (Underlying feature detection and alignment engine whose results are loaded into Metaboseek as a Feature Table and which supplies CAMERA grouping) — https://github.com/sneumann/xcms
- **CAMERA** (Performs isotope/adduct grouping (polarity, maxcharge, maxiso, ppm, mzabs, minfrac) to collapse co-eluting features into pseudospectra prior to manual metabolite curation) — https://github.com/sneumann/xcms
- **MSnbase** (Used with xcms to load xcms results into the app as a Feature Table and to access MS2 spectra for fragment verification) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- The filtered feature table contains exactly 57 molecular features after applying the 10-fold, top 1.25% intensity, and unadjusted p < 0.05 filters
- The curated output contains exactly 14 hacl-1-dependent metabolites after CAMERA isotope/adduct assignment and manual curation
- Each curated metabolite row retains mz, rt, isotope/adduct group, and p-value fields (field-presence check)
- Every curated metabolite's MS2 spectrum contains the diagnostic glyoxylate product ion at m/z 72.993
- Unadjusted (not FDR-corrected) p-values are used, consistent with the discovery-oriented screening design

## Limitations

- Metabolite annotations are limited to confidence Level 3 (tentative structure or compound class assignments only)
- Unadjusted p-values and very stringent fold/intensity filters are discovery-oriented heuristics; features outside the top 1.25% intensity or <10-fold enriched are excluded even if statistically significant
- Manual curation of CAMERA pseudospectra is required to collapse isotope/adduct features into metabolites, introducing an analyst-dependent step

## Evidence

- [intro] Processing of metabolomics data involves three major steps – feature detection, comparative statistical analysis, and structural characterization: "Processing of metabolomics data involves three major steps – feature detection, comparative statistical analysis, and structural characterization"
- [abstract] uncovers more than 200 previously uncharacterized metabolites in hacl-1 comparison: "which uncovers more than 200 previously uncharacterized α-oxidation-dependent metabolites in an untargeted comparison of wildtype and α-oxidation-defective hacl-1 mutants"
- [methods] Metaboseek xcms feature detection GUI: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [readme] README: load xcms results as Feature Table: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html)"
