---
name: lcmsms-fragmentation-image-encoding
description: Use when you have MS-DIAL-preprocessed LC-MS/MS feature tables with precursor
  (MS1) and fragment (MS2) m/z pairs plus intensities and need fixed-size image inputs
  for CNN-based bioactivity prediction of complex metabolomics extracts.
when_to_use_negative:
- Input is already an aligned feature matrix intended for tabular models like Random
  Forest, PCA, or OPLS-DA — this skill converts data to images instead
- The analysis goal is structural identification or annotation of individual molecular
  features (e.g., CANOPUS/MetDNA/Spec2Vec-style annotation); this encoding deliberately
  bypasses compound identification
- Data lacks MS2 fragment information (MS1-only features), since everything above
  the diagonal would be masked and no fragmentation landscape would be encoded
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and extracts the precursor (MS1) and fragment
    (MS2) m/z pairs with intensities that feed the image encoder
- name: matplotlib
  role: Renders the grayscale scatter-plot fingerprint images at 600 dpi with precursor
    intensity as point size and product-ion intensity as grayscale color
- name: TensorFlow
  role: Provides the image resize function used to produce the 75×70, 150×140, and
    300×280 pixel inputs for CNN model training
- name: MS2toImg-CNN
  role: Official implementation exposing main.py, which takes target image width and
    height arguments to train on the encoded fingerprints
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:6d0b9b719c9ad267df98690a0dee406f49b7a256a8f81ac41c04daef51a0f6d4
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - MS-DIAL
  - MS2toImg-CNN
  - Python
  - TensorFlow
  - matplotlib
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lcmsms-fragmentation-image-encoding

## Summary

Encodes raw LC-MS/MS fragmentation data into a single-channel 2D grayscale 'molecular fingerprint' image per sample, where MS1 m/z (100–1500) is the x-axis, MS2 m/z (0–1500) the y-axis, precursor intensity the point size, product-ion intensity the grayscale value, and the physically impossible MS2>MS1 region is masked to black — providing CNN-ready input without feature alignment or compound identification.

## When to use

Use when you have MS-DIAL-preprocessed LC-MS/MS feature tables with precursor (MS1) and fragment (MS2) m/z pairs plus intensities and need fixed-size image inputs for CNN-based bioactivity prediction of complex metabolomics extracts.

## When NOT to use

- Input is already an aligned feature matrix intended for tabular models like Random Forest, PCA, or OPLS-DA — this skill converts data to images instead
- The analysis goal is structural identification or annotation of individual molecular features (e.g., CANOPUS/MetDNA/Spec2Vec-style annotation); this encoding deliberately bypasses compound identification
- Data lacks MS2 fragment information (MS1-only features), since everything above the diagonal would be masked and no fragmentation landscape would be encoded

## Inputs

- MS-DIAL result txt file (feature table) with MS1/MS2 m/z pairs and their intensities per sample
- Peak intensity values filtered to > 10^-4
- TIC-normalized, adduct-filtered ([M-H]-, [M-H2O-H]-, [M+Na-2H]-, [M-K-2H]-, [M-FA-H]-) feature intensities

## Outputs

- One single-channel grayscale PNG molecular fingerprint image per sample, rendered at 600 dpi
- Resized image tensors at 75×70, 150×140, or 300×280 pixels for CNN training
- Images with the MS2 > MS1 region entirely black (pixel value 0)

## How to apply

First preprocess raw data with MS-DIAL to extract precursor MS1 m/z, precursor ion intensity, product MS2 m/z values, and product ion intensities per sample (Python/pandas). Normalize precursor ion intensities to the 0–1 range, and scale each product ion intensity between 0 and 1 relative to the product ions of the same precursor. Render a grayscale scatter image with matplotlib at 600 dpi: MS1 m/z on the x-axis (100–1500 m/z), MS2 m/z on the y-axis (0–1500 m/z), precursor intensity as point size, product-ion intensity as grayscale color intensity. Apply the diagonal masking procedure that sets every pixel above the diagonal (MS2 > MS1) to zero (black), retaining only physically possible fragmentation events. Finally resize with the TensorFlow image resize function to one of the three CNN training resolutions: 75×70, 150×140, or 300×280 pixels, invoked via main.py with those width/height arguments.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and extracts the precursor (MS1) and fragment (MS2) m/z pairs with intensities that feed the image encoder)
- **matplotlib** (Renders the grayscale scatter-plot fingerprint images at 600 dpi with precursor intensity as point size and product-ion intensity as grayscale color)
- **TensorFlow** (Provides the image resize function used to produce the 75×70, 150×140, and 300×280 pixel inputs for CNN model training)
- **MS2toImg-CNN** (Official implementation exposing main.py, which takes target image width and height arguments to train on the encoded fingerprints) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 300 280
```

## Evaluation signals

- Exactly one grayscale PNG fingerprint image exists per sample, rendered at 600 dpi
- Resized outputs match one of the three specified pixel dimensions: 75×70, 150×140, or 300×280
- All pixels in the MS2 > MS1 region are entirely black (zero), confirming the diagonal mask was applied
- Precursor ion intensities fall within the normalized 0–1 range; product-ion intensities are scaled 0–1 relative to product ions of the same precursor
- Axes span the article's ranges: MS1 m/z 100–1500 on x, MS2 m/z 0–1500 on y

## Limitations

- The repository is under review and redistribution, modification, or use of the code for other research or commercial purposes is not permitted until formal publication (license NOASSERTION)
- The framework depends on MS-DIAL feature extraction quality; it is designed to be less sensitive to feature-alignment errors, but only possible fragmentation events (MS2 ≤ MS1) are retained — the upper-diagonal region carries no information
- Dependency versions differ between paper (Python 3.10, Keras 2.14) and README (tested with Python 3.11), so the toolchain is not fully pinned

## Evidence

- [results] we applied a masking procedure in which all pixels within this non-informative region were set to zero (black), ensuring that only data corresponding to possible fragmentation events are used: [evidence span withheld — anchor status: not_found]
