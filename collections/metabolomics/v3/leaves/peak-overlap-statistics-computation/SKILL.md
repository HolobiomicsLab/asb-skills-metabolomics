---
name: peak-overlap-statistics-computation
description: Use when generating augmented training instances from user-provided reference
  features and decoy/background signals for PeakBot's CNN peak-picking model training.
when_to_use_negative:
- Peak detection on new LC-HRMS chromatograms with an already-trained PeakBot model
  — overlap statistics are a training-instance-generation step, not part of peak picking/inference.
- Building the user-provided reference feature list (Peaks.tsv / Walls.tsv / Backgrounds.tsv)
  or matching references across samples — this precedes the instance-generation step.
- Grouping detected features into a data matrix or featureML file after model inference.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakBot
  role: Python package providing the GPU (CUDA)-accelerated training-instance generator
    that combines reference areas with decoys, enforces overlap constraints, and updates
    overlapping peak boundaries
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow
  role: Consumes the generated overlap-annotated training instances to train the CNN
    peak-picking model (peak type, center, bounding box)
- name: CUDA toolkit
  role: Accelerates generation of the large number of augmented training instances
    over which overlap statistics are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:ef51531f6ef72dec088d0bdd57e5e510f3b1928735a9e1c9d24697d2e95a0c84
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
  - Python
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# peak-overlap-statistics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes and enforces per-instance overlap statistics (retention-time width overlap fraction and peak-area IoU) during GPU-accelerated generation of augmented CNN training instances in PeakBot, ensuring decoy peaks do not over-shadow ground-truth references.

## When to use

Use when generating augmented training instances from user-provided reference features and decoy/background signals for PeakBot's CNN peak-picking model training.

## When NOT to use

- Peak detection on new LC-HRMS chromatograms with an already-trained PeakBot model — overlap statistics are a training-instance-generation step, not part of peak picking/inference.
- Building the user-provided reference feature list (Peaks.tsv / Walls.tsv / Backgrounds.tsv) or matching references across samples — this precedes the instance-generation step.
- Grouping detected features into a data matrix or featureML file after model inference.

## Inputs

- Ground-truth reference features exported as super-sampled standardized retention-time x m/z areas from training LC-HRMS chromatograms
- Decoy (distraction) chromatographic peaks and background signal types (e.g. walls spanning large parts of chromatograms)
- GPU (CUDA) instance-generation context with TensorFlow installed

## Outputs

- training_instances.csv — parseable table of generated instances with class labels (peak type, center, bounding box) and per-instance overlap statistics (width-overlap fraction, peak-area IoU)
- Boundary-updated peak annotations for overlapping features within each generated instance

## How to apply

During CUDA-accelerated training-instance generation, combine ground-truth standardized (rt x mz) reference areas with randomly placed decoy peaks and background types such as walls. For each generated instance, compute the width-overlap fraction (fraction of peak width shared between combined features) and the peak-area Intersection-over-Union between features. Enforce the constraints from the article: width-overlap must not exceed 50% of peak width and peak-area IoU must not exceed 33%; where features overlap, automatically update the peak boundaries of the overlapping features. Assign each generated instance its class labels (peak type, center, bounding box) and write the instances with their overlap statistics to a CSV table (training_instances.csv) consumed by the TensorFlow CNN model.

## Related tools

- **PeakBot** (Python package providing the GPU (CUDA)-accelerated training-instance generator that combines reference areas with decoys, enforces overlap constraints, and updates overlapping peak boundaries) — https://github.com/christophuv/PeakBot
- **TensorFlow** (Consumes the generated overlap-annotated training instances to train the CNN peak-picking model (peak type, center, bounding box))
- **CUDA toolkit** (Accelerates generation of the large number of augmented training instances over which overlap statistics are computed)

## Examples

```
from peakbot.train import cuda; cuda.generateTestInstances(chromatogram, referenceFeatures, walls, backgrounds)  # generates overlap-constrained training instances for peakbot.trainPeakBotModel
```

## Evaluation signals

- training_instances.csv exists and parses as a table containing class-label fields (peak type, center, bounding box) plus overlap-statistic fields
- Every generated instance satisfies width-overlap <= 0.5 (maximum 50% of peak width)
- Every generated instance satisfies peak-area IOU <= 0.33 (maximum 33%)
- Overlapping peaks' boundaries are automatically updated and consistent with the annotated bounding boxes in the output table

## Limitations

- Requires CUDA-enabled NVIDIA graphics card plus CUDA toolkit and cuDNN libraries for fast instance generation
- PeakBot distinguishes only between true chromatographic peaks and supported background types (e.g. walls); other interference patterns may not be represented
- Reference feature and background values are determined automatically via a gradient-descend approach and can vary from chromatogram to chromatogram, so overlap statistics depend on per-chromatogram feature geometry

## Evidence

- [full_text] During training-instance generation, other features/background signals (decoys) are randomly placed into the ground-truth standardized area, with overlap limited to maximum 50% of peak width and maximum 33% peak-area IOU; overlapping peaks' boundaries are automatically updated.: [evidence span withheld — anchor status: not_found]
- [readme] The matched references are then used to generate a large number of training instances by iteratively combining them. Each such training instance can consist of a chromatographic peak or background signal and several other distraction peaks to generalize (augmentation of the training dataset).: [evidence span withheld — anchor status: external_doc]
- [readme] Instance generation function used to build the training set from chromatogram plus reference and background lists.: [evidence span withheld — anchor status: external_doc]
