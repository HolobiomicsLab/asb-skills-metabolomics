---
name: gpu-accelerated-synthetic-dataset-generation
description: Use when training a CNN peak-picking model needs many augmented instances
  from a limited set of user-provided reference features, and CPU-only instance generation
  is too slow.
when_to_use_negative:
- When the goal is peak picking on new, already-collected LC-HRMS chromatograms with
  a trained model — use PeakBot's detection path (peakbot.cuda.preProcessChromatogram
  then peakbot.runPeakBot) instead of generating training instances.
- When the user has fewer than ~100 different reference features; the paper indicates
  a minimum of 100 reference features is needed to achieve high-quality peak-picking
  results, so augmentation alone cannot compensate for too few references.
- When no CUDA-enabled GPU is available and instance-generation throughput is not
  a bottleneck — the CPU path exists but is slower.
edam_operation: http://edamontology.org/operation_3763
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0154
tools:
- name: PeakBot
  role: Provides the GPU (CUDA)-accelerated training-instance generator that iteratively
    combines matched reference features with distraction peaks and background types
    under overlap constraints
  repo: https://github.com/christophuv/PeakBot
- name: PeakBot_Example
  role: Supplies the quickExample_GPU.py script and reference/background TSV templates
    demonstrating how to invoke the training-instance generation and model training
    workflow
  repo: https://github.com/christophuv/PeakBot_Example
- name: TensorFlow
  role: Consumes the generated training instances to train the PeakBot-CNN model (peak
    type, center, bounding-box outputs)
- name: CUDA toolkit
  role: Required alongside cuDNN for the GPU-based instance generation; Anaconda virtual
    environment is recommended to install CUDA dependencies automatically
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:7047d3827789b1c2cb0c2256f21018d813d53665f4d8ccfde963be29e06f47e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/christophuv/PeakBot
  repo_url: https://github.com/christophuv/PeakBot
  tools:
  - CUDA toolkit
  - PeakBot
  - PeakBot_Example
  - Python
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3763
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# GPU-accelerated synthetic dataset generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate large numbers of augmented CNN training instances by iteratively combining ground-truth chromatographic peaks with randomly placed decoy peaks and background signals on a CUDA GPU, as implemented in PeakBot for LC-HRMS peak picking.

## When to use

Use when training a CNN peak-picking model needs many augmented instances from a limited set of user-provided reference features, and CPU-only instance generation is too slow.

## When NOT to use

- When the goal is peak picking on new, already-collected LC-HRMS chromatograms with a trained model — use PeakBot's detection path (peakbot.cuda.preProcessChromatogram then peakbot.runPeakBot) instead of generating training instances.
- When the user has fewer than ~100 different reference features; the paper indicates a minimum of 100 reference features is needed to achieve high-quality peak-picking results, so augmentation alone cannot compensate for too few references.
- When no CUDA-enabled GPU is available and instance-generation throughput is not a bottleneck — the CPU path exists but is slower.

## Inputs

- Matched ground-truth reference features from training LC-HRMS chromatograms (peaks with estimated borders and centers)
- User-provided reference feature list (TSV: RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation)
- Wall background list (TSV: MZ, Rtstart, Rtend, MZdeviation)
- Other background list (TSV: RTStart, RTEnd, MZLow, MZHigh)
- CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN libraries installed

## Outputs

- A large set of augmented training instances, each a standardized two-dimensional rt x mz area with class labels (peak type, center, bounding box)
- Validation/detection example images for user verification of instance composition

## How to apply

Take the matched reference features (ground-truth peaks with estimated borders and centers from the user's chromatograms) and export each as a super-sampled standardized retention-time by m/z area. Using PeakBot's GPU (CUDA)-accelerated instance generator, iteratively combine these references with randomly placed 'distraction' chromatographic peaks and background signal types (e.g. walls, which span large parts of the chromatogram) to build each training instance. Enforce the overlap constraints during combination: maximum 50% peak-width overlap and maximum 33% peak-area IoU between any combined features, automatically updating the boundaries of overlapping peaks where needed. Label each instance with its class labels (peak type, center, bounding box) so the TensorFlow CNN model can consume it. This augmentation generalizes the model beyond the finite reference set while keeping co-combined features physically plausible.

## Related tools

- **PeakBot** (Provides the GPU (CUDA)-accelerated training-instance generator that iteratively combines matched reference features with distraction peaks and background types under overlap constraints) — https://github.com/christophuv/PeakBot
- **PeakBot_Example** (Supplies the quickExample_GPU.py script and reference/background TSV templates demonstrating how to invoke the training-instance generation and model training workflow) — https://github.com/christophuv/PeakBot_Example
- **TensorFlow** (Consumes the generated training instances to train the PeakBot-CNN model (peak type, center, bounding-box outputs))
- **CUDA toolkit** (Required alongside cuDNN for the GPU-based instance generation; Anaconda virtual environment is recommended to install CUDA dependencies automatically)

## Examples

```
python quickExample_GPU.py  # or use peakbot.train.cuda.generateTestInstances followed by peakbot.trainPeakBotModel
```

## Evaluation signals

- Every generated instance satisfies the overlap constraints: width-overlap fraction <= 0.5 and peak-area IoU <= 0.33 between any combined features.
- Generated instance boundaries of overlapping peaks are automatically updated and consistent with the class labels (peak type, center, bounding box) attached to each instance.
- The instance set is large enough to train the CNN — compare loss and metric values from peakbot.trainPeakBotModel to the example's reported satisfaction criteria ('satisfying results with respect to the different loss and metric values').
- Spot-check exported detection-example images to confirm instances mix true chromatographic peaks with distraction peaks and background types (e.g. walls) as intended.
- At least 100 distinct reference features were used as the combination basis (isotopologs of the same compound count).

## Limitations

- Instance generation requires a CUDA-enabled Nvidia GPU with the CUDA toolkit and cuDNN libraries; without them the CPU path is used, which is slower for the large number of instances required.
- Only a minimum of ~100 reference features is stated as sufficient for high-quality peak picking; fewer references may still yield poor models even with augmentation.
- Care must be taken that no co-eluting compounds (isomers) are present in the reference chromatograms, otherwise the automatically determined reference feature values (via gradient-descend) may be unreliable.
- The overlap constraints (50% width overlap, 33% area IoU) bound how aggressively decoys may be placed into a ground-truth area; instances requiring denser overlaps fall outside this scheme.

## Evidence
