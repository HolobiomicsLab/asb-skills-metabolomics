---
name: lc-ms-feature-table-formatting-and-filtering
description: Use when peak-picked LC-HRMS feature tables from two untargeted metabolomics
  data sets must be cleaned and formatted before feature pair alignment with metabCombiner.
when_to_use_negative:
- The two data sets were acquired with excessively disparate chromatographic techniques
  (e.g. an RPLC dataset and a HILIC dataset), since elution order of metabolites must
  be mostly preserved
- Abundance values in the input tables are normalized or transformed in a way that
  removes ranked abundance order information
- The input is raw MS data (mzML/NetCDF) rather than peak-picked, conventionally aligned
  feature tables
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metabCombiner
  role: R package implementing the data formatting and filtering step (retention time
    range, missingness, and duplicate filters) that precedes feature m/z grouping
    and pairwise alignment detection
  repo: https://github.com/hhabra/metabCombiner
- name: metabCombiner R package
  role: Installable package (also on Bioconductor) providing the filtering workflow;
    install via devtools::install_github
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Runtime environment for the metabCombiner package (R >= 3.5; R 4.0+ for the
    main branch, 'R3.5' branch for older versions)
  repo: https://github.com/hhabra/metabCombiner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:44eb3f064982fb5f16604ef7dbbc0421659bcaa836349149299251d5d3442e73
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hhabra/metabCombiner
  repo_url: https://github.com/hhabra/metabCombiner
  tools:
  - GAM
  - R
  - fit_gam
  - fit_loess
  - metabCombiner
  - metabCombiner R package
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# lc-ms-feature-table-formatting-and-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Format peak-picked untargeted LC-MS feature tables for metabCombiner and apply its m/z, retention-time (rtmin/rtmax), missingness (misspc), and duplicate filters so that two disparately acquired data sets are comparable before alignment.

## When to use

Use when peak-picked LC-HRMS feature tables from two untargeted metabolomics data sets must be cleaned and formatted before feature pair alignment with metabCombiner.

## When NOT to use

- The two data sets were acquired with excessively disparate chromatographic techniques (e.g. an RPLC dataset and a HILIC dataset), since elution order of metabolites must be mostly preserved
- Abundance values in the input tables are normalized or transformed in a way that removes ranked abundance order information
- The input is raw MS data (mzML/NetCDF) rather than peak-picked, conventionally aligned feature tables

## Inputs

- Peak-picked, conventionally aligned untargeted LC-MS feature tables from two data sets acquired in the same ionization mode (e.g. B3N3 and IC43 urine tables from Metabolomics Workbench Study ST001122)
- Filter parameter settings: rtmin/rtmax, misspc, and duplicate detection tolerances (e.g. 0.0025 Da, 0.05 min)

## Outputs

- Filtered, metabCombiner-formatted feature tables ready for feature m/z grouping and pairwise alignment detection
- An initial feature pair alignment (FPA) table (e.g. 95898 rows for the ST001122 urine analysis)

## How to apply

Take peak-picked feature tables (e.g. the B3N3 and IC43 urine tables from Metabolomics Workbench Study ST001122) and format them for metabCombiner, ensuring samples are biologically similar, in the same ionization mode, and that abundances retain ranked abundance order (no normalizations that destroy it). Apply the retention time range filter restricting features to those between the rtmin and rtmax arguments, the missingness filter eliminating features below a threshold percentage given by misspc, and the duplicate filter detecting and removing features within close m/z and RT distances (e.g. 0.0025 Da and 0.05 min). Filtering must precede the feature m/z grouping stage (e.g. binGap = 0.01), which generates the initial search space of feature pair alignments; for the ST001122 urine task this produced an initial table of 95898 FPAs. Because preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate downstream, the preprocessing method and choice of parameters should be considered carefully for each data set.

## Related tools

- **metabCombiner** (R package implementing the data formatting and filtering step (retention time range, missingness, and duplicate filters) that precedes feature m/z grouping and pairwise alignment detection) — https://github.com/hhabra/metabCombiner
- **metabCombiner R package** (Installable package (also on Bioconductor) providing the filtering workflow; install via devtools::install_github) — https://github.com/hhabra/metabCombiner
- **R** (Runtime environment for the metabCombiner package (R >= 3.5; R 4.0+ for the main branch, 'R3.5' branch for older versions)) — https://github.com/hhabra/metabCombiner

## Examples

```
devtools::install_github("hhabra/metabCombiner", build_vignettes = TRUE); library(metabCombiner)
```

## Evaluation signals

- Verify each filtered table retains only features within the specified rtmin–rtmax window and above the misspc missingness threshold
- Confirm duplicate features within the close m/z and RT distances (e.g. 0.0025 Da & 0.05 min) were removed from both tables
- Check abundance columns were not normalized in a way that removed ranked abundance order information
- For the ST001122 urine reproduction, the initial unsupervised grouping stage (binGap = 0.01) should yield an FPA table of 95898 rows
- Both data sets must be in the same ionization mode (positive or negative) before proceeding

## Limitations

- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate into the combined analysis; the preprocessing method and choice of parameters should be considered carefully for each data set
- The paper does not state the version of the metabCombiner R package used
- The paper does not name the specific preprocessing software tools used (referenced only as 'several different software tools')

## Evidence

- [readme] README installation instructions: [evidence span withheld — anchor status: external_doc]
