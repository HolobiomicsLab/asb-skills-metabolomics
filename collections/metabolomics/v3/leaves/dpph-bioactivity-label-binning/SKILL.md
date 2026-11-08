---
name: dpph-bioactivity-label-binning
description: Use when raw DPPH radical-scavenging measurements for wild soybean accessions
  must be converted into binary class labels for supervised bioactivity prediction,
  and the article's 0.4/0.6 cutoffs must be reproduced.
when_to_use_negative:
- The bioactivity measurements are already binarized or supplied as a categorical
  phenotype column, so thresholding is unnecessary.
- The task is regression (e.g. predicting the continuous DPPH value) rather than the
  paper's binary low/high classification.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: scikit-learn
  role: Train the Random Forest classifier on the binarized labels (5-fold CV, accuracy/precision/recall/F1/AUC
    evaluation).
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: pandas
  role: Load the DPPH value table and apply the 0.4/0.6 thresholding to build the
    label vector.
- name: MS2toImg-CNN
  role: Use the binarized labels as the CNN's training and evaluation targets for
    the molecular fingerprint images.
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:a4499f194f9c22d53072059c7075f6a20e32d54b44c661caf8fbf9760e0f36f1
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - MS-DIAL
  - MS2toImg-CNN
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# dpph-bioactivity-label-binning

## Summary

Binning wild soybean DPPH radical-scavenging values into low/high activity classes by thresholding normalized bioactivity values at 0.4 and 0.6, creating a binary label vector that feeds the RF and CNN bioactivity classifiers.

## When to use

Use when raw DPPH radical-scavenging measurements for wild soybean accessions must be converted into binary class labels for supervised bioactivity prediction, and the article's 0.4/0.6 cutoffs must be reproduced.

## When NOT to use

- The bioactivity measurements are already binarized or supplied as a categorical phenotype column, so thresholding is unnecessary.
- The task is regression (e.g. predicting the continuous DPPH value) rather than the paper's binary low/high classification.

## Inputs

- Normalized DPPH radical-scavenging bioactivity values (float, per accession, 545 wild soybean accessions)
- Sample ID mapping file aligning DPPH values with the LC-MS/MS feature-table row order

## Outputs

- Binary label vector (low-activity / high-activity) of length equal to retained accessions
- Label-distribution summary (class counts and excluded mid-band samples)

## How to apply

1) Load normalized DPPH radical-scavenging values for each of the 545 wild soybean (Glycine soja Sieb. & Zucc.) accessions. 2) Discard samples whose normalized values fall in the 0.4–0.6 'no-man's-land' band, since the paper assigns only values <0.4 or >0.6. 3) Label samples with normalized value <0.4 as the low-activity class and >0.6 as the high-activity class, producing a binary vector aligned with the LC-MS/MS feature table order. 4) The resulting labels are used for 5-fold cross-validated Random Forest training (scikit-learn 1.3.0) on cosine-similarity and spectral-entropy feature tables, and as CNN training targets for the MS2toImg fingerprint images. 5) Verify that the label distribution roughly matches the paper's reported class counts and that the RF ROC AUC reproduces ~0.817 (Figure 4C) as a sanity check on the binning.

## Related tools

- **scikit-learn** (Train the Random Forest classifier on the binarized labels (5-fold CV, accuracy/precision/recall/F1/AUC evaluation).) — https://github.com/jsehhs/MS2toImg-CNN
- **pandas** (Load the DPPH value table and apply the 0.4/0.6 thresholding to build the label vector.)
- **MS2toImg-CNN** (Use the binarized labels as the CNN's training and evaluation targets for the molecular fingerprint images.) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
df['label'] = np.where(df['dpph_norm'] < 0.4, 'low', np.where(df['dpph_norm'] > 0.6, 'high', 'exclude'))
```

## Evaluation signals

- No sample is labeled in the 0.4–0.6 band; only values <0.4 and >0.6 receive labels.
- Number of retained accessions plus excluded mid-band samples equals the original 545 accessions.
- Random Forest 5-fold CV ROC AUC on the resulting labels reproduces the reported ~0.817 (Figure 4C).
- RF accuracy, precision, recall, and F1 values match the paper's reported baseline metrics within reasonable tolerance.

## Limitations

- Binning is only defined for the wild soybean DPPH assay; other assays (e.g., elastase inhibition, TFC, TPC) may require different thresholds or formats.
- Values exactly at 0.4 or 0.6 fall in an undefined zone; the paper's wording does not specify how ties are handled.
- The binning is sensitive to the normalization scheme used upstream (TIC-normalized LC-MS/MS intensities); changing the normalization may require re-deriving the 0.4/0.6 cutoffs.

## Evidence
