---
name: grayscale-scatter-plot-rendering
description: Use when raw LC-MS/MS feature tables (MS1/MS2 m/z pairs with intensities,
  e.g. from MS-DIAL output) must be converted into fixed-size grayscale images for
  image-based deep learning such as CNN bioactivity prediction.
when_to_use_negative:
- Input is already an aligned feature matrix or imaged spectra — the skill is for
  raw MS-DIAL-extracted m/z/intensity pairs, not tabular features destined for PCA/OPLS-DA/Random
  Forest
- A task requires structurally annotated or identified compounds — MS2toImg intentionally
  bypasses compound identification, so annotations are neither needed nor used
- Positive-ion-mode or adducts outside the selected set ([M-H]-, [M-H2O-H]-, [M+Na-2H]-,
  [M-K-2H]-, [M-FA-H]-) where the MS2>MS1 diagonal masking assumption and normalization
  scheme were validated
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and extracts the precursor (MS1) and fragment
    (MS2) m/z pairs with corresponding intensities that feed the scatter rendering
- name: matplotlib
  role: Renders the normalized m/z/intensity pairs as the grayscale scatter-plot image
    and applies the diagonal mask
- name: TensorFlow
  role: Resizes the rendered fingerprint images to 75x70, 150x140, or 300x280 pixels
    for CNN training
- name: MS2toImg-CNN
  role: Official implementation whose main.py consumes the rendered images at a chosen
    resolution
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:97ea5b70706eba99a6ee32b00b904f7e3b9b34b7ad85b1d69f7db1ce98964c34
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# grayscale-scatter-plot-rendering

## Summary

Encodes LC-MS/MS precursor/product m/z and intensity data into a single-channel 2D grayscale scatter-plot image (MS1 m/z on x-axis, MS2 m/z on y-axis) with the physically impossible MS2>MS1 region masked to black, producing CNN-ready 'molecular fingerprint' images without feature alignment or annotation.

## When to use

Use when raw LC-MS/MS feature tables (MS1/MS2 m/z pairs with intensities, e.g. from MS-DIAL output) must be converted into fixed-size grayscale images for image-based deep learning such as CNN bioactivity prediction.

## When NOT to use

- Input is already an aligned feature matrix or imaged spectra — the skill is for raw MS-DIAL-extracted m/z/intensity pairs, not tabular features destined for PCA/OPLS-DA/Random Forest
- A task requires structurally annotated or identified compounds — MS2toImg intentionally bypasses compound identification, so annotations are neither needed nor used
- Positive-ion-mode or adducts outside the selected set ([M-H]-, [M-H2O-H]-, [M+Na-2H]-, [M-K-2H]-, [M-FA-H]-) where the MS2>MS1 diagonal masking assumption and normalization scheme were validated

## Inputs

- MS-DIAL result table with per-sample precursor MS1 m/z, precursor ion intensity, product MS2 m/z values, and product ion intensities
- Target output image resolution (75x70, 150x140, or 300x280 pixels)

## Outputs

- One single-channel grayscale PNG 'molecular fingerprint' image per sample (MS1 m/z on x-axis 100-1500, MS2 m/z on y-axis 0-1500, MS2>MS1 region black)
- Resized images at 75x70, 150x140, or 300x280 pixels for CNN input

## How to apply

Start from an MS-DIAL-preprocessed feature table containing precursor (MS1) m/z, precursor ion intensity, product (MS2) m/z values, and product ion intensities per sample. Normalize the precursor ion intensity to the 0-1 range, and scale each product ion intensity between 0 and 1 relative to the product ions of the same precursor ion; optionally retain only peak intensities greater than 10^-4. Plot each sample as a grayscale scatter image with MS1 m/z on the x-axis (100-1500 m/z) and MS2 m/z on the y-axis (0-1500 m/z), encoding precursor intensity as point size and product-ion intensity as grayscale color intensity (e.g. via matplotlib at 600 dpi). Apply a masking procedure that sets every pixel in the region above the diagonal (MS2 > MS1) to zero (black), because a fragment cannot have a higher m/z than its precursor, so only physically possible fragmentation events should remain. Finally, resize the rendered images (e.g. with the TensorFlow image resize function) to one of the training resolutions 75x70, 150x140, or 300x280 pixels used by the MS2toImg-CNN pipeline. This encoding deliberately leverages the overall fragmentation landscape rather than precise feature matching, which makes downstream CNN prediction less sensitive to alignment errors.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and extracts the precursor (MS1) and fragment (MS2) m/z pairs with corresponding intensities that feed the scatter rendering)
- **matplotlib** (Renders the normalized m/z/intensity pairs as the grayscale scatter-plot image and applies the diagonal mask)
- **TensorFlow** (Resizes the rendered fingerprint images to 75x70, 150x140, or 300x280 pixels for CNN training)
- **MS2toImg-CNN** (Official implementation whose main.py consumes the rendered images at a chosen resolution) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 75 70
```

## Evaluation signals

- Exactly one grayscale PNG fingerprint image exists per sample, rendered at 600 dpi
- All pixels in the region above the diagonal (MS2 > MS1) are zero (black) in every output image
- Precursor ion intensities fall in the 0-1 range; product ion intensities are 0-1 relative to products of the same precursor
- Resized outputs match the three specified pixel dimensions exactly: 75x70, 150x140, and 300x280
- Axes span the specified ranges: MS1 m/z 100-1500 on x, MS2 m/z 0-1500 on y

## Limitations

- The MS2>MS1 masking discards everything above the diagonal; any feature extraction or noise in that region is unrecoverable after rendering
- The rendering bypasses precise feature alignment, so it depends on the overall fragmentation landscape and may not resolve individual features when m/z scatter points overlap
- Repository code is restricted until formal publication: 'Redistribution, modification, or use of the code for other research or commercial purposes is not permitted until formal publication'

## Evidence

- [results] we applied a masking procedure in which all pixels within this non-informative region were set to zero (black), ensuring that only data corresponding to possible fragmentation events are used for: [evidence span withheld — anchor status: not_found]
- [readme] to train model with 75 x 70 image:
```
python main.py 75 70
```: "to train model with 75 x 70 image:"
