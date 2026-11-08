---
name: vendor-software-vs-ccsfind-cross-validation
description: Use when you have drift tube IM-MS DTCCS N2 values from two acquisition/processing
  routes (targeted CCSfind vs vendor non-targeted software) and need to quantify agreement
  as percentage bias per compound and per stratum.
when_to_use_negative:
- Data are traveling-wave IM-MS or TIMS-derived CCS rather than drift tube (DTCCS)
  measurements requiring single-field Tfix/Beta calibration
- You only have one processing route (either CCSfind or vendor values alone), so no
  vendor-vs-CCSfind bias can be computed
- Input is raw vendor feature lists rather than a table of matched compound-level
  DTCCS values ready for bias calculation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CCSfind
  role: Produces the targeted DTCCS N2 values from mzML input files that are compared
    against vendor non-targeted values
  repo: https://github.com/sangeeta97/ccs_find.git
- name: pandas
  role: DataFrame construction, stratification, and averaging of per-compound absolute
    biases
- name: numpy
  role: Numerical computation of absolute relative bias per compound entry
- name: XlsxWriter
  role: Writing the stratified bias summary table to CSV/Excel output (ccs_bias_summary.csv)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:e8ed81c63ace8278dc01bd3889827db322d9d28489d01f5099efa69cd7e6c828
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - XlsxWriter
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# vendor-software-vs-ccsfind-cross-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-validate CCSfind targeted DTCCS N2 values against vendor software non-targeted DTCCS N2 values by computing per-compound absolute relative bias and stratified averages (e.g., by ionization mode and acquisition setting), reproducing the Table 1 comparison of the CCSfind paper.

## When to use

Use when you have drift tube IM-MS DTCCS N2 values from two acquisition/processing routes (targeted CCSfind vs vendor non-targeted software) and need to quantify agreement as percentage bias per compound and per stratum.

## When NOT to use

- Data are traveling-wave IM-MS or TIMS-derived CCS rather than drift tube (DTCCS) measurements requiring single-field Tfix/Beta calibration
- You only have one processing route (either CCSfind or vendor values alone), so no vendor-vs-CCSfind bias can be computed
- Input is raw vendor feature lists rather than a table of matched compound-level DTCCS values ready for bias calculation

## Inputs

- Table of vendor (non-targeted) and CCSfind (targeted) DTCCS N2 values per compound, ionization mode, and acquisition/demultiplexing setting (e.g., Table 1 layout, CSV/Excel)
- CCSfind mzML-derived results for drift tube IM-MS (LC-DTIM-MS profile mzML input files)
- Single-field CCS calibration coefficients (Tfix and Beta) and drift gas identity (e.g., nitrogen) for the data files

## Outputs

- Per-compound absolute relative bias table (|CCSfind - vendor| / vendor x 100%)
- Stratified summary of average absolute bias by stratum (all data; positive 4IMS/HR; negative; single-pulse), e.g., ccs_bias_summary.csv written with pandas/XlsxWriter

## How to apply

Load the vendor-vs-CCSfind DTCCS N2 comparison table (as in the paper's Table 1, covering multiple compounds, acquisition modes, and demultiplexing levels) into a pandas DataFrame. Compute the absolute relative bias |CCSfind - vendor| / vendor x 100% for each compound entry. Stratify the per-compound biases by ionization mode and acquisition setting: all data, positive-mode 4IMS/HR, negative mode, and single-pulse standard settings. Average the absolute biases within each stratum with numpy/pandas and check the strata against the reported reference values (all data ~0.15%; positive 4IMS/HR avg 0.08%; negative avg 0.16%; single-pulse 0.35%-0.93%). Write the stratified summary with pandas/XlsxWriter (e.g., ccs_bias_summary.csv), consistent with the CCSfind toolchain that ships XlsxWriter, pandas, numpy, and polars.

## Related tools

- **CCSfind** (Produces the targeted DTCCS N2 values from mzML input files that are compared against vendor non-targeted values) — https://github.com/sangeeta97/ccs_find.git
- **pandas** (DataFrame construction, stratification, and averaging of per-compound absolute biases)
- **numpy** (Numerical computation of absolute relative bias per compound entry)
- **XlsxWriter** (Writing the stratified bias summary table to CSV/Excel output (ccs_bias_summary.csv))

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('table1.csv'); df['bias'] = (df.ccsfind - df.vendor).abs()/df.vendor*100; df.groupby('stratum')['bias'].mean().to_csv('ccs_bias_summary.csv')
```

## Evaluation signals

- Average absolute bias across all data reproduces ~0.15%
- Positive-mode 4IMS/HR stratum average is ~0.08% and negative-mode stratum average is ~0.16%
- Single-pulse standard-settings stratum falls in the reported 0.35%-0.93% range
- Every per-compound bias row equals |CCSfind - vendor| / vendor x 100% when recomputed independently
- ccs_bias_summary.csv contains one row per stratum with numeric averages and no NaN strata

## Limitations

- Bias magnitudes are mode- and acquisition-dependent: single-pulse standard settings show larger bias (0.35%-0.93%) than 4IMS/HR positive-mode data (0.08%), so stratum-level reporting is essential
- CCSfind requires single-field CCS calibration coefficients (Tfix and Beta), drift gas identity, and a profile mzML input file; mis-calibration propagates into the comparison
- Python 3.9 is required for the CCSfind environment; older/newer environments may not run the comparison pipeline

## Evidence

- [results] The average absolute bias between CCSfind (targeted) and vendor (non-targeted) DTCCS N2 values across all data in Table 1 was 0.15%: [evidence span withheld — anchor status: not_found]
