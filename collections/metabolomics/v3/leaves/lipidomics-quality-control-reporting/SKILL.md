---
name: lipidomics-quality-control-reporting
description: Use when lipidome quantity tables (csv/xlsx pivot or flat, or mzTab-M)
  have been imported and you must verify that lipid intensity values follow the expected
  -digit reciprocal distribution before downstream analysis.
when_to_use_negative:
- The dataset has not yet been imported and lipid names standardized with Goslin —
  QC operates on the parsed quantity table, not raw vendor files
- You need statistical QC figures like PCA or clustering of samples — use LipidSpace's
  lipid space model and dendrogram views instead of the Benford check
- Quantities are already log-transformed or rank-normalized, which destroys the leading-digit
  distribution the test relies on
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidSpace
  role: Standalone workstation application providing the built-in Benford's-law QC
    function, interactive QC workflow, and example/tutorial datasets
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: C++ library used by LipidSpace to parse and standardize lipid names during
    data import prior to QC
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:5a6f8a1308cf7ed2b2ee6e18c159b2104b0299d704767436f37bea0d2583acb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LipidSpace
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipidomics-quality-control-reporting

## Summary

Quality control of imported lipidomics datasets in LipidSpace, centered on a Benford's-law leading-digit test of lipid quantity values plus interactive QC workflow guidance, to flag datasets whose quantitative distribution indicates unreliable measurement or excessive imputation.

## When to use

Use when lipidome quantity tables (csv/xlsx pivot or flat, or mzTab-M) have been imported and you must verify that lipid intensity values follow the expected leading-digit reciprocal distribution before downstream analysis.

## When NOT to use

- The dataset has not yet been imported and lipid names standardized with Goslin — QC operates on the parsed quantity table, not raw vendor files
- You need statistical QC figures like PCA or clustering of samples — use LipidSpace's lipid space model and dendrogram views instead of the Benford check
- Quantities are already log-transformed or rank-normalized, which destroys the leading-digit distribution the test relies on

## Inputs

- Lipidome quantity table in csv or xlsx format (pivot or flat layout, lipid species row- or column-wise), e.g. the bundled Example-Dataset.xlsx
- Lipidomics data files in mzTab-M format
- Sample-associated study variables (meta-information) attached to the lipidomes

## Outputs

- Structured QC report per lipidome containing observed leading-digit frequencies, expected Benford frequencies, and violation flags
- Per-lipidome Benford conformance verdict indicating datasets spanning too narrow a quantitative range
- Interactive QC workflow status guiding subsequent quality control steps

## How to apply

Load the lipidome quantity table via LipidSpace's Data Import, using the cppgoslin/Goslin library to parse and standardize lipid species names into valid shorthand identifiers. Apply the built-in 'Quality Control In LipidSpace' step from the integrated interactive workflow, which computes the first-digit distribution of the quantity values and compares it against the Benford expectation (frequency of digit d = log10(1 + 1/d)). A lipidome spanning several orders of magnitude should follow the reciprocal leading-digit distribution; violation suggests a low range of quantitative values or insufficient data imputation. Emit a structured QC report recording observed vs. expected digit frequencies and a per-lipidome conformance verdict, and use the interactive quality control tutorial to guide parameter selection. Only samples from comparable cohorts (e.g. healthy humans) should be pooled when evaluating QC across heterogeneous studies.

## Related tools

- **LipidSpace** (Standalone workstation application providing the built-in Benford's-law QC function, interactive QC workflow, and example/tutorial datasets) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (C++ library used by LipidSpace to parse and standardize lipid names during data import prior to QC) — https://github.com/lifs-tools/cppgoslin

## Examples

```
LipidSpace.exe  # then: File → Import example dataset, and follow the interactive quality control tutorial to run the Benford's-law QC on the loaded lipidomes
```

## Evaluation signals

- A lipidome with quantity values spanning several orders of magnitude should pass the QC check with no violation flagged
- The QC report records observed leading-digit frequencies against the expected Benford distribution log10(1 + 1/d)
- Violation flags should coincide with datasets showing a low range of quantitative values or insufficient data imputation
- All imported lipid names must resolve to valid shorthand identifiers via Goslin before QC verdicts are computed
- When evaluating QC across multiple published studies, restricting to a homogeneous subgroup (e.g. only samples from healthy humans) should be reflected in the evaluated sample set

## Limitations

- QC violation can arise from heterogeneous causes (low quantitative range or insufficient imputation) and requires manual interpretation rather than giving a single diagnosis
- Cohort heterogeneity confounds QC evaluation; the paper restricted QC comparison to samples from healthy humans within heterogeneous datasets
- At the time of the README, analyses cannot be saved or reloaded, so QC reports must be inspected/exported within a session

## Evidence

- [methods] LipidSpace includes a quality control function that tests whether the lipid quantity values of a loaded lipidome conform to Benford's law: "Quality Control In Lipidspace"
- [abstract] An integrated interactive workflow guides the user through the quality control steps.: "An integrated interactive workflow guides the user through the quality control steps."
- [methods] It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [results] For the heterogeneity of the cohorts, we only considered samples from healthy humans within the data sets.: "For the heterogeneity of the cohorts, we only considered samples from healthy humans within the data sets."
- [readme] LipidSpace supports both csv and xlsx files for import ... Further, lipidomics data files in mzTab-M format can be imported.: "LipidSpace supports both csv and xlsx files for import."
