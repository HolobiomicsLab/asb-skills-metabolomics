---
name: target-table-schema-standardization
description: Use when the input is a raw target list spreadsheet for targeted LC-MS
  integration and rows need identifiers, m/z, expected RT, and positive/negative ion-mode
  polarity assigned before TARDIS can consume them.
when_to_use_negative:
- Input is already a TARDIS-formatted target data frame or the output of createTargetList()
  — re-parsing would duplicate schema standardization
- The goal is untargeted preprocessing (xcms, MZmine, OpenMS), which does not take
  a target list as input
- Raw vendor/LC-MS data files need peak integration first; this skill only prepares
  the target annotation table
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TARDIS
  role: Provides createTargetList() to parse the user-supplied target table into the
    canonical schema consumed by its peak-integration functions
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Execution environment for loading the .xlsx/.csv target list and running createTargetList();
    TARDIS requires R >= 4.4.0
  repo: https://github.com/UGent-LIMET/TARDIS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:9e98ce9852f298da7b37085d9181ea2523d490d3d1fbc9486abd722920739bac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  repo_url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# target-table-schema-standardization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse a user-supplied target list (.xlsx/.csv) via TARDIS' createTargetList() into a standardized R data frame with the canonical columns target ID, compound name, m/z, expected RT, and ion polarity, ready as input to TARDIS' targeted peak integration.

## When to use

Use when the input is a raw target list spreadsheet for targeted LC-MS integration and rows need identifiers, m/z, expected RT, and positive/negative ion-mode polarity assigned before TARDIS can consume them.

## When NOT to use

- Input is already a TARDIS-formatted target data frame or the output of createTargetList() — re-parsing would duplicate schema standardization
- The goal is untargeted preprocessing (xcms, MZmine, OpenMS), which does not take a target list as input
- Raw vendor/LC-MS data files need peak integration first; this skill only prepares the target annotation table

## Inputs

- User-supplied target list spreadsheet (.xlsx or .csv) with compound identifiers, names, m/z values, expected retention times, and ionization-pattern columns
- targets.xlsx example file from the TARDIS vignette

## Outputs

- Standardized R data frame (from createTargetList()) with columns: target ID, name, m/z, expected RT, ion polarity
- Parsed targets table (e.g. targets_parsed.csv) with non-missing m/z, RT, and polarity for every row

## How to apply

Load the user-supplied targets file (e.g. the targets.xlsx shipped with the TARDIS vignette) into an R data frame with a spreadsheet reader such as readxl. Pass the table through TARDIS' createTargetList() function, which outputs the data frame object used as input to further TARDIS functions. Use the ionization-pattern columns to assign each target its polarity (positive or negative mode). Standardize the output columns to the canonical schema — target ID, name, m/z, expected RT, polarity — and verify completeness before downstream peak integration and quality-metric evaluation (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity). The parsed table feeds TARDIS, an open-source R package for targeted LC-MS metabolomics and lipidomics data.

## Related tools

- **TARDIS** (Provides createTargetList() to parse the user-supplied target table into the canonical schema consumed by its peak-integration functions) — https://github.com/UGent-LIMET/TARDIS
- **R** (Execution environment for loading the .xlsx/.csv target list and running createTargetList(); TARDIS requires R >= 4.4.0) — https://github.com/UGent-LIMET/TARDIS

## Examples

```
library(TARDIS); targets <- createTargetList(readxl::read_excel("targets.xlsx"))
```

## Evaluation signals

- Every target row has non-missing m/z, expected RT, and polarity values after parsing
- Output object is an R data frame matching the canonical schema: target ID, name, m/z, expected RT, polarity
- Each target carries exactly one ion polarity (positive or negative mode) assigned from the ionization-pattern columns
- The parsed data frame is accepted as input by subsequent TARDIS functions without schema errors

## Limitations

- TARDIS requires R version >= 4.4.0 (and BiocManager 3.20 for the latest version); Windows users additionally need Rtools, Mac users need Xcode and GNU Fortran
- The article does not state a version number for the TARDIS package, so parsed-table column expectations may vary across releases
- Package installation may hit connection timeouts because the package contains example data; increase R's timeout option (e.g. 300 s)

## Evidence

- [readme] Make sure `R` (**version >= 4.4.0**) is installed on your computer:: [evidence span withheld — anchor status: external_doc]
