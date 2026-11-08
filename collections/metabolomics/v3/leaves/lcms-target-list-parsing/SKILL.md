---
name: lcms-target-list-parsing
description: Use when a targeted LC–MS metabolomics/lipidomics analysis needs a compound
  target table (identifier, name, m/z, expected RT, polarity) converted into TARDIS'
  expected data frame input.
when_to_use_negative:
- Input is already a processed TARDIS results object (a list containing a data.frame
  with AUC per target per run) rather than a raw target table
- The goal is untargeted preprocessing, where tools like xcms, MZmine or OpenMS are
  the article's cited alternatives
- The file lacks m/z or expected RT values — every target row must have non-missing
  m/z, RT, and polarity to be usable
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: TARDIS
  role: Provides the createTargetList() function that parses the raw target table
    into the data frame consumed by downstream integration functions
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Runtime environment (>= 4.4.0 per README) in which the target file is loaded
    and parsed
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:d9f87c069b36492dfc39909697d2defbb1218d87dfc18bd3372328f12b883e20
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# lcms-target-list-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses a user-supplied target list (.xlsx/.csv) into the canonical TARDIS data frame schema — target ID, name, m/z, expected RT, and ion polarity — so it can drive targeted peak integration in TARDIS.

## When to use

Use when a targeted LC–MS metabolomics/lipidomics analysis needs a compound target table (identifier, name, m/z, expected RT, polarity) converted into TARDIS' expected data frame input.

## When NOT to use

- Input is already a processed TARDIS results object (a list containing a data.frame with AUC per target per run) rather than a raw target table
- The goal is untargeted preprocessing, where tools like xcms, MZmine or OpenMS are the article's cited alternatives
- The file lacks m/z or expected RT values — every target row must have non-missing m/z, RT, and polarity to be usable

## Inputs

- User-supplied target list file in .xlsx or .csv format
- Columns: compound identifier, compound name, m/z, expected retention time, ionization-pattern/polarity columns

## Outputs

- R data frame (canonical target-list schema: target ID, name, m/z, expected RT, polarity)
- Optional serialized targets_parsed.csv with one validated row per target

## How to apply

Load the targets.xlsx (or .csv) table into an R data frame using a spreadsheet reader such as readxl. Invoke TARDIS' createTargetList() parsing mechanism, which outputs a data frame used as input to further TARDIS functions. Use the ionization-pattern columns to assign each target its polarity (positive/negative mode), and standardize columns to the canonical schema: target ID, name, m/z, expected RT, polarity. Verify completeness before downstream integration, since TARDIS' peak integration and quality metrics (AUC, maximum intensity, points over the peak, signal-to-noise, bell-curve similarity) all key off this target frame.

## Related tools

- **TARDIS** (Provides the createTargetList() function that parses the raw target table into the data frame consumed by downstream integration functions) — https://github.com/UGent-LIMET/TARDIS
- **R** (Runtime environment (>= 4.4.0 per README) in which the target file is loaded and parsed) — https://cloud.r-project.org/index.html

## Examples

```
targets <- readxl::read_excel("targets.xlsx"); targetList <- TARDIS::createTargetList(targets)
```

## Evaluation signals

- Every target row in the parsed data frame has non-missing m/z, RT, and polarity values
- Output is an R data.frame object, matching TARDIS' expected input type for further functions
- Polarity assignment is complete for every row via the ionization-pattern columns (positive/negative mode)
- Column names match the canonical schema: target ID, name, m/z, expected RT, polarity
- Parsed target count equals the row count of the source .xlsx/.csv file (no rows dropped silently)

## Limitations

- Requires R >= 4.4.0 and, for installation, BiocManager 3.20; Windows users additionally need Rtools and Mac users Xcode/GNU Fortran
- Package installation may hit connection timeouts because the package contains example data; increase the R timeout (e.g. options(timeout = 300))
- The article does not state a TARDIS package version number or validated parameter thresholds for the parsing step itself

## Evidence

- [other] createTargetList loads user target list into R data frame: [evidence span withheld — anchor status: not_found]
- [readme] README points to vignettes for usage tutorial: [evidence span withheld — anchor status: external_doc]
- [readme] README installation prerequisites: [evidence span withheld — anchor status: external_doc]
