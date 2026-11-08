---
name: spreadsheet-data-ingestion
description: Use when a targeted LC–MS metabolomics/lipidomics workflow starts from
  a tabular target list (e.g., targets.xlsx in the TARDIS vignette) that must be loaded
  and standardized into the TARDIS data frame schema before peak integration.
when_to_use_negative:
- The input is raw LC–MS data (.raw/.mzML) needing targeted peak extraction — use
  TARDIS' integration/quality-assessment steps instead, which come after this ingestion
  step.
- The input is already a TARDIS-parsed data frame in the canonical schema (target
  ID, name, m/z, expected RT, polarity) — no spreadsheet parsing is needed.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TARDIS
  role: Provides the createTargetList() function that parses the spreadsheet into
    the target data frame used by downstream targeted peak integration and quality-metric
    functions
  repo: github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (>= 4.4.0) in which the spreadsheet is read and the TARDIS
    package is installed and executed
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:ac732f28ba3214292f163cc96115b0986628f94ad748c68779a712aa5e86a0c9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spreadsheet-data-ingestion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ingest a user-supplied target compound spreadsheet (.xlsx or .csv) into R as a structured data frame via TARDIS' createTargetList() parsing mechanism, yielding the canonical target schema (target ID, name, m/z, expected RT, ion polarity) that feeds TARDIS' targeted peak integration and quality assessment.

## When to use

Use when a targeted LC–MS metabolomics/lipidomics workflow starts from a tabular target list (e.g., targets.xlsx in the TARDIS vignette) that must be loaded and standardized into the TARDIS data frame schema before peak integration.

## When NOT to use

- The input is raw LC–MS data (.raw/.mzML) needing targeted peak extraction — use TARDIS' integration/quality-assessment steps instead, which come after this ingestion step.
- The input is already a TARDIS-parsed data frame in the canonical schema (target ID, name, m/z, expected RT, polarity) — no spreadsheet parsing is needed.

## Inputs

- User-supplied target list spreadsheet (.xlsx or .csv) with columns for identifier, compound name, m/z, expected retention time, and ionization-pattern/polarity information
- TARDIS vignette targets.xlsx example file

## Outputs

- R data frame of parsed targets in the canonical schema (target ID, name, m/z, expected RT, polarity), suitable as input to downstream TARDIS peak-integration functions
- Exported targets_parsed.csv file

## How to apply

Load the shipped targets.xlsx file into an R data frame using a spreadsheet reader, then invoke TARDIS' createTargetList() parsing mechanism, which converts the table into the data frame object consumed by all downstream TARDIS functions. Filter/annotate rows using the ionization-pattern columns to assign each target its polarity (positive or negative mode), and standardize output columns to the canonical schema: target ID, name, m/z, expected RT, and polarity. Confirm the parse succeeded by checking that every target row has non-missing m/z, RT, and polarity values before passing the data frame onward. TARDIS requires R >= 4.4.0 (and BiocManager 3.20 for the latest version), so verify the environment before loading the package with library(TARDIS).

## Related tools

- **TARDIS** (Provides the createTargetList() function that parses the spreadsheet into the target data frame used by downstream targeted peak integration and quality-metric functions) — github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (>= 4.4.0) in which the spreadsheet is read and the TARDIS package is installed and executed) — https://cloud.r-project.org/index.html

## Examples

```
targets <- readxl::read_excel("targets.xlsx"); targets_parsed <- TARDIS::createTargetList(targets); write.csv(targets_parsed, "targets_parsed.csv")
```

## Evaluation signals

- The output is an R data frame (the createTargetList() output object), not a raw import of the spreadsheet.
- Every target row has non-missing m/z, expected RT, and polarity values after parsing.
- Output columns match the canonical schema: target ID, name, m/z, expected RT, and polarity.
- Each target's polarity matches the ionization-pattern columns (positive/negative mode assignment).
- The parsed data frame is accepted as input by further TARDIS functions without schema errors.

## Limitations

- The article does not state a TARDIS package version number, so reproduce against the GitHub main branch and check the R >= 4.4.0 and BiocManager 3.20 requirements from the README.
- Package installation may hit connection timeouts because the package contains example data; the README recommends options(timeout = '300').
- The article's benchmark descriptions are generic (no named vendor software versions), so cross-validation of the parsed target schema against vendor tooling is not directly documented.

## Evidence

- [other] The user-provided target list is loaded into R via the createTargetList() function, which outputs a data frame object that is used as input to further functions in TARDIS.: [evidence span withheld — anchor status: not_found]
- [other] Load the targets.xlsx file shipped with the TARDIS vignette into an R data frame using a spreadsheet reader.: [evidence span withheld — anchor status: not_found]
