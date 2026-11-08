---
name: benchmark-criterion-value-validation
description: Use when a computed NOREVA criterion evaluation (Ca–Ce) for candidate
  processing workflows must be verified against the paper's Table 3 benchmark values
  and Good/Fair/Poor labels before trusting workflow rankings.
when_to_use_negative:
- The dataset is not one of the benchmark studies (e.g., MTBLS403, MTBLS656) with
  tabulated reference criterion values — there is no Table 3 ground truth to validate
  against, so run the assessment without this validation step.
- The input is still raw instrument data (e.g., proprietary vendor format) rather
  than the processed peak table produced by the upstream normalization stage.
- The goal is workflow discovery/optimization (scanning thousands of combinations)
  rather than verifying that criterion computations reproduce published benchmark
  values.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: R package (v2.1.1) providing the multi-criteria performance-assessment functions
    (e.g., normulticlassqcall, nortimecoursenoall) used to compute Ca–Ce and the Good/Fair/Poor
    classification
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Runtime environment in which the NOREVA protocol's benchmark validation is
    executed locally
  repo: https://github.com/idrblab/NOREVA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:43e6928a223ad014dc4537f28ae36b5dc5d30e44cfbdcfaa37d0aacb9623b5c5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# benchmark-criterion-value-validation

## Summary

Validates the implementation of NOREVA's five-criteria performance assessment (Ca PMAD, Cb purity, Cc CWrel, Cd AUC, Ce LogFC) by reproducing the tabulated criterion values and Good/Fair/Poor labels for the paper's benchmark workflows, writing results to criterion_scores.csv.

## When to use

Use when a computed NOREVA criterion evaluation (Ca–Ce) for candidate processing workflows must be verified against the paper's Table 3 benchmark values and Good/Fair/Poor labels before trusting workflow rankings.

## When NOT to use

- The dataset is not one of the benchmark studies (e.g., MTBLS403, MTBLS656) with tabulated reference criterion values — there is no Table 3 ground truth to validate against, so run the assessment without this validation step.
- The input is still raw instrument data (e.g., proprietary vendor format) rather than the processed peak table produced by the upstream normalization stage.
- The goal is workflow discovery/optimization (scanning thousands of combinations) rather than verifying that criterion computations reproduce published benchmark values.

## Inputs

- Processed peak table (output of the upstream NOREVA normalization stage) with rows as named samples and columns as metabolites
- Benchmark workflow outcomes whose Ca–Ce values and labels are tabulated in the article's Table 3
- Sample class/timepoint label file for time-course or multi-class datasets
- Column indices of internal standard (IS) metabolites, when reference/spike-in metabolites are available for criterion Ce

## Outputs

- criterion_scores.csv with per-criterion (Ca–Ce) values for each candidate workflow
- Good/Fair/Poor classification label per workflow, comparable to Table 3
- Pass/fail validation report confirming identical labels versus Table 3

## How to apply

Load the processed peak table output by the upstream normalization stage into R via the NOREVA R package, preserving the row (samples by name) and column structure. Compute the five criteria per workflow: criterion a (Ca, PMAD) for reproducibility of within-group variability, criterion b (Cb, purity), criterion c (Cc, CWrel), criterion d (Cd, AUC), and criterion e (Ce, LogFC). Skip Cc when any class or timepoint has fewer than 6 samples, and apply Ce only when reference (spike-in) metabolites are available. Aggregate the per-criterion results for each candidate workflow and apply the protocol's Good/Fair/Poor classification cutoffs. Correctness is judged by exact agreement: computed Ca–Ce values must reproduce the Table 3 tabulated values for the benchmark workflows, yield identical Good/Fair/Poor labels, and be written to criterion_scores.csv.

## Related tools

- **NOREVA** (R package (v2.1.1) providing the multi-criteria performance-assessment functions (e.g., normulticlassqcall, nortimecoursenoall) used to compute Ca–Ce and the Good/Fair/Poor classification) — https://github.com/idrblab/NOREVA
- **R** (Runtime environment in which the NOREVA protocol's benchmark validation is executed locally) — https://github.com/idrblab/NOREVA

## Examples

```
library(NOREVA); normulticlassqcall(fileName="processed_peak_table.csv", SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Computed Ca (PMAD) values for benchmark workflows match the Table 3 tabulated values exactly.
- Computed Cb (purity), Cc (CWrel), Cd (AUC), and Ce (LogFC) values match Table 3; Cc is absent for datasets with any class/timepoint of <6 samples and Ce is absent when no reference/spike-in metabolites exist.
- Good/Fair/Poor labels derived from the computed values are identical to the Table 3 labels for all benchmark workflows.
- Results are written to criterion_scores.csv with per-criterion values aggregated per workflow.
- The peak-table row/column structure (samples by name in rows) is preserved through the computation, as required by the protocol's standardized format.

## Limitations

- Criterion Cc (CWrel) is unsuitable for datasets with small sample size (<6) in one timepoint or class and must be skipped.
- Criterion Ce (LogFC) cannot be applied when no reference metabolite (e.g., spike-in) was provided — none of the paper's analyzed datasets could use criterion e.
- Partial assessment functions can overwrite output files from previous commands (identical file names to Step 10 outputs), producing misleading outputs; check output file naming when re-running.
- Assessment of thousands of workflows is time-consuming (hundreds to thousands of minutes even with parallel computing; >200 min for large datasets such as PMID28528106).

## Evidence

- [intro] The final stage of the NOREVA protocol assesses each candidate processing workflow collectively using five independent criteria, each supported by a well-known underlying theory.: [evidence span withheld — anchor status: not_found]
- [discussion] Cc small-sample exclusion: "the datasets of small sample size (<6) in one timepoint or class are unsuitable to be assessed using criterion Cc"
- [discussion] Ce requires reference metabolites: [evidence span withheld — anchor status: not_found]
- [discussion] Output overwrite hazard in partial assessment functions: [evidence span withheld — anchor status: not_found]
- [readme] README: five criteria with distinct theories and scan/rank capability: [evidence span withheld — anchor status: external_doc]
- [discussion] Manual peak-table standardization requirement: [evidence span withheld — anchor status: not_found]
