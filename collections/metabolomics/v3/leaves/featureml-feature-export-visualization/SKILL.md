---
name: featureml-feature-export-visualization
description: Use when PeakBot detection is complete and detected features plus background
  (wall/background) annotations must be written to featureML/TSV and visually spot-checked.
when_to_use_negative:
- Input is already a peak/feature table or featureML output — re-running detection
  and export would not change it.
- The task is training a new CNN model rather than exporting detections from an existing
  model (use the PeakBot_Example train.py workflow with reference Peaks/Walls/Backgrounds
  TSVs instead).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakBot
  role: Run the CNN detection on local maxima and export detections as featureML and
    TSV files; render example detections to images.
  repo: https://github.com/christophuv/PeakBot
- name: PeakBot_Example
  role: Shows how to group detected features (group.py) into a data matrix and a featureML
    file using KNNalignFeatures and groupFeatures.
  repo: https://github.com/christophuv/PeakBot_Example
- name: Python 3.8
  role: Runtime environment in which PeakBot detection and export scripts (detect.py,
    group.py) are executed.
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow 2.5.0
  role: Executes the CNN model that classifies local maxima and outputs peak-type,
    -center, and -bounding-box used in the exported features.
  repo: https://github.com/christophuv/PeakBot
- name: Anaconda
  role: Virtual environment manager that installs CUDA dependencies needed to run
    GPU-accelerated detection before export.
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:a20bd35c4d9c2dcd7336e1abdc771ff8f14f0446735aeb1c7239ae163debba32
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
  - PeakBot_Example
  - Python
  - Python 3.8
  - TensorFlow 2.5.0
  - cuDNN
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# featureml-feature-export-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Export PeakBot-detected chromatographic peaks and background annotations as featureML files and tab-separated-value tables, optionally rendering detection examples to images so the user can verify detections before downstream use.

## When to use

Use when PeakBot detection is complete and detected features plus background (wall/background) annotations must be written to featureML/TSV and visually spot-checked.

## When NOT to use

- Input is already a peak/feature table or featureML output — re-running detection and export would not change it.
- The task is training a new CNN model rather than exporting detections from an existing model (use the PeakBot_Example train.py workflow with reference Peaks/Walls/Backgrounds TSVs instead).

## Inputs

- Profile-mode LC-HRMS chromatogram files (background/blank or sample) already preprocessed with peakbot.cuda.preProcessChromatogram
- A trained PeakBot CNN model (Python 3.8, TensorFlow 2.5.0, CUDA/cuDNN GPU environment via Anaconda yml)
- LC-HRMS run settings: noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM

## Outputs

- featureML file of grouped detected features
- Tab-separated-values file of detected features and background (wall) annotations
- Rendered images of example detections for user verification
- Aligned feature data matrix across samples

## How to apply

After running PeakBot's CNN model on profile-mode LC-HRMS chromatograms, group detected features across samples with peakbot.cuda.KNNalignFeatures and peakbot.cuda.groupFeatures to produce an aligned data matrix, and export the detections as featureML files and tab-separated-values files. When the run was on a background sample, the exported annotations include the background classes PeakBot learned (walls — signals present throughout large parts of the chromatograms — versus true chromatographic peaks), which can later be compiled into an msms_exclusion_list.tsv. For each detection type, render example detections to images for user verification; the CNN outputs a peak-type, -center, and -bounding-box, so images should show a plausible bounding box around the apex. Control the run's search space via PeakBot's minimal settings: noiseLevel, minIntensity, minRT/maxRT (e.g. exclude waste and column reconditioning regions), RTpeakWidth, and intraScanMaxAdjacentSignalDifferencePPM/interScanMaxSimilarSignalDifferencePPM, which can be estimated with the provided estimateParameters.py script.

## Related tools

- **PeakBot** (Run the CNN detection on local maxima and export detections as featureML and TSV files; render example detections to images.) — https://github.com/christophuv/PeakBot
- **PeakBot_Example** (Shows how to group detected features (group.py) into a data matrix and a featureML file using KNNalignFeatures and groupFeatures.) — https://github.com/christophuv/PeakBot_Example
- **Python 3.8** (Runtime environment in which PeakBot detection and export scripts (detect.py, group.py) are executed.) — https://github.com/christophuv/PeakBot
- **TensorFlow 2.5.0** (Executes the CNN model that classifies local maxima and outputs peak-type, -center, and -bounding-box used in the exported features.) — https://github.com/christophuv/PeakBot
- **Anaconda** (Virtual environment manager that installs CUDA dependencies needed to run GPU-accelerated detection before export.) — https://github.com/christophuv/PeakBot

## Examples

```
python detect.py; python group.py  # peakbot.cuda.preProcessChromatogram + runPeakBot, then KNNalignFeatures/groupFeatures to write featureML & TSV
```

## Evaluation signals

- Exported TSV contains detected features with peak-type, peak-center and bounding-box consistent with the CNN output; background rows distinguish walls from true chromatographic peaks.
- featureML file loads in standard metabolomics viewers and its feature list matches the aligned data-matrix rows from peakbot.cuda.groupFeatures.
- Rendered example-detection images show a bounding box and apex centered on a plausible chromatographic peak; wall detections span the large RT regions they were annotated over (e.g. Rtstart/Rtend style extents).
- No detections appear in excluded regions: scans earlier than minRT or later than maxRT, or below noiseLevel/minIntensity, are absent from the export.
- For an MS/MS exclusion-list follow-up, the compiled msms_exclusion_list.tsv built from these wall annotations is non-empty and matches the exported wall TSV entries.

## Limitations

- Detection quality depends on run settings such as intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM, which are described as a bit difficult to evaluate; estimateParameters.py is provided to help.
- Walls are signals present throughout the entire or large parts of the chromatograms; unusually narrow or shifted background structures may not be captured unless narrow RT windows are specified in the walls reference list.
- Data referenced by the PeakBot_Example README is not hosted, so reproducing the example exports requires the user's own LC-HRMS chromatograms and reference lists.

## Evidence

- [results] Export background detections as featureML and TSV: [evidence span withheld — anchor status: not_found]
- [readme] group.py produces data matrix and featureML: [evidence span withheld — anchor status: not_found]
- [readme] Minimal settings to set before detection: [evidence span withheld — anchor status: not_found]
- [abstract] CNN output includes bounding-box and center: [evidence span withheld — anchor status: not_found]
