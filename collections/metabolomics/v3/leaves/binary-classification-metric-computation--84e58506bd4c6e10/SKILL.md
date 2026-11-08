---
name: binary-classification-metric-computation
description: Use when comparing in silico MS/MS prediction tools (e.g., SingleFrag
  vs. CFM-ID vs. 3DMolMS) on the same test compounds and you need peak-level classification
  metrics plus spectral cosine similarity on a common m/z grid.
when_to_use_negative:
- When the goal is to predict peak intensities rather than peak presence/absence —
  SingleFrag deliberately classifies peak presence per MZ position instead of regressing
  intensities across experimental conditions
- When spectra are not available on a common m/z grid or at matched collision energies
  — the comparison requires discretizing predicted and experimental spectra on the
  same 0.2 m/z bins
- When evaluating a single spectrum interactively (the spectraPrediction.ipynb notebook
  already provides a graphical predicted-vs-experimental comparison)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Deep learning tool (ANN, GNN, COM variants) whose per-MZ-position peak predictions
    are reconstructed into whole spectra and benchmarked with the classification metrics
  repo: https://github.com/MaribelPR/SingleFrag
- name: CFM-ID
  role: State-of-the-art rule-based baseline whose predicted spectra are scored with
    the same metrics on the test set
- name: 3DMolMS
  role: Deep learning baseline predicting MS/MS from 3D conformations, evaluated with
    the same metrics on the identical test compounds
  repo: josiehong/3DMolMS
- name: mol2vec
  role: Generates the 300-dimensional SMILES embeddings that are the ANN model input;
    indirectly determines the SingleFrag predictions being scored
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:1070970d50469f232c0daecd387c6d63ba8e1b496bb93f94b84ff9ef605a4de1
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - CFM-ID
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# binary-classification-metric-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmark whole predicted MS/MS spectra against experimental ones by treating each m/z bin as a binary peak-present/peak-absent classification and computing per-tool, per-molecule precision, recall, accuracy, F1, and cosine similarity.

## When to use

Use when comparing in silico MS/MS prediction tools (e.g., SingleFrag vs. CFM-ID vs. 3DMolMS) on the same test compounds and you need peak-level classification metrics plus spectral cosine similarity on a common m/z grid.

## When NOT to use

- When the goal is to predict peak intensities rather than peak presence/absence — SingleFrag deliberately classifies peak presence per MZ position instead of regressing intensities across experimental conditions
- When spectra are not available on a common m/z grid or at matched collision energies — the comparison requires discretizing predicted and experimental spectra on the same 0.2 m/z bins
- When evaluating a single spectrum interactively (the spectraPrediction.ipynb notebook already provides a graphical predicted-vs-experimental comparison)

## Inputs

- Experimental MS/MS spectra for the 2,929 test-compound SMILES from the SingleFrag Figshare dataset (inputMols file)
- Reconstructed SingleFrag predicted spectra (results/predSpectra.json) built from per-MZ-position prediction files (results/dfPredictionPeak_*.pickle)
- Per-position classification threshold files (thresholdsANN.pickle, thresholdsGNN.pickle, thresholdsCOM.pickle)
- Predicted in silico spectra from CFM-ID 4.0 and 3DMolMS on the same test compounds

## Outputs

- Per-tool per-molecule table of precision, recall, accuracy, F1, and cosine similarity (fig3g-i_tool_metrics.csv) supporting the Fig. 3g-i benchmark

## How to apply

Discretize both predicted and experimental spectra onto a common 0.2 m/z binning grid over the 1,000 most frequented MZ positions of the training set, so each bin becomes a binary presence/absence call. For SingleFrag variants (ANN, GNN, COM), reconstruct full spectra from per-MZ-position prediction files (results/dfPredictionPeak_*.pickle, results/predSpectra.json), applying each model's per-position classification thresholds (thresholdsANN/GNN/COM.pickle) to call peak presence. Run the comparison tools (CFM-ID 4.0, 3DMolMS) on the same 2,929 test-compound SMILES to obtain their predicted spectra, binned identically. For each tool and each molecule, compute precision, recall, accuracy, F1, and cosine similarity of the predicted spectrum against the experimental spectrum. Assemble the metrics into a per-tool per-molecule table (fig3g-i_tool_metrics.csv) to support benchmark comparison.

## Related tools

- **SingleFrag** (Deep learning tool (ANN, GNN, COM variants) whose per-MZ-position peak predictions are reconstructed into whole spectra and benchmarked with the classification metrics) — https://github.com/MaribelPR/SingleFrag
- **CFM-ID** (State-of-the-art rule-based baseline whose predicted spectra are scored with the same metrics on the test set)
- **3DMolMS** (Deep learning baseline predicting MS/MS from 3D conformations, evaluated with the same metrics on the identical test compounds) — josiehong/3DMolMS
- **mol2vec** (Generates the 300-dimensional SMILES embeddings that are the ANN model input; indirectly determines the SingleFrag predictions being scored)

## Examples

```
molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- All metrics are computed after discretizing predicted and experimental spectra on the same 0.2 m/z grid — verify no tool was binned differently
- Expected outcome on the test set: all three SingleFrag variants (ANN, GNN, COM) achieve higher precision, accuracy, and cosine similarity than CFM-ID and 3DMolMS, with recall comparable to 3DMolMS (slightly better for ANN and combined models)
- The output table (fig3g-i_tool_metrics.csv) must contain one row per tool per molecule covering all five metrics
- Threshold sanity check: a peak is called in an MZ bin only if the model prediction is greater than or equal to that position's threshold from thresholdsANN/GNN/COM.pickle

## Limitations

- GNN model shows slightly lower performance than ANN, attributed to unoptimized graph features and layer architectures — the authors deliberately kept neural network architectures simple
- Metric conclusions are specific to spectra acquired at multiple collision energies from curated libraries; other libraries (GNPS, MSDial, MassBank) often lack sufficient curation and contain spectra at only one or a few collision energies per compound
- Binary presence/absence metrics do not capture peak intensity differences across experimental conditions

## Evidence

- [results] Compared on the test set at 0.2 m/z binning, all three SingleFrag variants achieve higher precision, accuracy, and cosine similarity than CFM-ID and 3DMolMS, with recall comparable to 3DMolMS: [evidence span withheld — anchor status: not_found]
