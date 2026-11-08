---
name: benford-law-conformance-testing-of-lipid-quantity-values
description: Use when a lipidome quantity table (csv/xlsx flat or pivot table, or
  mzTab-M) is loaded into LipidSpace and you must verify that the distribution of
  digits in the lipid quantity values follows Benford's law as part of the built-in
  import quality control workflow.
when_to_use_negative:
- The goal is to identify which lipids distinguish study variables or produce result
  figures — use LipidSpace's feature analysis/statistics modules instead of the Benford
  QC check.
- The goal is structural comparison or lipidome similarity (MCS + Jaccard distance,
  lipid space models) rather than data-integrity screening of quantity values.
- The input is not raw lipid quantities — e.g., already log-transformed, normalized,
  or rank-based values, for which the leading-digit distribution is meaningless.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidSpace
  role: Standalone workstation application whose built-in quality control function
    performs the Benford's-law test on loaded lipidome quantity values and emits the
    QC report
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid names during data import so each quantity row
    maps to a valid lipid identifier before the QC check runs
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:c6c28557ee910541f7bd48f9bd232eb78a82de78ec1152881b423c7bd1b857b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LipidSpace
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benford-law-conformance-testing-of-lipid-quantity-values

## Summary

A data-integrity QC check from LipidSpace that tests whether the leading digits of lipid quantity values in an imported lipidome conform to Benford's law, flagging lipidomes whose quantity distributions look implausible. It catches datasets with a suspiciously narrow quantitative range or insufficient imputation before downstream lipidome comparison.

## When to use

Use when a lipidome quantity table (csv/xlsx flat or pivot table, or mzTab-M) is loaded into LipidSpace and you must verify that the distribution of leading digits in the lipid quantity values follows Benford's law as part of the built-in import quality control workflow.

## When NOT to use

- The goal is to identify which lipids distinguish study variables or produce result figures — use LipidSpace's feature analysis/statistics modules instead of the Benford QC check.
- The goal is structural comparison or lipidome similarity (MCS + Jaccard distance, lipid space models) rather than data-integrity screening of quantity values.
- The input is not raw lipid quantities — e.g., already log-transformed, normalized, or rank-based values, for which the leading-digit distribution is meaningless.

## Inputs

- Lipidome quantity table in csv or xlsx format, as a flat or pivot table (lipid species row-wise and samples column-wise, or vice versa)
- Lipidomics data in mzTab-M format
- Bundled example dataset (e.g., examples/Example-Dataset.xlsx, importable via File → Import example dataset)

## Outputs

- Structured QC report with observed leading-digit frequencies, expected Benford frequencies, and a per-lipidome conformance verdict
- Flagged violations for lipidomes whose leading-digit distribution does not conform to Benford's law

## How to apply

Load the lipidome quantity table into LipidSpace (File → Import, supporting flat or pivot layouts in csv/xlsx as well as mzTab-M), so that lipid species names are parsed and standardized with the Goslin C++ library and each row maps to a valid lipid shorthand identifier with a per-sample quantity column. Then run LipidSpace's built-in Benford quality control from the interactive QC workflow: compute the leading-digit frequency distribution of the quantity values and compare it against the Benford expectation log10(1 + 1/d) for each digit d. Flag as a violation any loaded lipidome whose leading-digit distribution does not conform; per the paper, lipidomics observations spanning several orders of magnitude should follow the leading-digit reciprocal distribution. Interpret violations as indicators of a low range of quantitative values or insufficient data imputation rather than automatically as analysis errors, and consult the emitted structured QC report (observed vs. expected digit frequencies with a conformance verdict per lipidome).

## Related tools

- **LipidSpace** (Standalone workstation application whose built-in quality control function performs the Benford's-law test on loaded lipidome quantity values and emits the QC report) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (Parses and standardizes lipid names during data import so each quantity row maps to a valid lipid identifier before the QC check runs) — https://github.com/lifs-tools/cppgoslin

## Evaluation signals

- The QC report is structured and parseable and contains both the observed leading-digit frequencies and the expected Benford frequencies (log10(1 + 1/d)).
- The report carries a conformance verdict per lipidome/sample, not only a global verdict.
- A dataset known to span several orders of magnitude (e.g., the bundled example dataset) passes with no violation flagged, consistent with LipidSpace's rule that lipidomics observations over several orders of magnitude follow the leading-digit reciprocal distribution.
- The bundled example dataset imports cleanly via File → Import example dataset before QC is run.
- Flagged violations can be traced to a plausible cause, such as a narrow range of quantitative values or insufficient data imputation.

## Limitations

- A Benford violation is not proof of error — it is indicative of a low range of quantitative values or insufficient data imputation, so the verdict must be interpreted in context.
- The check is embedded in LipidSpace's interactive workflow; per the README, analyses (including QC results) cannot currently be stored or loaded, so the report must be captured during the session.
- The test presupposes a large set of lipid quantity values per lipidome; the paper's own QC evaluation was restricted to samples from healthy humans to control cohort heterogeneity.

## Evidence

- [abstract] An integrated interactive workflow guides the user through the quality control steps.: "An integrated interactive workflow guides the user through the quality control steps."
- [methods] It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [results] we evaluated in the third phase seven published human plasma lipidomics experiments within four studies27,28,33,34 measured with different acquisition techniques for QC.: [evidence span withheld — anchor status: not_found]
- [readme] allows for a rapid (re)analysis of experiments, identifies lipids responsible for shaping the respective lipidome, and provides methods for quality control: "allows for a rapid (re)analysis of experiments, identifies lipids responsible for shaping the respective lipidome, and provides methods for quality control"
