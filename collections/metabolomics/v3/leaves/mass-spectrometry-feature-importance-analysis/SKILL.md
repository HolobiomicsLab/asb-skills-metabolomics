---
name: mass-spectrometry-feature-importance-analysis
description: Use when a pretreated SCMS feature matrix (m/z peaks per single cell)
  and a trained classifier exist and you need to rank essential metabolite peaks for
  discriminating primary vs metastatic melanoma cell phenotypes.
when_to_use_negative:
- Input is not a pretreated SCMS feature matrix but raw mass-spectrometry spectra
  (noise/background subtraction, TIC normalization, and peak alignment at 10 ppm must
  be done first).
- No trained classifier is available to score accuracy changes, e.g., only unsupervised
  profiles such as PCA of IGR39/IGR37 cells exist — PCA cannot effectively discriminate
  these phenotypically similar cells, so the removal test cannot be applied.
- The model is a conventional transfer-learning or deep-learning model without a reusable
  adapted checkpoint; the significance criterion is defined specifically against MetaPhenotype's
  classification accuracy on limited (1/5-shot) adaptation data.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaPhenotype
  role: Adapted MAML-based classifier whose accuracy change upon peak removal defines
    the significance criterion
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Acquisition technique generating the single-cell metabolomic profiles that
    supply the m/z feature matrix
- name: Geena2
  role: Peak alignment of exported SCMS peaks with a mass tolerance of 10 ppm prior
    to significance testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:703505a486d50e37190dd32564ee6eebfee3be5b5fb4798d91eb8420ecddcae1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - GitHub (MetaPhenotype repository)
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mass-spectrometry-feature-importance-analysis

## Summary

Identify significant m/z features in single-cell mass spectrometry (SCMS) data by iteratively removing each peak and re-evaluating an adapted MetaPhenotype model, flagging ions whose removal drops cell-phenotype classification accuracy by at least 1%.

## When to use

Use when a pretreated SCMS feature matrix (m/z peaks per single cell) and a trained classifier exist and you need to rank essential metabolite peaks for discriminating primary vs metastatic melanoma cell phenotypes.

## When NOT to use

- Input is not a pretreated SCMS feature matrix but raw mass-spectrometry spectra (noise/background subtraction, TIC normalization, and peak alignment at 10 ppm must be done first).
- No trained classifier is available to score accuracy changes, e.g., only unsupervised profiles such as PCA of IGR39/IGR37 cells exist — PCA cannot effectively discriminate these phenotypically similar cells, so the removal test cannot be applied.
- The model is a conventional transfer-learning or deep-learning model without a reusable adapted checkpoint; the significance criterion is defined specifically against MetaPhenotype's classification accuracy on limited (1/5-shot) adaptation data.

## Inputs

- Pretreated single-cell mass spectrometry (SCMS) feature matrix of m/z peaks per cell (e.g., dataset.csv from the MetaPhenotype repo)
- Adapted MetaPhenotype model checkpoints (1-shot, 5-shot, or untreated cell-pair model)
- Baseline cell-phenotype classification accuracy on the evaluation set (from AUROC/F1/accuracy evaluation)

## Outputs

- Table of significant m/z peaks per condition with accuracy-drop values (significant_peaks.csv: condition, m/z, accuracy decrease)
- Counts of significant ions per comparison (14/30 untreated IGR39/IGR37; 3/53 treated)
- Significant-ion annotations for volcano-plot overlay (bold dots)

## How to apply

Load the adapted MetaPhenotype model (1- or 5-shot adapted to drug-treated IGR39/IGR37, or the untreated cell-pair model) from the github.com/songyuan93/MetaPhenotype repository together with the pretreated SCMS feature matrix produced by noise subtraction, background subtraction, TIC normalization, and peak alignment (10 ppm tolerance, Geena2). Record the baseline classification accuracy of the adapted model on its evaluation set, then iteratively remove one m/z peak at a time from the SCMS input and re-evaluate the model. Flag a peak as significant (essential) when its removal decreases classification accuracy by ≥1%; for untreated IGR39/IGR37 this yields 14 and 30 significant ions, and for treated cells 3 and 53. Tabulate condition, m/z, and accuracy-drop values (e.g., significant_peaks.csv) and mark significant ions for overlay as bold dots on volcano plots of each comparison. The rationale is that a peak is biologically meaningful only if the meta-learning model's phenotype prediction degrades without it, filtering the large SCMS peak list down to a small set of essential species.

## Related tools

- **MetaPhenotype** (Adapted MAML-based classifier whose accuracy change upon peak removal defines the significance criterion) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Acquisition technique generating the single-cell metabolomic profiles that supply the m/z feature matrix)
- **Geena2** (Peak alignment of exported SCMS peaks with a mass tolerance of 10 ppm prior to significance testing)

## Examples

```
python base.py  # dataset.csv loaded via dataloade.py; after adaptation, remove each m/z peak in turn, re-run, keep peaks whose removal drops accuracy by >=1%
```

## Evaluation signals

- The significant-peak counts per condition match the paper-reported values: 14 and 30 for untreated IGR39/IGR37, and 3 and 53 for treated IGR39/IGR37.
- Every flagged ion shows an accuracy decrease of at least 1% when removed from the SCMS input, verified by re-running the adapted MetaPhenotype model.
- Baseline accuracy and every leave-one-peak-out accuracy are computed after the same 50-epoch evaluation protocol (AUROC, F-1 score, and prediction accuracy).
- significant_peaks.csv contains one row per significant ion with condition, m/z, and accuracy-drop values and no missing-value entries left from the pretreatment step.
- Significant ions are rendered as bold dots on the volcano plots of the four comparisons without altering p-value or fold-change calculations.

## Limitations

- The ≥1% accuracy-drop threshold is an empirical cutoff; peaks whose removal degrades accuracy by less than 1% are discarded even if statistically enriched in volcano plots.
- MetaPhenotype's MAML base is memory- and computationally intensive (multiple inner-loop gradient steps with backpropagation through updates) and sensitive to hyperparameters, which affects how cheaply the leave-one-peak-out loop can be repeated.
- Significance depends on the adapted model's small-sample evaluation; on the ambient SCMS setting with relatively small numbers of cells, the accuracy estimates that drive the criterion may be noisy.

## Evidence

- [results] Significance criterion is the accuracy drop threshold: [evidence span withheld — anchor status: not_found]
- [readme] Model entry point in the repository: [evidence span withheld — anchor status: external_doc]
- [readme] Input data file expected by the repo: [evidence span withheld — anchor status: external_doc]
