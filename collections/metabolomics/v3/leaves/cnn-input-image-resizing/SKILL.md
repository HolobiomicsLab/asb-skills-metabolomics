---
name: cnn-input-image-resizing
description: Reference for selecting and applying the correct TensorFlow image-resize
  target dimensions when converting LC-MS/MS scatter fingerprints into fixed-size
  inputs for the MS2toImg CNN, especially when reproducing the wild soybean bioactivity
  benchmark.
when_to_use_negative:
- The input is already a feature/peak table (e.g. MS-DIAL .txt output) rather than
  a rendered grayscale image — resize applies only to the image stage of the pipeline.
- The CNN checkpoint being reproduced was trained at a different resolution than the
  one being applied — the input layer size is fixed.
- Target is a multi-channel or false-color representation rather than the single-channel
  grayscale molecular fingerprint the framework defines.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow
  role: Provides the image resize function used to rescale the rendered grayscale
    fingerprint to the target CNN input dimensions.
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: matplotlib
  role: Renders the upstream 600-dpi scatter image that is the input to this resizing
    step.
- name: Python 3.10
  role: Runtime environment for the resizing and model-training pipeline.
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:8126decee22bdcc2d9fbc819b3d7d26ecb83cb801c35782e7aa509eccaec147d
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - Python
  - Python 3.10
  - TensorFlow
  - matplotlib
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cnn-input-image-resizing

## Summary

Resizes rendered 600-dpi grayscale molecular fingerprint images (MS1 m/z on x, MS2 m/z on y, masked above the diagonal) to one of three canonical CNN input sizes — 75×70, 150×140, or 300×280 pixels — using TensorFlow's image resize utility, balancing predictive accuracy against computational cost.

## When to use

Reference for selecting and applying the correct TensorFlow image-resize target dimensions when converting LC-MS/MS scatter fingerprints into fixed-size inputs for the MS2toImg CNN, especially when reproducing the wild soybean bioactivity benchmark.

## When NOT to use

- The input is already a feature/peak table (e.g. MS-DIAL .txt output) rather than a rendered grayscale image — resize applies only to the image stage of the pipeline.
- The CNN checkpoint being reproduced was trained at a different resolution than the one being applied — the input layer size is fixed.
- Target is a multi-channel or false-color representation rather than the single-channel grayscale molecular fingerprint the framework defines.

## Inputs

- 600-dpi grayscale PNG scatter image per sample (MS1 m/z x-axis, MS2 m/z y-axis, diagonal region masked to zero)
- Target resolution specification as width/height pair: 75×70, 150×140, or 300×280

## Outputs

- Tensor of shape (target_height, target_width, 1) suitable as CNN input, with masked MS2>MS1 region still zero-valued

## How to apply

After rendering each sample's grayscale scatter image at 600 dpi via matplotlib (MS1 m/z on the x-axis, MS2 m/z on the y-axis, with the MS2>MS1 region masked to black), load the PNG and pass it through the TensorFlow image resize function to one of the paper's three trained resolutions: 75×70, 150×140, or 300×280 pixels. The three sizes are not arbitrary — the article reports optimizing input resolution to trade predictive ability against computational efficiency, so the choice must match the checkpoint/benchmark being reproduced (main.py accepts width and height as CLI arguments). Verify the resized output matches the requested pixel dimensions exactly and that the masked region remains entirely black (zero intensity) after interpolation, since bicubic-style resampling could otherwise bleed values into the non-informative diagonal region. Use the same resolution consistently across the full dataset (all 545 wild soybean accessions) because the CNN input layer is fixed at that size.

## Related tools

- **TensorFlow** (Provides the image resize function used to rescale the rendered grayscale fingerprint to the target CNN input dimensions.) — https://github.com/jsehhs/MS2toImg-CNN
- **matplotlib** (Renders the upstream 600-dpi scatter image that is the input to this resizing step.)
- **Python 3.10** (Runtime environment for the resizing and model-training pipeline.)

## Examples

```
python main.py 75 70
```

## Evaluation signals

- Resized image dimensions exactly equal the requested (width, height) pair — 75×70, 150×140, or 300×280 pixels.
- All pixels corresponding to the MS2 > MS1 region remain zero (black) after resizing.
- Exactly one resized image exists per sample (545 wild soybean accessions in the benchmark).
- Consistent channel count: output remains single-channel grayscale.
- Downstream CNN accepts the resized tensor without shape errors at its fixed input layer.

## Limitations

- The article does not name the specific interpolation mode used by the TensorFlow resize call, so bit-exact reproduction depends on the official MS2toImg-CNN implementation.
- Resizing cannot recover information lost when the original 600-dpi render compressed dynamic range; low-intensity product ions may wash out at 75×70.
- Only three resolutions were validated in the paper; using a non-standard size is untested for predictive accuracy.

## Evidence

- [results] Three image resolutions were explored to balance predictive accuracy and computational efficiency: 75×70, 150×140, and 300×280 pixels.: [evidence span withheld — anchor status: not_found]
- [results] Resized the rendered images with the TensorFlow image resize function to 75×70, 150×140, and 300×280 pixels, the sizes used for CNN model training.: [evidence span withheld — anchor status: not_found]
