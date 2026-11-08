---
name: deep-neural-network-retention-time-matching
description: Use when labeled feature pairs (m/z, charge, RT, intensity) from aligned
  LC-MS runs are available and you must decide whether a deep learning direct-matching
  model outperforms classical ML classifiers (RF, KNN, SVM, LR) on RT alignment quality.
when_to_use_negative:
- Input is already a peptide/protein identification table where match-between-runs
  (MaxQuant, PANDA, MSFragger, DIA-NN MBR-style) alignment is sufficient
- The analysis goal is identification-based RT alignment for a small single-pair dataset
  rather than large-cohort feature correspondence with labeled pairs
- No labeled true/false feature pairs exist, since the DNN requires supervised training
  data
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3172
tools:
- name: DeepRTAlign
  role: Produces the aligned feature-feature pairs (m/z, charge, RT, intensity with
    score and group) used as labeled training data for the DNN direct-matching classifier
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: PyTorch
  role: Framework used to construct and train the deep neural network with 3 hidden
    layers of 5000 neurons each
  repo: https://pytorch.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:c4aa057f3eb03d66f187a22009ecd90663a238b31df803241ddb22012b1bfec6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# deep-neural-network-retention-time-matching

## Summary

Trains a PyTorch deep neural network (3 hidden layers of 5000 neurons each) on labeled feature-feature pairs produced by DeepRTAlign RT alignment to discriminate true from false cross-run feature matches, and benchmarks it against RF, KNN, SVM, and LR classifiers by ROC AUC.

## When to use

Use when labeled feature pairs (m/z, charge, RT, intensity) from aligned LC-MS runs are available and you must decide whether a deep learning direct-matching model outperforms classical ML classifiers (RF, KNN, SVM, LR) on RT alignment quality.

## When NOT to use

- Input is already a peptide/protein identification table where match-between-runs (MaxQuant, PANDA, MSFragger, DIA-NN MBR-style) alignment is sufficient
- The analysis goal is identification-based RT alignment for a small single-pair dataset rather than large-cohort feature correspondence with labeled pairs
- No labeled true/false feature pairs exist, since the DNN requires supervised training data

## Inputs

- Labeled feature-feature pairs from DeepRTAlign-aligned LC-MS runs (m/z, charge, RT, intensity attributes; true/false match labels)
- DeepRTAlign alignment output (e.g., information_target.csv from the mass_align_all_information folder, containing sample, time, mz, charge, intensity, score, group columns)
- PyTorch (CPU version) environment

## Outputs

- Trained DNN direct-matching classifier (3 hidden layers x 5000 neurons) plus trained RF, KNN, SVM, and LR reference classifiers
- Per-model held-out test AUC values compiled into a ranked comparison table (reproducing Supplementary Table 3)

## How to apply

Collect labeled feature-feature pairs from the DeepRTAlign-aligned LC-MS cohort (e.g., the 400,000 pairs from the HCC-T training set), carrying m/z, charge, RT, and intensity attributes, and split them into training and held-out test sets. Construct the deep neural network in PyTorch with 3 hidden layers of 5000 neurons each and train it on the feature-pair training set. Train random forest, k-nearest neighbors, SVM, and logistic regression on the identical training split so the comparison is fair. Evaluate all five models on the held-out test set, compute AUC for each, rank them, and confirm the DNN attains the highest test AUC (the ranking reported in Supplementary Table 3). Rationale: DeepRTAlign pairs a coarse alignment (pseudo warping function) with a deep learning-based direct matching model so that both monotonic and non-monotonic RT shifts can be handled, and its score is used downstream for quality control.

## Related tools

- **DeepRTAlign** (Produces the aligned feature-feature pairs (m/z, charge, RT, intensity with score and group) used as labeled training data for the DNN direct-matching classifier) — https://github.com/PHOENIXcenter/deeprtalign
- **PyTorch** (Framework used to construct and train the deep neural network with 3 hidden layers of 5000 neurons each) — https://pytorch.org/

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx
```

## Evaluation signals

- After 10-fold cross-validation on the HCC-T training set, the DNN must attain the highest AUC among DNN, RF, KNN, SVM, and LR on independent test sets, matching the ranking in Supplementary Table 3
- All five classifiers must be trained on the identical training split of the ~400,000 feature-pair dataset for a fair comparison
- Input feature pairs must carry m/z, charge, RT, and intensity attributes consistent with the DeepRTAlign information_target.csv schema
- QC of the underlying alignment should use DeepRTAlign's score-based default workflow (fdr 0.01), or score > 0.5 if FDR is set to 1 for dissimilar samples

## Limitations

- Default QC workflow can report very few results when samples are quite different or many features share almost the same m/z; set FDR to 1 and select score > 0.5 in that case
- Alignment has limited influence on quantification in theory; improving quantification accuracy by jointly optimizing feature extraction and alignment remains future work
- Generalizability boundary was determined but limits of applicability remain; utility for low-abundant biomarker discovery is expected, not demonstrated

## Evidence

- [intro] deep learning-based direct matching: "Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts"
- [readme] PyTorch dependency: "Install Pytorch CPU version, please refer to https://pytorch.org/."
- [abstract] DeepRTAlign purpose: "Here, we develop a deep learning-based RT alignment tool, DeepRTAlign, for large cohort LC-MS data analysis."
- [readme] QC via score: "In the default workflow, score will be used for quality control (default). This workflow works well in most cases. But when the samples are quite different"
