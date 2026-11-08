---
name: percentage-delta-verification
description: Decision support for validating reported benchmark deltas — e.g., checking
  that a recomputed +6.3 pp improvement in correct molecular formula assignment and
  a 792→541 drop in wrongly assigned formulas reproduce from deposited source data.
when_to_use_negative:
- The comparison is across entirely different datasets or benchmark protocols, so
  a raw percentage-point delta would not be a like-for-like verification target.
- You only need the final reported number and have no deposited source counts to recompute
  from — a delta cannot be independently checked.
- Input is already a validated delta table with a prior verification pass; recomputation
  would add no information.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS 4
  role: The newer version whose top-1 molecular formula assignment counts are being
    compared against SIRIUS 3.0 to verify the reported deltas
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Baseline (SIRIUS 3.0) molecular formula identification setting whose counts
    provide the reference point for the percentage-point delta
  repo: https://github.com/sirius-ms/sirius/releases
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:66844149b4be31fc278cb9bf667a6ab532e264fa2bcbeffa25a9390c8c17bbba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# percentage-delta-verification

## Summary

Recompute reported percentage-point deltas and count reductions from deposited source data (e.g., Supplementary Fig. 6a / Supplementary Table 3 of the SIRIUS 4 paper) and verify they match published values within rounding tolerance, catching transcription or pipeline errors before results are reused.

## When to use

Decision support for validating reported benchmark deltas — e.g., checking that a recomputed +6.3 pp improvement in correct molecular formula assignment and a 792→541 drop in wrongly assigned formulas reproduce from deposited source data.

## When NOT to use

- The comparison is across entirely different datasets or benchmark protocols, so a raw percentage-point delta would not be a like-for-like verification target.
- You only need the final reported number and have no deposited source counts to recompute from — a delta cannot be independently checked.
- Input is already a validated delta table with a prior verification pass; recomputation would add no information.

## Inputs

- Deposited source data table for the published figure/table (e.g., Supplementary Fig. 6a / Supplementary Table 3 counts over 3,965 compounds)
- Reported values to verify: percentage-point deltas, absolute counts of correct and wrong assignments, and percentage reductions

## Outputs

- Comparison table with recomputed top-1 assignment counts and deltas per setting (e.g., formula_assignment_comparison.csv)
- Per-metric verification verdict: reproduced exactly / within rounding tolerance / mismatch flagged

## How to apply

Load the deposited source table underlying the published figure/table (here, the SIRIUS 4 evaluation on 3,965 compounds, Supplementary Fig. 6a / Supplementary Table 3) with pandas. Extract top-1 molecular formula assignment counts for each setting being compared — for SIRIUS 3.0 vs SIRIUS 4 this means the isotope-pattern-only setting and the combined MS1 + MS/MS setting. Compute the percentage-point deltas between the two versions' correct-assignment rates in each setting and the absolute reduction in wrongly assigned formulas, expressing the latter as a percentage. Assemble the recomputed counts and deltas into a single comparison table (e.g., formula_assignment_comparison.csv) and compare each value against the reported figure (+26.6 pp isotope-only, +6.3 pp MS1+MS/MS, 31.7% reduction in wrong assignments, with 792 and 541 reproduced exactly). Treat any mismatch beyond rounding tolerance as a flag that either the deposited data, the reported number, or the recomputation pipeline is inconsistent, and inspect before propagating the delta downstream.

## Related tools

- **SIRIUS 4** (The newer version whose top-1 molecular formula assignment counts are being compared against SIRIUS 3.0 to verify the reported deltas) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Baseline (SIRIUS 3.0) molecular formula identification setting whose counts provide the reference point for the percentage-point delta) — https://github.com/sirius-ms/sirius/releases

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('supplementary_table3.csv'); delta = df['sirius4_ms1_msms_top1'] - df['sirius30_ms1_msms_top1']; print(delta, (df['wrong_sirius30']-df['wrong_sirius4'])/df['wrong_sirius30']*100)"
```

## Evaluation signals

- Recomputed isotope-pattern-only delta matches the reported +26.6 percentage points within rounding tolerance
- Recomputed MS1 + MS/MS delta matches the reported +6.3 percentage points within rounding tolerance
- Wrongly assigned molecular formula counts reproduce 792 (SIRIUS 3.0) and 541 (SIRIUS 4) exactly
- Wrong-assignment percentage reduction matches the reported 31.7% within rounding tolerance
- The assembled comparison table contains counts for both the isotope-pattern-only and MS1+MS/MS settings with no missing cells

## Limitations

- Verification is only meaningful when the deposited source data corresponds to the exact figure/table being cited (e.g., Supplementary Fig. 6a vs Supplementary Table 3 may report different settings).
- Percentage-point deltas are sensitive to small count errors on large denominators; a mismatch may indicate a rounding convention difference rather than a genuine inconsistency.
- Counts of wrongly assigned formulas depend on the evaluation set composition (3,965 compounds here) and cannot be extrapolated to other benchmarks.

## Evidence

- [abstract] SIRIUS 4 improves correct formula assignments and reduces wrongly assigned formulas: "Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets."
- [methods] Combined MS1+MS/MS setting is the evaluation basis for the delta: "if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the parent ion"
- [methods] Molecular formula assignment is the measured output: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [intro] SIRIUS version comparison defines the baseline for the delta: "SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small mol"
