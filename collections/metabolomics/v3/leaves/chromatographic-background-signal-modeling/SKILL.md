---
name: chromatographic-background-signal-modeling
description: Reference for modeling non-peak background signals (e.g. walls spanning
  large chromatogram parts) when generating augmented training instances for peak-picking
  CNN training in LC-HRMS.
when_to_use_negative:
- If the goal is peak detection in already-trained models on new chromatograms (use
  PeakBot's runPeakBot instead of instance generation)
- If the chromatograms contain no relevant background signal types (walls) that need
  to be modeled for CNN differentiation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakBot
  role: Implements the GPU (CUDA) based generation of training instances combining
    reference peaks with background signals such as walls, for CNN training
  repo: https://github.com/christophuv/PeakBot
- name: Python
  role: Language in which PeakBot and the background-signal augmentation logic are
    implemented
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow
  role: Package in which the CNN model trained on the augmented instances (with background
    classes) is implemented
  repo: https://github.com/christophuv/PeakBot
- name: CUDA toolkit
  role: Accelerates generation of the large number of training instances including
    background-signal combinations
- name: cuDNN
  role: Library required alongside CUDA for the GPU-accelerated training-instance
    generation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:c2b2d0ad1736125f371707b8e060e20c6070740a73ada261cafb4817dc15dd11
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
  - cuDNN
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# chromatographic-background-signal-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Modeling of background signal types (e.g. 'walls') during training-instance generation in PeakBot so the CNN learns to differentiate true chromatographic peaks from irrelevant background information.

## When to use

Reference for modeling non-peak background signals (e.g. walls spanning large chromatogram parts) when generating augmented training instances for peak-picking CNN training in LC-HRMS.

## When NOT to use

- If the goal is peak detection in already-trained models on new chromatograms (use PeakBot's runPeakBot instead of instance generation)
- If the chromatograms contain no relevant background signal types (walls) that need to be modeled for CNN differentiation

## Inputs

- User-provided reference feature list (TSV with RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation)
- Wall background list (TSV with MZ, Rtstart, Rtend, MZdeviation)
- Other background list (TSV with RTStart, RTEnd, MZLow, MZHigh)
- LC-HRMS profile-mode training chromatograms

## Outputs

- Augmented training instances combining peaks/background signals with bounded overlap (max 50% width overlap, max 33% area IoU)
- Class labels per instance (peak type, center, bounding-box) for CNN training
- Generated instance set used as input to peakbot.trainPeakBotModel

## How to apply

During training-instance generation in PeakBot, combine ground-truth reference features not only with random decoy peaks but also with explicitly defined background signal types, notably 'walls' that span the entire or large parts of the chromatograms. Walls are specified in a tab-separated-values file with columns MZ, Rtstart, Rtend, and MZdeviation, where a random retention time between Rtstart and Rtend is sampled (narrow windows can be used to target specific walls); additional 'other background' regions are defined via RTStart, RTEnd, MZLow, MZHigh columns. Enforce overlap constraints of maximum 50% peak-width overlap and maximum 33% peak-area IoU when placing background signals or decoys into the ground-truth standardized area, automatically updating boundaries of overlapping features. The rationale is to generalize augmentation and teach the CNN to reject irrelevant background information rather than only recognizing true chromatographic peaks.

## Related tools

- **PeakBot** (Implements the GPU (CUDA) based generation of training instances combining reference peaks with background signals such as walls, for CNN training) — https://github.com/christophuv/PeakBot
- **Python** (Language in which PeakBot and the background-signal augmentation logic are implemented) — https://github.com/christophuv/PeakBot
- **TensorFlow** (Package in which the CNN model trained on the augmented instances (with background classes) is implemented) — https://github.com/christophuv/PeakBot
- **CUDA toolkit** (Accelerates generation of the large number of training instances including background-signal combinations)
- **cuDNN** (Library required alongside CUDA for the GPU-accelerated training-instance generation)

## Examples

```
from peakbot.train import cuda; cuda.generateTestInstances(chromatogram, referencePeaks_tsv, walls_tsv, backgrounds_tsv)
```

## Evaluation signals

- Each generated instance satisfies width-overlap <= 0.5 and peak-area IoU <= 0.33
- training_instances.csv is a parseable table containing class-label and overlap-statistic fields
- Trained CNN correctly distinguishes chromatographic peaks from background signals (e.g. walls) with high accuracy (~0.99 in article training/validation datasets)
- At least 100 distinct reference features (including isotopologs) are used, per article/README recommendation
- Example detection images exported for user-verification show peaks distinguished from wall-type background signals

## Limitations

- Care must be taken that no co-eluting compounds (isomers) are present when automatically determining reference feature values per chromatogram
- intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM are difficult to evaluate manually; PeakBot offers estimateParameters.py for this
- A minimum of 100 reference features is needed for high-quality peak-picking results; walls spanning large parts of the chromatograms must be specified explicitly in TSV form
- If wall retention-time windows are too narrow or too broad, the CNN may fail to learn wall-background distinctions targeted by the user

## Evidence

- [readme] background signal types (e.g. walls spanning large parts of chromatograms) are supported so PeakBot differentiates true peaks from irrelevant background: "also different background types are supported by PeakBot so that it differentiates between true chromatographic peaks and irrelevant background information (e.g., walls, which are signals present"
- [readme] GPU acceleration for generating large numbers of training instances including background combinations: "As a large number of training instances is required to train the CNN model and to achieve a high performance of the model, a GPU (CUDA) based approach is implemented that decreases the time required"
- [readme] Wall specification format with random RT between Rtstart and Rtend: [evidence span withheld — anchor status: not_found]
- [abstract] Accuracy metric achieved by PeakBot on training and independent validation datasets: "PeakBot achieved a high performance with respect to discriminating between chromatographic peaks and background signals (accuracy of 0.99)"
- [readme] Requirement of at least 100 reference features for training: [evidence span withheld — anchor status: external_doc]
