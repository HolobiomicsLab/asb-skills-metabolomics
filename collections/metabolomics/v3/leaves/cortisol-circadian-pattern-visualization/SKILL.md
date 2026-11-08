---
name: cortisol-circadian-pattern-visualization
description: Use when a NOREVA time-course workflow scan has been completed on a peak
  table (e.g., MTBLS656, T0–T48) and you need biological sanity-check evidence that
  the top-ranked workflow preserves a known circadian pattern (cortisol) that a low-ranked
  workflow distorts.
when_to_use_negative:
- The dataset is case–control or multiclass rather than time-course — use NOREVA's
  multiclass functions instead of the time-course scan and cortisol-pattern check.
- A timepoint/class has fewer than 6 samples, making criterion Cc unsuitable and weakening
  the workflow ranking that this visualization depends on.
- The input is raw instrument data rather than a standardized peak table — peak-table
  generation (Proteowizard/XCMS) and manual row/column standardization must happen
  first.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: R package used to scan the 1,968 candidate time-course processing workflows
    and rank them; its time-course functions produce the ranked outputs from which
    the cortisol traces are extracted
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Execution environment for the NOREVA protocol and for assembling the two cortisol
    traces into the comparison figure and CSV output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:a0d45e9096e6beb20d953cc63f6d61856de2200094fe1d7e5340fe644b9779e0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - MetaboLights MTBLS656
  - NOREVA
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cortisol-circadian-pattern-visualization

## Summary

Biological-validation visualization skill for the NOREVA protocol: after scanning ~1,968 processing workflows on the MTBLS656 plasma time-course peak table, extract the cortisol intensity time series (T0–T48) from the first-ranked (MEI+NON+CUT+[QUA-LEV]) and last-ranked (KNN+NON+LOG+[PAR-SUM]) workflows and plot them side-by-side to show that the optimal workflow preserves the reported circadian rise/fall while the worst one distorts it.

## When to use

Use when a NOREVA time-course workflow scan has been completed on a peak table (e.g., MTBLS656, T0–T48) and you need biological sanity-check evidence that the top-ranked workflow preserves a known circadian pattern (cortisol) that a low-ranked workflow distorts.

## When NOT to use

- The dataset is case–control or multiclass rather than time-course — use NOREVA's multiclass functions instead of the time-course scan and cortisol-pattern check.
- A timepoint/class has fewer than 6 samples, making criterion Cc unsuitable and weakening the workflow ranking that this visualization depends on.
- The input is raw instrument data rather than a standardized peak table — peak-table generation (Proteowizard/XCMS) and manual row/column standardization must happen first.

## Inputs

- MTBLS656 plasma time-course peak table (sample rows, feature columns, time-point labels T0–T48) in NOREVA's unified format, from MetaboLights or NOREVA_exampledata.zip
- NOREVA time-course scan ranking output (all 1,968 workflows with multi-criteria performance scores)
- Processed data matrices from the first-ranked (MEI+NON+CUT+[QUA-LEV]) and last-ranked (KNN+NON+LOG+[PAR-SUM]) workflows

## Outputs

- figure5_cortisol_pattern.png — comparison panel of cortisol traces from first- vs last-ranked workflows across T0–T48
- CSV table of the underlying processed cortisol intensity values for both workflows
- Verified workflow ranking file (first and last entries checked)

## How to apply

First run the NOREVA time-course scan (e.g., nortimecoursenoall or nortimecourseqcall) over the candidate workflows combining S1 filtering/imputation (MEI, KNN, MIN, CUT), S2 QCS correction (NON), S3 transformation (LOG, NON) and S4 normalization (QUA-LEV, PAR-SUM), and obtain the ranked output. Verify the ranking file lists MEI+NON+CUT+[QUA-LEV] first and KNN+NON+LOG+[PAR-SUM] last. From each workflow's processed output, extract the cortisol feature's intensity values across all time points T0–T48. Plot the first-ranked trace against the reported circadian pattern of cortisol and overlay the last-ranked trace in a single Figure 5-style comparison panel; the expected result is a preserved rise/fall profile for the first-ranked workflow and a distorted profile for the last-ranked. Save the panel as figure5_cortisol_pattern.png with the underlying processed intensities written to an accompanying CSV table.

## Related tools

- **NOREVA** (R package used to scan the 1,968 candidate time-course processing workflows and rank them; its time-course functions produce the ranked outputs from which the cortisol traces are extracted) — https://github.com/idrblab/NOREVA
- **R** (Execution environment for the NOREVA protocol and for assembling the two cortisol traces into the comparison figure and CSV output)

## Examples

```
Rscript -e "library(NOREVA); res <- nortimecoursenoall('MTBLS656_peaktable.csv', SAalpha='Y', SAbeta='Y', SAgamma='Y')"
```

## Evaluation signals

- The produced ranking file lists MEI+NON+CUT+[QUA-LEV] as the first-ranked and KNN+NON+LOG+[PAR-SUM] as the last-ranked of the 1,968 scanned workflows.
- The first-ranked workflow's cortisol trace qualitatively matches the reported circadian pattern (rise/fall) across T0–T48.
- The last-ranked workflow's cortisol trace visibly distorts the circadian pattern relative to the first-ranked trace.
- The output CSV contains processed cortisol intensities for both workflows across every time point T0–T48 with no missing entries.
- The saved figure5_cortisol_pattern.png contains both traces in a single comparison panel.

## Limitations

- The workflow scan underlying this visualization is time-consuming: 'the time spent applying the NOREVA protocol varies from hundreds to thousands of minutes' even with parallel computing.
- Validation is qualitative — no reference metabolite (spike-in) was available in the analyzed datasets, so criterion Ce (LogFC) could not be used and correctness rests on visual pattern matching.
- The peak table must be manually standardized to NOREVA's unified format, which is 'relatively time consuming' due to diverse raw data file organizations.
- Well-performing workflows are dataset-dependent, so the first/last-ranked workflow identities for MTBLS656 cannot be assumed to transfer to other time-course datasets.

## Evidence

- [readme] time-course scan capability: [evidence span withheld — anchor status: external_doc]
- [readme] time-course function invocation: [evidence span withheld — anchor status: external_doc]
- [intro] combinatorial workflow diversity rationale: [evidence span withheld — anchor status: not_found]
- [discussion] runtime caveat: "the time spent applying the NOREVA protocol varies from hundreds to thousands of minutes, which demands a significant acceleration"
- [supplementary] example data availability: "All data used in this publication have been made available on the NOREVA website (https://idrblab.org/noreva/NOREVA_exampledata.zip)"
