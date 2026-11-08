---
name: worm-metabolomics-sample-grouping
description: Use when an LC/MS metabolomics dataset (e.g. xcms peak table plus raw
  MS/MS runs from C. elegans WT and hacl-1 mutant or other condition contrasts) must
  be grouped by sample class before differential filtering in Metaboseek.
when_to_use_negative:
- The input is already a fully grouped and annotated feature table — skip to TableAnalysisModule/MultiFilterModule
  analysis instead of re-importing.
- Data are not LC/MS metabolomics features from xcms (e.g. proteomics or RNA-seq tables)
  — Metaboseek grouping assumes MS1 feature tables with m/z and retention-time columns.
- Only a single sample group is present, so no comparative grouping or fold-change
  analysis is possible.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Metaboseek
  role: Host app whose LoadDataModule, TableAnalysisModule, MultiFilterModule and
    CAMERA wrapper import, group and analyze the feature table; installed as an R
    package or Docker image
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Detects and aligns molecular features from LC/MS data across samples; its
    analysis stores the sample grouping reused by Metaboseek
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used together with xcms (mzR-based) to convert xcms results into the Feature
    Table loaded into Metaboseek
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: CAMERA
  role: Groups isotope/adduct features after grouping-based enrichment filtering,
    collapsing features into metabolites
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:e6486ff0fda841e1584eb06108eeabff9e4bc80223260d7e0df205ee21a5f466
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - CAMERA
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# worm-metabolomics-sample-grouping

## Summary

Loads xcms-derived LC/MS feature tables and raw MS data into Metaboseek with the correct WT vs hacl-1 sample-group metadata so that group-wise comparative statistics (e.g. hacl-1 L1-starved vs WT) can be run on consistently grouped samples.

## When to use

Use when an LC/MS metabolomics dataset (e.g. xcms peak table plus raw MS/MS runs from C. elegans WT and hacl-1 mutant or other condition contrasts) must be grouped by sample class before differential filtering in Metaboseek.

## When NOT to use

- The input is already a fully grouped and annotated feature table — skip to TableAnalysisModule/MultiFilterModule analysis instead of re-importing.
- Data are not LC/MS metabolomics features from xcms (e.g. proteomics or RNA-seq tables) — Metaboseek grouping assumes MS1 feature tables with m/z and retention-time columns.
- Only a single sample group is present, so no comparative grouping or fold-change analysis is possible.

## Inputs

- xcms peak/feature-detection results from raw LC/MS (negative-ion) runs of C. elegans samples
- Metaboseek .mskFT negative-ion feature table
- Associated raw MS/MS data files (e.g. mzML/vendor raw runs) for the same samples
- Sample grouping/metadata mapping each MS run to a condition (e.g. WT vs hacl-1 L1-starved)

## Outputs

- Grouped feature table (.mskFT) with per-group intensity columns in Metaboseek
- Group-annotated intensity matrix ready for TableAnalysisModule differential testing
- Group-defined comparison (hacl-1 vs WT) passed to MultiFilterModule and cameraWrapper()

## How to apply

Detect and align molecular features from raw LC/MS data across samples with Metaboseek's xcms GUI module, then load the xcms results into the app as a Feature Table (built via xcms and MSnbase/mzR) so that the negative-ion .mskFT table and associated MS data can be imported through LoadDataModule together with the sample grouping information stored from the xcms analysis. Assign samples to groups (e.g. WT vs hacl-1 L1-starved larvae) at load time, because all downstream modules — TableAnalysisModule two-sample t-tests (unadjusted p-values), MultiFilterModule enrichment filters (≥10-fold change, top 1.25% intensity, p < 0.05), and CAMERA cameraWrapper() isotope/adduct grouping — operate on those group columns. Verify grouping by checking that group labels match the intended contrasts before running any statistical comparison, since mislabeled groups silently corrupt fold-change and p-value columns.

## Related tools

- **Metaboseek** (Host app whose LoadDataModule, TableAnalysisModule, MultiFilterModule and CAMERA wrapper import, group and analyze the feature table; installed as an R package or Docker image) — https://github.com/mjhelf/Metaboseek
- **xcms** (Detects and aligns molecular features from LC/MS data across samples; its analysis stores the sample grouping reused by Metaboseek) — https://github.com/sneumann/xcms
- **MSnbase** (Used together with xcms (mzR-based) to convert xcms results into the Feature Table loaded into Metaboseek) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **CAMERA** (Groups isotope/adduct features after grouping-based enrichment filtering, collapsing features into metabolites) — https://github.com/sneumann/xcms

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); Metaboseek::Metaboseek()  # then use LoadDataModule to import the .mskFT with WT/hacl-1 sample grouping
```

## Evaluation signals

- The loaded .mskFT table exposes the intended group columns (e.g. WT and hacl-1 L1-starved) with every MS run assigned to exactly one group and no ungrouped samples.
- Group intensity columns match the xcms-derived sample ordering, so fold-change direction (hacl-1 vs WT) is verifiable for a known enriched feature.
- After applying group-wise filters, the filtered table contains exactly 57 hacl-1-enriched features under the ≥10-fold, top 1.25% intensity, unadjusted p < 0.05 thresholds.
- Downstream CAMERA grouping on the grouped table yields exactly 14 curated hacl-1-dependent metabolites, each showing the diagnostic glyoxylate fragment at m/z 72.993 in MS2.
- Re-running the grouping plus filtering pipeline reproduces the same feature/metabolite counts (deterministic row-count check).

## Limitations

- Grouping is only meaningful after xcms feature detection/alignment; raw vendor files alone cannot be grouped or compared.
- Molecular features alone do not allow unambiguous compound identification; isotope/adduct (CAMERA) grouping and MS2 fragment evidence (e.g. m/z 72.993) are needed for metabolite-level conclusions.
- Metaboseek's shiny server deployment depends on the mzR package, which can fail to install (e.g. timeout errors on shinyapps.io), so grouping must be done in a locally installed or Docker-based instance.

## Evidence

- [methods] Metaboseek offers a graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data across multiple samples.: "Metaboseek offers a graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data across multiple samples."
- [methods] You can then load xcms results into the app as a "Feature Table" using xcms and MSnbase packages: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [intro] Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation): "Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation)"
- [readme] Metaboseek is built on functions from the xcms package, and provides a web-based graphical user interface built with shiny: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
