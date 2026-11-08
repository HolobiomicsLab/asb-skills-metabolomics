---
name: scms-peak-ablation-significance-testing
description: Use when you have a trained meta-learning classifier (MetaPhenotype)
  on pretreated SCMS feature tables and need to select the small number of m/z species
  critical for phenotype classification, e.g. to overlay as bold dots on volcano plots.
when_to_use_negative:
- You only need unsupervised exploration of metabolomic differences — the article
  notes PCA cannot effectively discriminate highly similar cells like IGR39/IGR37,
  but peak ablation requires a trained supervised classifier and labeled evaluation
  data.
- Input is raw, unaligned SCMS spectra — ablation must be run on the pretreated, TIC-normalized,
  Geena2-aligned peak table, not on exported raw peaks.
- You have no trained MetaPhenotype checkpoint for the target condition — significance
  is defined relative to that specific adapted model's classification accuracy, not
  a generic feature-importance metric.
tools:
- name: MetaPhenotype
  role: Classifier whose accuracy is re-evaluated after each single-peak removal;
    defines the ≥1% significance criterion
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Experimental technique that produced the single-cell m/z peak matrices being
    ablated
- name: Geena2
  role: Online bioinformatics tool for peak alignment (10 ppm tolerance) that generates
    the aligned feature table used in ablation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:c4d7061dc2315c6c1cc9cf46f22ac3fd0a90fad3afda35edfad1c9ca4d55976d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - GitHub (MetaPhenotype repository)
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# scms-peak-ablation-significance-testing

## Summary

Identifies m/z species essential for cell-phenotype discrimination in single-cell mass spectrometry (SCMS) data by iteratively removing one peak at a time and flagging peaks whose removal drops MetaPhenotype classification accuracy by ≥1%.

## When to use

Use when you have a trained meta-learning classifier (MetaPhenotype) on pretreated SCMS feature tables and need to select the small number of m/z species critical for phenotype classification, e.g. to overlay as bold dots on volcano plots.

## When NOT to use

- You only need unsupervised exploration of metabolomic differences — the article notes PCA cannot effectively discriminate highly similar cells like IGR39/IGR37, but peak ablation requires a trained supervised classifier and labeled evaluation data.
- Input is raw, unaligned SCMS spectra — ablation must be run on the pretreated, TIC-normalized, Geena2-aligned peak table, not on exported raw peaks.
- You have no trained MetaPhenotype checkpoint for the target condition — significance is defined relative to that specific adapted model's classification accuracy, not a generic feature-importance metric.

## Inputs

- Pretreated SCMS feature matrix (dataset.csv in the MetaPhenotype Input data folder): m/z peaks per single cell, after noise subtraction, background subtraction, TIC normalization, peak alignment (Geena2, 10 ppm), and missing-value removal
- Trained/adapted MetaPhenotype model checkpoint (base.py / meta.py) for the cell pair of interest (1-shot or 5-shot adapted to drug-treated IGR39/IGR37, or the untreated cell-pair model)
- Cell-phenotype labels (primary vs metastatic) for the evaluation set of each cell pair

## Outputs

- Per-condition list of significant m/z peaks whose ablation reduces classification accuracy by ≥1%
- Accuracy-drop values for each significant peak (baseline accuracy minus post-removal accuracy)
- significant_peaks.csv (condition, m/z, accuracy drop) and highlighted species for volcano-plot overlay as bold dots

## How to apply

Load the adapted MetaPhenotype checkpoints (1- and 5-shot adapted models for drug-treated IGR39/IGR37 plus the untreated cell-pair model) and the pretreated SCMS feature matrix exported via the Geena2 alignment workflow (mass tolerance 10 ppm, noise/background subtraction, TIC normalization). For each cell pair, record the baseline classification accuracy of the adapted model on its evaluation set. Then perform leave-one-peak-out ablation: iteratively remove a single m/z peak from the SCMS input, re-run the model (e.g. via meta.py with the modified dataset.csv), and flag the peak as significant when removal decreases classification accuracy by at least 1%. Tabulate per condition the significant m/z and their accuracy drops, reproducing 14 and 30 significant peaks for untreated IGR39/IGR37 and 3 and 53 for treated IGR39/IGR37, and mark these ions for overlay as bold dots on the corresponding volcano plots.

## Related tools

- **MetaPhenotype** (Classifier whose accuracy is re-evaluated after each single-peak removal; defines the ≥1% significance criterion) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Experimental technique that produced the single-cell m/z peak matrices being ablated)
- **Geena2** (Online bioinformatics tool for peak alignment (10 ppm tolerance) that generates the aligned feature table used in ablation)

## Examples

```
python base.py  # re-run MetaPhenotype on dataset.csv with one m/z peak removed; flag peak as significant if classification accuracy drops by ≥1%
```

## Evaluation signals

- Significant-peak counts per condition match the paper-reported values: 14 and 30 for untreated IGR39/IGR37, 3 and 53 for drug-treated IGR39/IGR37
- Every flagged ion shows an accuracy decrease of at least 1% when removed from the SCMS input
- Baseline classification accuracy of the adapted model is recorded before any peak removal, so each drop is computed against the same reference
- Output table (significant_peaks.csv) contains condition, m/z, and accuracy-drop columns and covers all four comparisons
- Marked significant ions appear as bold dots on the volcano plots for the untreated and treated IGR39/IGR37 comparisons

## Limitations

- Significance is model- and condition-specific: a peak is only 'essential' relative to the particular adapted MetaPhenotype checkpoint and evaluation set used, so results do not transfer across models without re-testing.
- MAML-based models are memory- and computationally intensive and sensitive to hyperparameters, which constrains how many ablation rounds can be practically re-run.
- The criterion is tied to the ≥1% accuracy-drop threshold on this melanoma cell-pair task; other datasets may need recalibration of the threshold.
- Detected species were only tentatively labeled using METLIN and HMDB databases, so significant m/z values should be reported with putative annotations.

## Evidence
