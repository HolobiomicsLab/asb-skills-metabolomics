---
name: mz-retention-time-annotation-handling
description: Use when a targeted LC–MS metabolomics or lipidomics workflow starts
  from a user-authored target file (.xlsx/.csv) that must be parsed into a data frame
  with identifier, name, m/z, expected RT and ion polarity before peak integration.
when_to_use_negative:
- Input is already a TARDIS-formatted target data frame that only needs to be passed
  to the peak-integration step
- Input is raw LC–MS data rather than a target table — the subsequent TARDIS targeted
  peak integration, not this parsing skill, is what operates on raw chromatograms
- The analysis is untargeted feature discovery (e.g. xcms, MZmine, OpenMS preprocessing)
  rather than targeted peak integration of a known target list
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TARDIS
  role: Provides the createTargetList() parsing mechanism that converts the user's
    .xlsx/.csv target table into a data frame input for downstream targeted LC–MS
    peak integration
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (>= 4.4.0, installed with BiocManager 3.20) in which the
    target table is loaded, parsed, annotated with polarity, and exported
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:4cd817f191acb0c145c3566ca4e8a1d1d88e19f044e549a65311855b82f73453
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mz-retention-time-annotation-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses a user-supplied target table (.xlsx or .csv) into a structured R data frame via TARDIS' createTargetList() mechanism, carrying identifier, name, m/z, expected RT and ion polarity per target so it can feed TARDIS' targeted peak integration for LC–MS metabolomics/lipidomics data.

## When to use

Use when a targeted LC–MS metabolomics or lipidomics workflow starts from a user-authored target file (.xlsx/.csv) that must be parsed into a data frame with identifier, name, m/z, expected RT and ion polarity before peak integration.

## When NOT to use

- Input is already a TARDIS-formatted target data frame that only needs to be passed to the peak-integration step
- Input is raw LC–MS data rather than a target table — the subsequent TARDIS targeted peak integration, not this parsing skill, is what operates on raw chromatograms
- The analysis is untargeted feature discovery (e.g. xcms, MZmine, OpenMS preprocessing) rather than targeted peak integration of a known target list

## Inputs

- User-supplied target list file in .xlsx or .csv format (e.g. targets.xlsx from the TARDIS vignette)
- R session (version >= 4.4.0) with the TARDIS package installed via BiocManager
- Spreadsheet reader (e.g. readxl) to load the raw table into an R data frame

## Outputs

- R data frame of parsed targets with canonical columns: target ID, name, m/z, expected RT, ion polarity
- targets_parsed.csv export of the standardized target table

## How to apply

Load the target spreadsheet (e.g. the targets.xlsx shipped with the TARDIS vignette) into R using a spreadsheet reader, ensuring R >= 4.4.0 and TARDIS installed via BiocManager as the README specifies. Invoke TARDIS' createTargetList() parsing mechanism on the loaded table, which outputs a data frame consumed by further TARDIS functions. Use the ionization-pattern columns to assign each target its polarity (positive/negative mode) as a decision point for ion-mode-specific integration. Standardize the output columns to the canonical schema: target ID, name, m/z, expected RT, and polarity. Validate that every target row has non-missing m/z, expected RT and polarity before handing the data frame to TARDIS' targeted peak integration, whose results object returns a data.frame of per-target AUC values per run.

## Related tools

- **TARDIS** (Provides the createTargetList() parsing mechanism that converts the user's .xlsx/.csv target table into a data frame input for downstream targeted LC–MS peak integration) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (>= 4.4.0, installed with BiocManager 3.20) in which the target table is loaded, parsed, annotated with polarity, and exported)

## Examples

```
library(TARDIS); targets <- createTargetList(readxl::read_xlsx("targets.xlsx"))
```

## Evaluation signals

- Every row of the parsed data frame has non-missing m/z, expected RT and polarity values
- Output columns match the canonical schema exactly: target ID, name, m/z, expected RT, polarity
- Row count of the parsed data frame equals the number of target rows in the source .xlsx/.csv file
- The object is an R data frame that TARDIS' downstream functions accept as input without re-shaping
- Each target is assigned exactly one polarity (positive or negative mode) consistent with its ionization-pattern column

## Limitations

- No version number for the TARDIS R package is stated in the article; reproducibility depends on pinning the GitHub install
- The README requires R >= 4.4.0 (plus Rtools on Windows or Xcode/GNU Fortran on Mac) and BiocManager 3.20, and warns that installing the package (which bundles example data) often hits connection timeouts, mitigated with options(timeout = "300")
- The article describes the vendor-software comparison and biological validation only generically; the target-file format specification lives in the vignettes rather than the paper text

## Evidence

- [other] The user-provided target list is loaded into R via the createTargetList() function, which outputs a data frame object: [evidence span withheld — anchor status: not_found]
- [readme] R version requirement for running TARDIS: [evidence span withheld — anchor status: external_doc]
