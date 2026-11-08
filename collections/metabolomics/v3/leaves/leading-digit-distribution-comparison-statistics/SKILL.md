---
name: leading-digit-distribution-comparison-statistics
description: Use when importing a lipidome quantity table (csv/xlsx, pivot or flat,
  or mzTab-M) and you need a global QC verdict on whether quantitative values plausibly
  span several orders of magnitude before downstream analysis.
when_to_use_negative:
- The input is raw mass-spectrometric peak data rather than a tabulated lipidome quantity
  table — this check operates on lipid quantity values after lipid search-engine identification
  (e.g., LipidXplorer, LDA, MS-DIAL output).
- The goal is per-lipid or per-sample variance QC rather than a dataset-level plausibility
  check on quantitative value ranges — Benford conformance is a global leading-digit
  test, not a replicate or batch QC method.
- Quantities are ratios, log-transformed, or otherwise rescaled so leading digits
  no longer reflect measurement magnitude — the test presumes raw quantitative values
  spanning orders of magnitude.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: LipidSpace
  role: Standalone tool whose built-in quality control module performs the Benford's-law
    leading-digit check on imported lipidome quantities and reports violations
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin
  role: C++ library (cppgoslin) used to parse and standardize lipid names so rows
    of the quantity table map to valid lipid identifiers before QC
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:21b025b1db34fb668274d9429cff13da17c4b94f67c74209f6eea2c29417a266
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - LipidSpace
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# leading-digit-distribution-comparison-statistics

## Summary

Quality control check, as implemented in LipidSpace's built-in QC, that tests whether the leading-digit distribution of lipid quantity values in a loaded lipidome conforms to Benford's law (expected frequency of digit d = log10(1 + 1/d)), flagging datasets whose quantities span too narrow a range or are over-imputed.

## When to use

Use when importing a lipidome quantity table (csv/xlsx, pivot or flat, or mzTab-M) and you need a global QC verdict on whether quantitative values plausibly span several orders of magnitude before downstream analysis.

## When NOT to use

- The input is raw mass-spectrometric peak data rather than a tabulated lipidome quantity table — this check operates on lipid quantity values after lipid search-engine identification (e.g., LipidXplorer, LDA, MS-DIAL output).
- The goal is per-lipid or per-sample variance QC rather than a dataset-level plausibility check on quantitative value ranges — Benford conformance is a global leading-digit test, not a replicate or batch QC method.
- Quantities are ratios, log-transformed, or otherwise rescaled so leading digits no longer reflect measurement magnitude — the test presumes raw quantitative values spanning orders of magnitude.

## Inputs

- Lipidome quantity/intensity table (csv or xlsx, flat or pivot layout; mzTab-M also supported by LipidSpace)
- Lipid species name column parsed and standardized via Goslin (e.g., PC 18:0_16:1 shorthand identifiers)

## Outputs

- Structured QC report (e.g., benford_qc_report.json) with observed leading-digit frequencies, expected Benford frequencies log10(1 + 1/d), and per-sample conformance verdicts
- Per-lipidome violation flags (conform / violation, e.g. low quantitative range or insufficient data imputation)

## How to apply

Load the lipidome quantity table and standardize every lipid species name with the Goslin library (cppgoslin in LipidSpace) so each row maps to a valid lipid shorthand identifier, extracting the intensity/quantity column per sample. Compute the leading-digit distribution of the quantity values and compare it against the Benford expectation, where the expected frequency of digit d is log10(1 + 1/d). Flag a dataset as a violation when its first-digit frequencies do not conform to the Benford distribution, since lipidomics observations spanning several orders of magnitude should follow the leading-digit reciprocal distribution; a narrow value range or excessive data imputation produces deviations. Record per-sample conformance verdicts and the observed vs. expected digit frequencies in a structured QC report that an integrated interactive workflow guides the user through, as in LipidSpace's Quality Control section.

## Related tools

- **LipidSpace** (Standalone tool whose built-in quality control module performs the Benford's-law leading-digit check on imported lipidome quantities and reports violations) — https://github.com/lifs-tools/lipidspace
- **Goslin** (C++ library (cppgoslin) used to parse and standardize lipid names so rows of the quantity table map to valid lipid identifiers before QC) — https://github.com/lifs-tools/cppgoslin

## Examples

```
import math; expected = {d: math.log10(1 + 1/d) for d in range(1, 10)}
```

## Evaluation signals

- The QC report is parseable JSON (LipidSpace itself embeds nlohmann/json) and contains both observed and expected leading-digit distributions.
- A dataset known to span several orders of magnitude (e.g., the bundled example dataset) passes with no violation flagged, consistent with LipidSpace's rule that multi-order-of-magnitude lipidomics data follow the reciprocal leading-digit distribution.
- Expected frequencies sum to 1 across digits d = 1..9 under log10(1 + 1/d), and observed frequencies also sum to 1 over all quantity values.
- Per-sample verdicts are present for every loaded lipidome, not only for the aggregate table.
- A synthetic quantity vector confined to a narrow range (or heavily imputed) is flagged as a violation.

## Limitations

- Non-conformance is only indicative: a violation signals a low range of quantitative values or insufficient data imputation, not necessarily a data-processing error.
- The check is dataset-level; it cannot localize which lipid species or samples caused the deviation and must be complemented by the other QC steps of the integrated interactive workflow.
- Any transformation of quantities (log-scaling, normalization) before the test invalidates the leading-digit assumption.

## Evidence

- [abstract] An integrated interactive workflow guides the user through the quality control steps.: "An integrated interactive workflow guides the user through the quality control steps."
- [methods] Goslin parsing of lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [readme] README statement of QC capability: "It allows for a rapid (re)analysis of experiments, identifies lipids responsible for shaping the respective lipidome, and provides methods for quality control."
- [readme] README on import formats: "LipidSpace supports both csv and xlsx files for import. The tabels can be either pivot tables (that is lipid species in columns, samples in rows or lipid species in rows and samples in columns) or"
