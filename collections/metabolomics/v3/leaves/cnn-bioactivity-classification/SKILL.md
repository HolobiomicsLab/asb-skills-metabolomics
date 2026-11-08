---
name: cnn-bioactivity-classification
description: Use when you have raw LC-MS/MS data for many biological samples (e.g.,
  545 wild soybean accessions) plus a binary or binned bioactivity endpoint (DPPH,
  elastase, TFC, TPC) and want end-to-end activity prediction without compound annotation.
when_to_use_negative:
- Input is already an aligned feature table or you have structurally annotated metabolites
  with per-compound features — this skill is specifically for raw, unaligned LC-MS/MS
  fragmentation data
- You need feature-level interpretation (which metabolite drives activity); the image-based
  fingerprint approach bypasses compound identification entirely
- Data comes from a non-MS platform or lacks MS/MS fragmentation information (no precursor/fragment
  m/z pairs)
edam_operation: http://edamontology.org/operation_3762
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL
  role: Preprocesses raw MS/MS data to extract precursor (MS1) and fragment (MS2)
    m/z pairs and their intensities before image rendering
- name: matplotlib
  role: Renders MS1/MS2 scatter-plot grayscale molecular fingerprint images at 600
    dpi
- name: TensorFlow
  role: Image resize to target resolutions and CNN training backend (Keras v2.14)
- name: Keras 2.14
  role: Defines and trains the CNN architecture with binary cross-entropy and Adam
    optimizer
- name: scikit-learn
  role: Fivefold StratifiedKFold cross-validation and binarization of bioactivity
    labels
- name: MS2toImg-CNN
  role: Official implementation of the end-to-end framework
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:1535bc999e45cedfd0d96ea16719fcd3979271af1554d046d73fe54e55c48004
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - Keras
  - Keras 2.14
  - MS-DIAL
  - MS2toImg
  - MS2toImg-CNN
  - Python
  - TensorFlow
  - matplotlib
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3762
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# cnn-bioactivity-classification

## Summary

Trains a convolutional neural network on 2D grayscale molecular fingerprint images rendered from raw LC-MS/MS data (MS1 m/z on x-axis, MS2 m/z on y-axis, intensities encoded as size/grayscale) to predict bioactivity of natural product extracts without prior compound identification or feature alignment.

## When to use

Use when you have raw LC-MS/MS data for many biological samples (e.g., 545 wild soybean accessions) plus a binary or binned bioactivity endpoint (DPPH, elastase, TFC, TPC) and want end-to-end activity prediction without compound annotation.

## When NOT to use

- Input is already an aligned feature table or you have structurally annotated metabolites with per-compound features — this skill is specifically for raw, unaligned LC-MS/MS fragmentation data
- You need feature-level interpretation (which metabolite drives activity); the image-based fingerprint approach bypasses compound identification entirely
- Data comes from a non-MS platform or lacks MS/MS fragmentation information (no precursor/fragment m/z pairs)

## Inputs

- Raw LC-MS/MS data (MS1 precursor and MS2/MS-MS fragment m/z with intensities) from wild soybean (Glycine soja) extracts, preprocessed via MS-DIAL
- Bioactivity endpoint labels per sample (e.g., DPPH radical scavenging, elastase inhibition, total flavonoid content, total phenolic content), normalized to 0–1 and binarized

## Outputs

- 2D grayscale molecular fingerprint images (75×70, 150×140, or 300×280 pixels) per sample
- Trained CNN classification model per bioassay
- Per-assay classification accuracy and standard deviation (e.g., AUC 0.96 for DPPH) compiled into a four-assay accuracy summary table

## How to apply

