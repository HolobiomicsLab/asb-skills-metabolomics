---
name: inter-batch-variation-assessment
description: Use when you have LC-MS or SRM proteomics runs with pooled QC injections
  interspersed at a fixed interval (e.g. every 5, 6, or 8 subject samples) and you
  need to measure and report how much unwanted inter-batch variation remains, before
  versus after batch correction such as QC-RFSC.
when_to_use_negative:
- The dataset has no repeated pooled QC injections, so RSD in QC samples cannot be
  computed.
- You only have a single feature table with no paired pre/post correction matrix —
  the skill is built around raw-vs-corrected comparison.
- You want within-subject biological variability rather than instrument/batch precision;
  RSD across QC injections is not a biological measure.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: Provides shiftCor/statAnalysis pipeline; its corrected matrix is the input
    for RSD-based inter-batch variation assessment
  repo: github.com/13479776/statTarget
- name: QC-RFSC
  role: Random forest signal correction method being evaluated by RSD curves for inter-batch
    variation removal
  repo: github.com/13479776/statTarget
- name: Skyline
  role: Source of SRM proteomics peak abundance tables (174 transitions, 69 proteins)
    used in the RSD assessment
- name: transX
  role: Converts Skyline/XCMS exports into statTarget input format for the correction
    and RSD analysis
  repo: github.com/13479776/statTarget
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:17a1df74d6b822e0babcf6a0aa9f48a2ecc634141e89a040a25ded88b96e7f76
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RFSC
  - Skyline
  - statTarget
  - transX
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# inter-batch-variation-assessment

## Summary

Quantify how much inter-batch (and intra-batch) signal variation persists in QC samples by computing per-feature relative standard deviation (RSD%) in QC injections and building cumulative RSD distribution curves, as implemented in statTarget for LC-MS metabolomics and SRM proteomics data.

## When to use

Use when you have LC-MS or SRM proteomics runs with pooled QC injections interspersed at a fixed interval (e.g. every 5, 6, or 8 subject samples) and you need to measure and report how much unwanted inter-batch variation remains, before versus after batch correction such as QC-RFSC.

## When NOT to use

- The dataset has no repeated pooled QC injections, so RSD in QC samples cannot be computed.
- You only have a single feature table with no paired pre/post correction matrix — the skill is built around raw-vs-corrected comparison.
- You want within-subject biological variability rather than instrument/batch precision; RSD across QC injections is not a biological measure.

## Inputs

- Feature abundance table (transitions x proteins or m/z x samples) from Skyline or XCMS export, compatible with statTarget transX
- Sample annotation/metadata file mapping each run to QC vs subject group and injection order (statTarget samPeno format)
- Raw (uncorrected) and batch-corrected (e.g. QC-RFSC) sample matrices for paired comparison

## Outputs

- Per-feature RSD% values in QC samples for raw and corrected data
- Cumulative RSD frequency distribution curves (Figure 4A-C style)
- Summaries such as 'proportion of peaks with RSD < 30 %' and fold-increases after correction

## How to apply

Take the QC sample rows from a feature abundance table (e.g. 174 transitions × 69 proteins from Skyline SRM export, or XCMS-extracted metabolite peaks) and compute RSD% = SD/mean for each feature across QC injections in both the raw and corrected matrices. Plot the cumulative frequency distribution of RSD values (Figure 4A–C style) so that a single curve at a chosen cutoff — typically RSD < 30 %, sometimes also 15 % — reports the proportion of features passing. Compare the pre/post curves and record the gain: QC-RFSC raised peaks with RSD < 30 % from 43.9 % to 86.2 % in metabolomics Dataset 1 and from 52.1 % to 90.9 % in Dataset 2, versus QC-RLSC and QC-SVR baselines (e.g. 68.4 % vs 47.7 % on the proteomics data). Judge success by whether the corrected curve shifts left and the fraction of features below the RSD cutoff increases substantially, and optionally confirm removal of inter-batch structure by running PCA (Pareto scaling via statAnalysis) on raw vs corrected matrices.

## Related tools

- **statTarget** (Provides shiftCor/statAnalysis pipeline; its corrected matrix is the input for RSD-based inter-batch variation assessment) — github.com/13479776/statTarget
- **QC-RFSC** (Random forest signal correction method being evaluated by RSD curves for inter-batch variation removal) — github.com/13479776/statTarget
- **Skyline** (Source of SRM proteomics peak abundance tables (174 transitions, 69 proteins) used in the RSD assessment)
- **transX** (Converts Skyline/XCMS exports into statTarget input format for the correction and RSD analysis) — github.com/13479776/statTarget

## Examples

```
R> shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- RSD is computed only on QC sample rows, not subject rows; verify the subset selection.
- The corrected curve should shift left of the raw curve in the cumulative RSD plot.
- Check that the reported proportion at the RSD cutoff (e.g. 30 %) rises meaningfully after correction (e.g. 43.9 % -> 86.2 %).
- PCA score plot on corrected data should show reduced inter-batch separation versus raw data.
- Cross-check against comparison methods (QC-RLSC, QC-SVR) run on the same QC injections.

## Limitations

- RSD < 30 % cutoffs are empirical precision targets, not guarantees of biological interpretability.
- Requires regularly interspersed QC injections (every 5–8 samples in the cited datasets); sparse QC sampling weakens the estimate.
- High RSD features may reflect either drift/batch variation or genuinely unstable analytes and still need filtering via quality assessment criteria.

## Evidence

- [results] QC-RFSC increased the proportion of proteomics peaks with RSDs less than 30% to 87.9%, outperforming QC-RLSC (68.4%) and QC-SVR (47.7%): [evidence span withheld — anchor status: not_found]
- [results] Proportion of peaks with RSDs less than 30 % increased significantly from 43.9% to 86.2 % after QC-RFSC method: [evidence span withheld — anchor status: not_found]
- [results] QC-RFSC on metabolomics dataset 2 raised peaks with RSD < 30 % to 90.9% from raw 52.1%: "The proportion of peaks with RSDs less than 30 % increased significantly to 90.9% after QC-RFSC compared with the raw data (52.1%)."
- [methods] statAnalysis call with Frule and scaling used for evaluation: "statAnalysis(file,Frule = 0.8, normM = "NONE", imputeM = "KNN", glog = TRUE,scaling = "Pareto")"
