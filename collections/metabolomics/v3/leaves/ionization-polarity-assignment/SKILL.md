---
name: ionization-polarity-assignment
description: 'Use when preparing a targeted LC–MS metabolomics/lipidomics target list
  for TARDIS: the input is a spreadsheet of candidate analytes with identifiers, m/z,
  expected RT and ionization-pattern columns that must be parsed and polarity-assigned
  before peak integration.'
when_to_use_negative:
- The input is already a TARDIS-formatted target data frame with polarity assigned
  — re-running parsing is redundant
- The task is untargeted preprocessing (e.g., xcms, MZmine, OpenMS feature finding)
  rather than building a targeted analyte list
- The raw data were not acquired with defined ionization modes, so polarity cannot
  be assigned from ionization-pattern columns
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TARDIS
  role: R package whose createTargetList() parsing mechanism ingests the user target
    table and produces the data frame used by further TARDIS functions
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (>= 4.4.0) in which the target spreadsheet is read and
    parsed into the TARDIS target data frame
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:79524fbececd58a89eba306ffe791e18e0c3ebb7615fcc222d0ff5677405c739
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ionization-polarity-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses a user-supplied target list (.xlsx/.csv) via TARDIS' createTargetList() into a standardized R data frame carrying target ID, name, m/z, expected RT, and ionization polarity (positive/negative mode), which serves as the target input for TARDIS' downstream targeted peak integration of LC–MS data.

## When to use

Use when preparing a targeted LC–MS metabolomics/lipidomics target list for TARDIS: the input is a spreadsheet of candidate analytes with identifiers, m/z, expected RT and ionization-pattern columns that must be parsed and polarity-assigned before peak integration.

## When NOT to use

- The input is already a TARDIS-formatted target data frame with polarity assigned — re-running parsing is redundant
- The task is untargeted preprocessing (e.g., xcms, MZmine, OpenMS feature finding) rather than building a targeted analyte list
- The raw data were not acquired with defined ionization modes, so polarity cannot be assigned from ionization-pattern columns

## Inputs

- User-supplied target list in .xlsx or .csv format (columns: target identifier, compound name, m/z, expected RT, ionization-pattern columns)
- R (>= 4.4.0) environment with the TARDIS package installed

## Outputs

- Standardized R data frame with columns: target ID, name, m/z, expected RT, ion polarity (positive/negative)
- Parsed target table file (e.g., targets_parsed.csv) validated for non-missing m/z, RT, and polarity
- Data frame suitable as input to downstream TARDIS peak-integration functions

## How to apply

Load the target spreadsheet (e.g., the targets.xlsx shipped with the TARDIS vignette) into an R data frame, then invoke TARDIS' createTargetList() parsing mechanism to produce the canonical schema: target ID, name, m/z, expected RT, and polarity. Use the ionization-pattern columns to assign each target its polarity (positive/negative mode), since TARDIS integrates targeted chromatographic peaks from LC–MS raw data acquired per ionization mode. Validate that every target row has non-missing m/z, RT, and polarity before writing the parsed table (e.g., targets_parsed.csv), because this data frame is the input to TARDIS' further functions. Refer to the package vignettes (quick_start, gui_tutorial, case_study) for the exact expected column layout.

## Related tools

- **TARDIS** (R package whose createTargetList() parsing mechanism ingests the user target table and produces the data frame used by further TARDIS functions) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (>= 4.4.0) in which the target spreadsheet is read and parsed into the TARDIS target data frame)

## Examples

```
targets <- readxl::read_excel("targets.xlsx"); target_list <- TARDIS::createTargetList(targets)
```

## Evaluation signals

- Every target row in the parsed data frame has non-missing m/z, expected RT, and polarity values
- Output columns match the canonical schema exactly: target ID, name, m/z, expected RT, polarity
- Each target is assigned exactly one polarity (positive or negative mode) derived from its ionization-pattern column(s)
- Row count of the parsed data frame equals the number of non-empty rows in the source .xlsx/.csv file
- The parsed data frame is accepted without error as input by TARDIS' downstream peak-integration functions

## Limitations

- Connection timeouts can occur when installing TARDIS because the package contains example data; increase the R timeout option (e.g., options(timeout = "300"))
- Windows users additionally need Rtools, and Mac users need Xcode and a GNU Fortran compiler, to install and run the parsing workflow
- The article does not state a version number for the TARDIS R package; behavior of createTargetList() should be verified against the installed package's vignettes

## Evidence

- [other] createTargetList parsing: [evidence span withheld — anchor status: not_found]
- [readme] vignette guidance: [evidence span withheld — anchor status: external_doc]
- [readme] version requirement: [evidence span withheld — anchor status: external_doc]
