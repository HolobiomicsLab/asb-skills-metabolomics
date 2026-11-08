---
name: cnn-training-instance-augmentation
description: Use when training a custom PeakBot CNN from LC-HRMS profile-mode chromatograms
  with ≥100 matched reference features, and you must synthesize augmented peak/background
  training instances with bounded decoy overlap.
when_to_use_negative:
- Input is already centroided or a processed feature table — PeakBot operates on LC-HRMS
  profile-mode data
- Goal is detecting peaks in new chromatograms with an already-trained PeakBot model
  — use `peakbot.cuda.preProcessChromatogram` and `peakbot.runPeakBot` (detect.py)
  instead
- You cannot supply at least 100 different reference features (isotopologs of the
  same feature count) — instance generation will not support a high-quality model
edam_operation: http://edamontology.org/operation_0389
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PeakBot
  role: Provides the GPU (CUDA)-based training-instance generator `peakbot.train.cuda.generateTestInstances`
    with decoy-placement and overlap-bound enforcement
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow 2.5.0
  role: Consumes the generated instances to train the peak-type/center/bounding-box
    CNN (`peakbot.trainPeakBotModel`)
  repo: https://github.com/christophuv/PeakBot
- name: CUDA toolkit
  role: Required GPU acceleration for the training-instance generation step
- name: Anaconda
  role: Recommended virtual environment so all CUDA dependencies are installed automatically
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:dff4afd73af9a2d91bdadaceb160fa54dd5493cadd48d37c5c78c39260b47010
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
  - Anaconda
  - CUDA toolkit
  - PeakBot
  - Python
  - TensorFlow 2.5.0
  - cuDNN
  edam_operation: http://edamontology.org/operation_0389
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cnn-training-instance-augmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a large augmented set of CNN training instances for PeakBot peak-picking by iteratively combining ground-truth reference features (exported as standardized rt x m/z areas) with randomly placed decoy/distraction peaks and background 'walls', accelerated on the GPU via CUDA.

## When to use

Use when training a custom PeakBot CNN from LC-HRMS profile-mode chromatograms with ≥100 matched reference features, and you must synthesize augmented peak/background training instances with bounded decoy overlap.

## When NOT to use

- Input is already centroided or a processed feature table — PeakBot operates on LC-HRMS profile-mode data
- Goal is detecting peaks in new chromatograms with an already-trained PeakBot model — use `peakbot.cuda.preProcessChromatogram` and `peakbot.runPeakBot` (detect.py) instead
- You cannot supply at least 100 different reference features (isotopologs of the same feature count) — instance generation will not support a high-quality model

## Inputs

- Matched ground-truth reference feature list (TSV with columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation)
- Wall background list (TSV with columns MZ, Rtstart, Rtend, MZdeviation)
- Other background list (TSV with columns RTStart, RTEnd, MZLow, MZHigh)
- LC-HRMS profile-mode training chromatograms
- CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN installed

## Outputs

- training_instances.csv containing generated instances with class labels (peak type, center, bounding box) and overlap statistics (width-overlap fraction, area IoU)
- Training dataset T (plus derived validation sets V, iT, iV, eV) consumed by peakbot.trainPeakBotModel

## How to apply

Extract ground-truth reference peaks from the training chromatograms (smoothing and gradient-descend peak detection with border/center estimation), then export each reference as a super-sampled standardized retention-time x m/z area. Using PeakBot's GPU (CUDA)-based instance generator (`peakbot.train.cuda.generateTestInstances`), iteratively combine the ground-truth areas with randomly placed distraction peaks and background signal types (walls spanning large parts of the chromatograms) to augment the training dataset. Enforce the combination constraints — maximum 50% peak-width overlap and maximum 33% peak-area IoU between combined features — automatically updating the boundaries of overlapping peaks. Assign each instance its class labels (peak type, peak center, bounding box) in the form consumed by the TensorFlow 2.5.0 CNN, and write the instances to training_instances.csv with per-instance overlap statistics.

## Related tools

- **PeakBot** (Provides the GPU (CUDA)-based training-instance generator `peakbot.train.cuda.generateTestInstances` with decoy-placement and overlap-bound enforcement) — https://github.com/christophuv/PeakBot
- **TensorFlow 2.5.0** (Consumes the generated instances to train the peak-type/center/bounding-box CNN (`peakbot.trainPeakBotModel`)) — https://github.com/christophuv/PeakBot
- **CUDA toolkit** (Required GPU acceleration for the training-instance generation step)
- **Anaconda** (Recommended virtual environment so all CUDA dependencies are installed automatically)

## Examples

```
curl https://raw.githubusercontent.com/christophuv/PeakBot_Example/main/quickExample_GPU.py > quickExample_GPU.py; python quickExample_GPU.py
```

## Evaluation signals

- training_instances.csv exists and parses as a table containing class-label fields and overlap-statistic fields
- Every generated instance satisfies width-overlap <= 0.5 and peak-area IoU <= 0.33
- At least 100 reference features (counting isotopologs) contributed matched ground-truth areas to the generator
- Downstream CNN training reaches peakType_ACCPeakNoPeak > 0.99 on iV with train/validation metric gap < 5%
- Instances include both peak/background class labels and wall-type background signals (e.g., walls spanning large parts of chromatograms)

## Limitations

- Care must be taken that no co-eluting compounds (isomers) are present in the reference chromatograms; actual reference values are recalculated per chromatogram via gradient-descend and vary between samples
- Peak shapes (fronting, tailing, etc.) must be represented in the reference list or the model will not recognize them
- A large number of training instances is required; without a CUDA-enabled Nvidia GPU (toolkit + cuDNN) generation is slow

## Evidence

- [readme] Each such training instance can consist of a chromatographic peak or background signal and several other distraction peaks to generalize (augmentation of the training dataset).: [evidence span withheld — anchor status: external_doc]
