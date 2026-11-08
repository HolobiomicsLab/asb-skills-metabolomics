---
name: spectral-diagonal-region-masking
description: Use when converting MS-DIAL-derived precursor/fragment m/z and intensity
  pairs into a 2D grayscale fingerprint image and needing to remove the structurally
  impossible MS2 > MS1 region before CNN training.
when_to_use_negative:
- Analyzing raw vendor LC-MS/MS files directly — MS-DIAL preprocessing to extract
  MS1/MS2 m/z-intensity pairs must happen first
- Data where fragment ions may legitimately exceed precursor m/z (e.g., in-source
  fragments or misassigned precursors) — blanket diagonal masking would discard real
  signal
- The goal is peak-table-based statistical analysis (PCA, OPLS-DA, Random Forest)
  rather than image-based CNN input
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and extracts precursor (MS1) and fragment (MS2)
    m/z pairs with intensities that feed the image conversion
- name: matplotlib
  role: Renders the grayscale scatter image with MS1 m/z on the x-axis and MS2 m/z
    on the y-axis at 600 dpi
- name: TensorFlow
  role: Resizes rendered fingerprint images to 75x70, 150x140, and 300x280 pixels
    for CNN model training
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:24f6d1135ff4bf39dc6f5d7f591ac1b8fa1e8e6285a94f592f23b238b5c5a01b
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - MS-DIAL
  - Python
  - TensorFlow
  - matplotlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectral-diagonal-region-masking

## Summary

Reconstruct the MS2toImg encoding step: render LC-MS/MS MS1/MS2 m/z and intensity pairs as a single-channel grayscale scatter image and mask the physically impossible region above the diagonal (MS2 > MS1) to zero.

## When to use

Use when converting MS-DIAL-derived precursor/fragment m/z and intensity pairs into a 2D grayscale fingerprint image and needing to remove the structurally impossible MS2 > MS1 region before CNN training.

## When NOT to use

- Analyzing raw vendor LC-MS/MS files directly — MS-DIAL preprocessing to extract MS1/MS2 m/z-intensity pairs must happen first
- Data where fragment ions may legitimately exceed precursor m/z (e.g., in-source fragments or misassigned precursors) — blanket diagonal masking would discard real signal
- The goal is peak-table-based statistical analysis (PCA, OPLS-DA, Random Forest) rather than image-based CNN input

## Inputs

- MS-DIAL result txt file with precursor MS1 m/z, precursor intensity, product MS2 m/z values, and product ion intensities per sample

## Outputs

- One grayscale PNG molecular fingerprint image per sample (600 dpi)
- TensorFlow-resized copies at 75x70, 150x140, and 300x280 pixels for CNN training

## How to apply

Load the MS-DIAL result txt file and extract four fields per sample: precursor MS1 m/z, precursor ion intensity, product MS2 m/z values, and product ion intensities (Python/pandas), retaining only peak intensities > 10^-4. Normalize precursor ion intensities to 0-1 and scale product ion intensities to 0-1 relative to the product ions of the same precursor. Render a grayscale scatter plot with matplotlib, MS1 m/z on the x-axis (100-1500 m/z) and MS2 m/z on the y-axis (0-1500 m/z), encoding precursor intensity as point size and product-ion intensity as grayscale color intensity, at 600 dpi. Apply the diagonal masking: set all pixels where MS2 > MS1 to zero (black), since a fragment ion cannot have a higher m/z than its precursor, so this region carries no fragmentation information. Finally, resize the images with the TensorFlow image resize function to 75x70, 150x140, and 300x280 pixels for CNN input.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and extracts precursor (MS1) and fragment (MS2) m/z pairs with intensities that feed the image conversion)
- **matplotlib** (Renders the grayscale scatter image with MS1 m/z on the x-axis and MS2 m/z on the y-axis at 600 dpi)
- **TensorFlow** (Resizes rendered fingerprint images to 75x70, 150x140, and 300x280 pixels for CNN model training) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 75 70
```

## Evaluation signals

- Exactly one grayscale PNG fingerprint image exists per sample, rendered at 600 dpi
- The resized outputs match the three specified pixel dimensions (75x70, 150x140, 300x280)
- All pixels in the MS2 > MS1 region are entirely black (zero intensity)
- Precursor intensities fall within 0-1 and product ion intensities are scaled relative to product ions of the same precursor
- Axis ranges match MS1 m/z 100-1500 (x) and MS2 m/z 0-1500 (y)

## Limitations

- Licensing restriction: redistribution, modification, or use of the code for other research or commercial purposes is not permitted until formal publication; contact the authors for permission
- README tested only with Python 3.11 while the paper reports Python 3.10 with Keras 2.14; framework versions may differ
- Numeric CNN performance values, hyperparameters (Supplementary Table 1), and detailed conversion rules beyond those stated are not fully specified in the text

## Evidence

- [results] Mask non-informative region above the diagonal (MS2 > MS1) to zero: [evidence span withheld — anchor status: not_found]
