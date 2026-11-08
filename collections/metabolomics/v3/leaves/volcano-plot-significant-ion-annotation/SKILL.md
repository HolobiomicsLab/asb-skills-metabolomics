---
name: volcano-plot-significant-ion-annotation
description: Use when a trained/adapted MetaPhenotype model classifies SCMS m/z peaks
  into cell phenotypes and you need to flag which ions are essential discriminators
  for volcano-plot annotation.
when_to_use_negative:
- You need statistically significant ions from fold-change and p-value thresholds
  alone (a standard volcano-plot analysis) — this skill flags ions by their effect
  on MetaPhenotype classification accuracy, not by statistical significance.
- The trained model has not yet been adapted to the target cell pair — baseline accuracy
  must be established before any removal test can be interpreted.
- The SCMS data have not undergone the pretreatment protocol (noise subtraction, background
  subtraction, TIC normalization, peak alignment with 10 ppm tolerance); unprocessed
  spectra will give unreliable accuracy drops.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning model whose adapted checkpoints provide the baseline
    and post-removal classification accuracies used to flag essential ions
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Source of the single-cell m/z feature matrix whose peaks are individually
    removed in the significance test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:9cbd23bfa1ab08f8ddec74ce5a811d437d4834d823edfef2bb862129cd17172e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GitHub (MetaPhenotype repository)
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# volcano-plot-significant-ion-annotation

## Summary

Identify the small set of essential m/z species whose removal degrades MetaPhenotype's cell-phenotype classification accuracy by at least 1%, then overlay them as bold dots on the SCMS volcano plots for each cell-pair comparison.

## When to use

Use when a trained/adapted MetaPhenotype model classifies SCMS m/z peaks into cell phenotypes and you need to flag which ions are essential discriminators for volcano-plot annotation.

## When NOT to use

- You need statistically significant ions from fold-change and p-value thresholds alone (a standard volcano-plot analysis) — this skill flags ions by their effect on MetaPhenotype classification accuracy, not by statistical significance.
- The trained model has not yet been adapted to the target cell pair — baseline accuracy must be established before any removal test can be interpreted.
- The SCMS data have not undergone the pretreatment protocol (noise subtraction, background subtraction, TIC normalization, peak alignment with 10 ppm tolerance); unprocessed spectra will give unreliable accuracy drops.

## Inputs

- Pretreated SCMS feature matrix (dataset.csv-style m/z peak intensities from Single-probe SCMS on melanoma cell lines)
- Adapted MetaPhenotype model checkpoints (1-shot and 5-shot adapted to drug-treated IGR39/IGR37; untreated cell-pair model)
- Per-cell-pair evaluation sets for baseline classification accuracy

## Outputs

- significant_peaks.csv with columns: condition, m/z, accuracy-drop
- Per-condition list of significant ions (14/30 untreated IGR39/IGR37; 3/53 treated IGR39/IGR37)
- Volcano plots with significant ions rendered as bold dots overlaid on all four comparisons

## How to apply

Load the adapted MetaPhenotype checkpoints (1- and 5-shot adapted to drug-treated IGR39/IGR37, plus the untreated cell-pair model) together with the pretreated SCMS feature matrix (noise-subtracted, background-subtracted, TIC-normalized, 10 ppm-aligned) from the songyuan93/MetaPhenotype repository. Record the baseline classification accuracy of the adapted model on its evaluation set for each cell pair. Iteratively remove a single m/z peak from the SCMS input, re-evaluate the adapted model, and flag the peak as significant (essential) when its removal lowers classification accuracy by 1% or more — the paper's significance cutoff. Tabulate the flagged ions per condition, reproducing the reported 14 and 30 significant peaks for untreated IGR39/IGR37 and 3 and 53 for treated IGR39/IGR37, then mark them as bold dots overlaid on the volcano plots of the four comparisons and write condition, m/z, and accuracy-drop values to a significant_peaks.csv table.

## Related tools

- **MetaPhenotype** (MAML-based meta-learning model whose adapted checkpoints provide the baseline and post-removal classification accuracies used to flag essential ions) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Source of the single-cell m/z feature matrix whose peaks are individually removed in the significance test)

## Examples

```
python base.py
```

## Evaluation signals

- Significant-peak counts per condition match the paper-reported values: 14 and 30 for untreated IGR39/IGR37, 3 and 53 for treated IGR39/IGR37
- Every flagged ion shows an accuracy decrease of at least 1% upon removal from the SCMS input
- significant_peaks.csv contains condition, m/z, and accuracy-drop columns for each flagged ion
- Volcano plots for all four comparisons show significant ions as bold dots while other ions remain unbolded
- Re-running the removal loop yields the same set of flagged ions, confirming determinism of the ≥1% accuracy-drop criterion

## Limitations

- The ≥1% accuracy-drop threshold is a discrete criterion, so ions with borderline effects near the cutoff may flip between significant/non-significant across re-runs or slight model-configuration changes.
- Significance is model-dependent — ions essential to the MetaPhenotype classifier may not coincide with ions deemed important by statistical tests (e.g., fold-change/p-value volcano-plot criteria).
- The reported counts apply specifically to IGR39/IGR37 melanoma cell pairs with and without drug treatment; other cell types or instruments require re-establishing the baseline and re-deriving the ion set.

## Evidence

- [readme] dataset.csv is the input for the MetaPhenotype model: [evidence span withheld — anchor status: external_doc]