Preprocess raw data with MS-DIAL to extract MS1/MS2 m/z–intensity pairs, select adduct ions ([M-H]⁻, [M-H₂O-H]⁻, [M+Na-2H]⁻, [M-K-2H]⁻, [M-FA-H]⁻), normalize intensities via TIC, merge features when the average of MS1, RT and MS/MS similarity scores exceeds 0.8, and filter peaks with intensity below 10⁻⁴. Render each sample as a grayscale scatter-plot image (matplotlib, 600 dpi) with precursor m/z on x-axis and product m/z on y-axis, precursor intensity as point size, product intensity as grayscale, masking pixels where MS2 > MS1 to zero, then resize to 75×70, 150×140, or 300×280 using TensorFlow image resize. Binarize the bioactivity endpoint (e.g., DPPH normalized values <0.4 = low-activity, >0.6 = high-activity) and train the CNN — two convolutional layers with 48 3×3 filters plus max pooling, five dense layers each followed by dropout, sigmoid output with binary cross-entropy loss, Adam optimizer — under fivefold StratifiedKFold cross-validation. Reapply the identical architecture to other endpoints (elastase inhibition, TFC, TPC) changing only the labels, then evaluate on independent test sets and compile accuracy tables.

## Related tools

- **MS-DIAL** (Preprocesses raw MS/MS data to extract precursor (MS1) and fragment (MS2) m/z pairs and their intensities before image rendering)
- **matplotlib** (Renders MS1/MS2 scatter-plot grayscale molecular fingerprint images at 600 dpi)
- **TensorFlow** (Image resize to target resolutions and CNN training backend (Keras v2.14))
- **Keras 2.14** (Defines and trains the CNN architecture with binary cross-entropy and Adam optimizer)
- **scikit-learn** (Fivefold StratifiedKFold cross-validation and binarization of bioactivity labels)
- **MS2toImg-CNN** (Official implementation of the end-to-end framework) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 150 140
```

## Evaluation signals

- Confusion matrices from 5-fold cross-validation should show high diagonal values, low standard deviations, and minimal false positives/negatives
- AUC of 0.96 expected for the DPPH radical scavenging high vs. low discrimination task
- Independent test-set accuracies for elastase inhibition, TFC, and TPC should be high and comparable to DPPH performance (generalizability check)
- Image dimensions must exactly match one of 75×70, 150×140, or 300×280 pixels; 150×140 was the optimized resolution
- All pixels above the MS1 m/z diagonal (MS2 > MS1) must be zero (black) in the rendered images

## Limitations

- Binarization of activity (e.g., DPPH <0.4 vs >0.6) discards intermediate values, reducing usable sample count
- Image resolution trades predictive accuracy against computational cost; 300×280 can be expensive
- Framework is sensitive to overall fragmentation landscape but inherently less sensitive to alignment errors — not a substitute for compound-level structural elucidation
- License is NOASSERTION; redistribution/modification not permitted until formal publication, requiring author permission

## Evidence

- [abstract] MS2toImg, a method that converts raw LC-MS/MS data into a two-dimensional images representing the global fragmentation pattern of each sample: "MS2toImg, a method that converts raw LC-MS/MS data into a two-dimensional images representing the global fragmentation pattern of each sample"
- [results] assigning the MS1 m/z to the x-axis and the MS2 m/z to the y-axis: "These values were then plotted onto a scatter plot, assigning the MS1 m/z to the x-axis and the MS2 m/z to the y-axis (Figure 1)."
- [results] we applied a masking procedure in which all pixels within this non-informative region were set to zero: [evidence span withheld — anchor status: not_found]
- [results] MS-DIAL[36] was used to preprocess raw data and extract the precursor (MS1) and fragment (MS2 or MS/MS) m/z pairs along with their corresponding intensities.: "For each sample, MS-DIAL[36] was used to preprocess raw data and extract the precursor (MS1) and fragment (MS2 or MS/MS) m/z pairs along with their corresponding intensities."
- [methods] Only peak intensity values greater than 10⁻⁴ were retained for further analysis.: "Only peak intensity values greater than 10⁻⁴ were retained for further analysis."
- [results] The entire framework was implemented in Python (v3.10) using the Keras (v2.14) and Tensor Flow libraries.: "The entire framework was implemented in Python (v3.10) using the Keras (v2.14) and Tensor Flow libraries."
