---
name: hepatocellular-carcinoma-early-recurrence-prediction
description: Use when reproducing or benchmarking HCC early-recurrence classifiers
  from aligned LC-MS feature intensities, and when testing whether aligned MS features
  outperform peptide- or protein-level classifiers.
when_to_use_negative:
- Feature lists have not yet been RT-aligned — run DeepRTAlign first to obtain information_target.csv
  with group identifiers
- The goal is identification-based (peptide/protein) biomarker analysis rather than
  evaluating ID-free aligned-feature classifiers
- The cohort is the validation set rather than the C1 training cohort (N=52) with
  early-recurrence labels
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Produces the RT-aligned LC-MS features (information_target.csv in mass_align_all_information)
    whose group-keyed intensities serve as the top-performing classifier input matrix
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: scikit-learn
  role: Trains the SVM classifiers and computes five-fold cross-validation AUC on
    each top-200 mRMR matrix
- name: pandas
  role: Loads and pivots information_target.csv into the sample-by-group intensity
    matrix used as classifier input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:be9ec2180874f4a3da5ad0feed5c58a32a6ec934f9cb932d2d316c9db3c3461f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# hepatocellular-carcinoma-early-recurrence-prediction

## Summary

Trains and compares SVM classifiers for early hepatocellular carcinoma (HCC) recurrence on the C1 training cohort (N=52), using mRMR-selected top-200 DeepRTAlign-aligned MS features versus top-200 peptide and top-200 protein matrices, scored by five-fold cross-validation AUC.

## When to use

Use when reproducing or benchmarking HCC early-recurrence classifiers from aligned LC-MS feature intensities, and when testing whether aligned MS features outperform peptide- or protein-level classifiers.

## When NOT to use

- Feature lists have not yet been RT-aligned — run DeepRTAlign first to obtain information_target.csv with group identifiers
- The goal is identification-based (peptide/protein) biomarker analysis rather than evaluating ID-free aligned-feature classifiers
- The cohort is the validation set rather than the C1 training cohort (N=52) with early-recurrence labels

## Inputs

- information_target.csv from the mass_align_all_information folder (DeepRTAlign QC-passed aligned features with sample, time, mz, charge, intensity, score, group columns)
- Peptide-level abundance matrix for the C1 training cohort
- Protein-level abundance matrix for the C1 training cohort
- Early-recurrence labels for the 52 C1 training samples

## Outputs

- Sample-by-group intensity matrix pivoted from information_target.csv (group column as aligned feature identity)
- Top-200 mRMR-selected aligned-feature, peptide, and protein matrices
- Five-fold cross-validation mean AUC per classifier
- Results comparison table written to hcc_classifier_auc_results.csv

## How to apply

Load information_target.csv from the mass_align_all_information folder produced by DeepRTAlign and pivot it into a sample-by-group intensity matrix, treating the 'group' column as the aligned feature identity (each row is a feature with columns sample, time, mz, charge, intensity, score, group). Load the peptide-level and protein-level abundance matrices plus early-recurrence labels, restricting all matrices to the C1 training cohort of 52 samples. Run minimum-redundancy maximum-relevance (mRMR) feature selection independently on each of the three matrices, retaining the top 200 analytes per level. Train scikit-learn SVM classifiers on each top-200 matrix using the early-recurrence labels and score each with five-fold cross-validation, recording mean AUC. Success is judged by reproducing the reported AUCs: 0.998 for aligned MS features, 0.931 for peptides, and 0.757 for proteins, demonstrating that aligned MS features carry more effective predictive information than peptides or proteins.

## Related tools

- **DeepRTAlign** (Produces the RT-aligned LC-MS features (information_target.csv in mass_align_all_information) whose group-keyed intensities serve as the top-performing classifier input matrix) — https://github.com/PHOENIXcenter/deeprtalign
- **scikit-learn** (Trains the SVM classifiers and computes five-fold cross-validation AUC on each top-200 mRMR matrix)
- **pandas** (Loads and pivots information_target.csv into the sample-by-group intensity matrix used as classifier input)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx  # then pivot information_target.csv on group, mRMR-select top 200, and CV an sklearn SVC
```

## Evaluation signals

- Reproduced five-fold CV AUC for the top-200 aligned-feature SVM is approximately 0.998
- Peptide-level and protein-level SVM AUCs fall near the reported 0.931 and 0.757 respectively
- Aligned-feature classifier AUC exceeds both peptide and protein classifier AUCs, confirming aligned MS features contain more effective information
- Pivoted matrix dimensions match the C1 cohort (52 samples) with group column entries as unique aligned feature IDs
- hcc_classifier_auc_results.csv is written with the three classifier-level AUC values

## Limitations

- The classifier was demonstrated only on HCC early-recurrence prediction; generalizability to other phenotypes is not established
- DeepRTAlign's default QC workflow can yield very few results when samples differ substantially or many features share nearly the same m/z; in that case set FDR to 1 and keep features with score > 0.5
- Utility of ID-free aligned features for low-abundant biomarker discovery remains an expectation, not a demonstrated result

## Evidence

- [abstract] abstract: trained classifier: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [discussion] discussion: features vs peptides/proteins: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [readme] readme: output location: "The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv"
- [readme] readme: column semantics: "intensity: the intensity corresponding to this feature.
- score: the score of aligned pairs.
- group: the group name corresponding to this feature, aligned features share the same group name."
