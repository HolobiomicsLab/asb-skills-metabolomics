---
name: mrmr-feature-selection
description: Use when building predictive classifiers from high-dimensional LC-MS
  matrices (hundreds to thousands of aligned features, peptides, or proteins) and
  you need to reduce dimensionality to a fixed-size top-N feature set (e.g., top 200)
  prior to SVM training and cross-validated AUC evaluation.
when_to_use_negative:
- Input is already a small, low-dimensional feature table where dimensionality reduction
  is unnecessary
- The aim is retention time alignment itself rather than downstream feature selection
  — use DeepRTAlign's alignment pipeline (steps 1–6) first, then apply mRMR to its
  information_target.csv output
- The outcome is continuous rather than a binary class label, unless the mRMR implementation
  supports regression targets
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Upstream tool that produces the aligned MS feature matrix (information_target.csv,
    group column = aligned feature identity) on which mRMR selection is run
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: scikit-learn
  role: Provides the SVM classifiers trained on each mRMR top-200 matrix and the five-fold
    cross-validation AUC scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:91694d7e7cd24605138bfb27e13220048d9b87341401df2a10d7573f9d6be05d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mrmr-feature-selection

## Summary

Apply minimum-redundancy maximum-relevance (mRMR) feature selection to pick the top 200 analytes (aligned MS features, peptides, or proteins) before training SVM classifiers for HCC early-recurrence prediction, enabling fair comparison of classifier performance across feature levels.

## When to use

Use when building predictive classifiers from high-dimensional LC-MS matrices (hundreds to thousands of aligned features, peptides, or proteins) and you need to reduce dimensionality to a fixed-size top-N feature set (e.g., top 200) prior to SVM training and cross-validated AUC evaluation.

## When NOT to use

- Input is already a small, low-dimensional feature table where dimensionality reduction is unnecessary
- The aim is retention time alignment itself rather than downstream feature selection — use DeepRTAlign's alignment pipeline (steps 1–6) first, then apply mRMR to its information_target.csv output
- The outcome is continuous rather than a binary class label, unless the mRMR implementation supports regression targets

## Inputs

- information_target.csv from the mass_align_all_information folder (DeepRTAlign aligned-feature output: sample, time, mz, charge, intensity, group columns)
- Peptide-level abundance matrix restricted to the training cohort
- Protein-level abundance matrix restricted to the training cohort
- Early-recurrence class labels for the C1 training set (N=52 samples)

## Outputs

- Top-200 mRMR-selected feature/peptide/protein submatrices
- Five-fold cross-validation mean AUC per classifier (expected: 0.998 features, 0.931 peptides, 0.757 proteins)
- Results table hcc_classifier_auc_results.csv

## How to apply

Restrict all abundance matrices to the training cohort (e.g., C1, N=52 HCC early-recurrence samples) with matched class labels. Run mRMR independently on each analyte level — the sample-by-group intensity matrix pivoted from information_target.csv (group column = aligned feature identity from DeepRTAlign output), the peptide-level matrix, and the protein-level matrix — retaining the top 200 analytes in each. Train scikit-learn SVM classifiers on each top-200 matrix using the early-recurrence labels, then score with five-fold cross-validation and record mean AUC. Rationale: mRMR balances relevance to the recurrence label against redundancy among features, which matters because aligned MS features are highly correlated; in this study the top-200 aligned-feature SVM achieved AUC 0.998 versus 0.931 for peptides and 0.757 for proteins. Write the reproduced AUCs to a results table and compare against reported values as validation.

## Related tools

- **DeepRTAlign** (Upstream tool that produces the aligned MS feature matrix (information_target.csv, group column = aligned feature identity) on which mRMR selection is run) — https://github.com/PHOENIXcenter/deeprtalign
- **scikit-learn** (Provides the SVM classifiers trained on each mRMR top-200 matrix and the five-fold cross-validation AUC scoring)

## Examples

```
import pandas as pd; df = pd.read_csv('mass_align_all_information/information_target.csv'); X = df.pivot_table(index='sample', columns='group', values='intensity', aggfunc='max')
```

## Evaluation signals

- Reproduced five-fold CV AUCs match reported values: 0.998 (top-200 aligned MS features), 0.931 (top-200 peptides), 0.757 (top-200 proteins)
- Each mRMR-selected matrix contains exactly 200 analytes (features, peptides, or proteins)
- All three matrices are restricted to the same C1 cohort of 52 samples with consistent early-recurrence labels
- The aligned-feature matrix is a valid sample-by-group pivot of information_target.csv, with the group column serving as the aligned feature identity
- Results table hcc_classifier_auc_results.csv is written and ranks aligned MS features highest, consistent with the finding that aligned MS features carry more effective information than peptides and proteins

## Limitations

- mRMR top-200 selection is applied per analyte level; the comparison depends on DeepRTAlign alignment quality — default QC (FDR 0.01, score-based) can yield very few results when samples are quite different, in which case setting FDR to 1 and filtering score > 0.5 is recommended
- Aligned-feature groups may contain multiple features per sample; default keep_best_feature=1 and keep_best_group=1 settings affect the intensity matrix that feeds mRMR
- The reported AUCs are specific to the N=52 C1 training cohort and five-fold CV; the article notes generalizability boundaries were determined but limits of applicability remain

## Evidence

- [abstract] using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma.: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [discussion] The results showed that aligned MS features have more effective information compared with peptides and proteins.: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [results] In five-fold cross-validation on training set C1, the SVM classifier built on the top 200 mRMR-selected aligned MS features achieved the highest AUC (0.998): [evidence span withheld — anchor status: not_found]
- [other] Load information_target.csv from the mass_align_all_information folder with pandas and pivot it into a sample-by-group intensity matrix, treating the group column as the aligned feature identity.: [evidence span withheld — anchor status: not_found]
- [readme] The results will output to the mass_align_all_information folder.: "The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv"
