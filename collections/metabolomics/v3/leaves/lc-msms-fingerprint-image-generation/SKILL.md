---
name: lc-msms-fingerprint-image-generation
description: Use when raw untargeted LC-MS/MS metabolomics data (MS1/MS2 m/z–intensity
  pairs from MS-DIAL) must be encoded as 2D grayscale fingerprint images as CNN input
  for bioactivity classification without compound annotation.
when_to_use_negative:
- Input is already an aligned feature table or annotated feature matrix — the whole
  point of this skill is to avoid feature alignment and identification
- The analysis goal requires structural identification or annotation of individual
  metabolite features (e.g., CANOPUS/MetDNA/Spec2Vec-style annotation) rather than
  holistic fingerprint-based prediction
- The dataset comes from a data-acquisition mode or adduct set outside the validated
  negative-mode list ([M-H]-, [M-H2O-H]-, [M+Na-2H]-, [M-K-2H]-, [M-FA-H]-), where
  the fragmentation landscape and masking assumptions may not hold
edam_operation: http://edamontology.org/operation_2945
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data to extract MS1/MS2 m/z pairs and corresponding
    intensities used as the coordinates and values of the fingerprint image
- name: matplotlib
  role: Renders the 2D grayscale scatter-plot molecular fingerprint images at 600
    dpi with precursor intensity as point size and product intensity as grayscale
- name: TensorFlow
  role: Resizes the rendered fingerprint images to the target resolutions (75x70,
    150x140, 300x280) via the TensorFlow image resize function
- name: MS2toImg-CNN
  role: Official implementation containing main.py, which drives the image generation
    and resizing pipeline invoked as 'python main.py <width> <height>'
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_content_hash: sha256:dd04c09f6872bd3350f912e2f635100700a6d250c2180c79fccd6ca070e48a23
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - Keras
  - MS-DIAL
  - MS2toImg
  - MS2toImg-CNN
  - Python
  - Python 3.10
  - TensorFlow
  - matplotlib
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# lc-msms-fingerprint-image-generation

## Summary

Converts raw LC-MS/MS fragmentation data (MS-DIAL-extracted MS1/MS2 m/z pairs with intensities) into single-channel 2D grayscale 'molecular fingerprint' scatter-plot images that serve as direct CNN input for bioactivity prediction, bypassing feature alignment and compound identification.

## When to use

Use when raw untargeted LC-MS/MS metabolomics data (MS1/MS2 m/z–intensity pairs from MS-DIAL) must be encoded as 2D grayscale fingerprint images as CNN input for bioactivity classification without compound annotation.

## When NOT to use

- Input is already an aligned feature table or annotated feature matrix — the whole point of this skill is to avoid feature alignment and identification
- The analysis goal requires structural identification or annotation of individual metabolite features (e.g., CANOPUS/MetDNA/Spec2Vec-style annotation) rather than holistic fingerprint-based prediction
- The dataset comes from a data-acquisition mode or adduct set outside the validated negative-mode list ([M-H]-, [M-H2O-H]-, [M+Na-2H]-, [M-K-2H]-, [M-FA-H]-), where the fragmentation landscape and masking assumptions may not hold

## Inputs

- Raw LC-MS/MS metabolomics data preprocessed by MS-DIAL into per-sample lists of MS1 precursor m/z, MS2 fragment m/z, and their intensities (e.g., from the 545 wild soybean Glycine soja accession dataset)
- Per-sample bioactivity labels (e.g., normalized DPPH values with <0.4 = low activity, >0.6 = high activity) for downstream CNN training

## Outputs

- Single-channel 2D grayscale molecular fingerprint images per sample at 600 dpi, masked above the MS1–MS2 diagonal
- TensorFlow-resized image sets at 75x70, 150x140, or 300x280 pixels ready for CNN training

## How to apply

First preprocess raw LC-MS/MS files with MS-DIAL to extract precursor (MS1) and fragment (MS2) m/z pairs with intensities, retaining only peak intensities greater than 10^-4, normalizing intensities with the TIC method, selecting adduct ions ([M-H]-, [M-H2O-H]-, [M+Na-2H]-, [M-K-2H]-, [M-FA-H]-), merging equivalent features when the average MS1/RT/MS/MS similarity exceeds 0.8 with a 1-minute RT alignment tolerance. Scale precursor ion intensity between 0 and 1 and product ion intensities relative to product ions of the same precursor. Render a grayscale scatter plot per sample at 600 dpi with matplotlib: MS1 m/z on the x-axis, MS2 m/z on the y-axis, precursor intensity as point size, product intensity as grayscale value. Apply a masking procedure setting all pixels in the non-informative region above the diagonal (MS2 m/z > MS1 m/z) to zero, since only physically possible fragmentation events should be encoded. Resize images with the TensorFlow resize function to the chosen target resolution (75x70, 150x140, or 300x280) via 'python main.py <width> <height>'; 150x140 is the validated optimum balancing accuracy and computational efficiency.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data to extract MS1/MS2 m/z pairs and corresponding intensities used as the coordinates and values of the fingerprint image)
- **matplotlib** (Renders the 2D grayscale scatter-plot molecular fingerprint images at 600 dpi with precursor intensity as point size and product intensity as grayscale)
- **TensorFlow** (Resizes the rendered fingerprint images to the target resolutions (75x70, 150x140, 300x280) via the TensorFlow image resize function)
- **MS2toImg-CNN** (Official implementation containing main.py, which drives the image generation and resizing pipeline invoked as 'python main.py <width> <height>') — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 150 140
```

## Evaluation signals

- Verify every pixel in the region where MS2 m/z > MS1 m/z is zero (black) after masking, since these are physically impossible fragmentation events
- Check precursor ion intensities are scaled between 0 and 1 and product ion intensities are scaled relative to product ions of the same precursor
- Confirm output image dimensions exactly match the requested target (e.g., 150x140) and that images were rendered at 600 dpi
- Confirm only peak intensity values greater than 10^-4 were retained during MS-DIAL preprocessing
- Downstream CNN validation accuracy at the resulting resolution should meet or exceed the RF baseline (RF AUC 0.817 vs CNN AUC 0.96 on DPPH)

## Limitations

- High-resolution images (300x280) tend to overfit (large training–validation gap) while low resolution (75x70) underperforms; 150x140 is the validated tradeoff
- The masking step discards the entire MS2 > MS1 region, so any information in that non-informative area is irrecoverably lost
- The framework is explicitly designed to be robust to alignment errors by leveraging the overall fragmentation landscape rather than precise feature matching, but it provides no per-feature structural interpretation
- The README states the code is under peer review and redistribution, modification, or use for other research or commercial purposes is not permitted until formal publication

## Evidence

- [results] diagonal masking: [evidence span withheld — anchor status: not_found]
