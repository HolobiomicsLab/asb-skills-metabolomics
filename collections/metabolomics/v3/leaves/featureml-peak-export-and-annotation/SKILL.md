---
name: featureml-peak-export-and-annotation
description: Use when PeakBot peak-picking output from LC-HRMS profile-mode chromatograms
  must be exported as a tsv peak-area table and featureML files for inspection/annotation
  in TOPPView, or as images for user verification.
when_to_use_negative:
- Input is already a grouped feature table or peak-area matrix — go straight to downstream
  statistics instead of re-running grouping/export
- Peaks have not yet been detected with a trained PeakBot CNN model — training (train.py)
  and detection (detect.py) must precede grouping (group.py)
- The goal is de novo peak picking on centroided or non-LC-HRMS data — PeakBot requires
  profile-mode LC-HRMS chromatograms
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakBot
  role: Provides the grouping/export functions (peakbot.cuda.KNNalignFeatures, peakbot.cuda.groupFeatures)
    that turn CNN-detected peaks into a data matrix and featureML file
  repo: https://github.com/christophuv/PeakBot
- name: TOPPView (OpenMS)
  role: Used to view the exported featureML files and to interactively inspect m/z
    differences of signals in ppm
- name: Python
  role: Environment in which the group.py export workflow runs; pandas loads the exported
    tsv peak-area table for downstream per-feature statistics
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:c952443b4895c0111af75d32de1bf3066d68da0ecc7beef1d887fc1e7098432d
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
  - TOPPView (OpenMS)
  - cuDNN
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45
n_claims: 20
---

# featureml-peak-export-and-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports chromatographic peaks detected by PeakBot's CNN (with peak-type, peak-center, and bounding-box) as a tab-separated peak-area table and as featureML files that can be viewed and annotated in TOPPView from the OpenMS toolbox.

## When to use

Use when PeakBot peak-picking output from LC-HRMS profile-mode chromatograms must be exported as a tsv peak-area table and featureML files for inspection/annotation in TOPPView, or as images for user verification.

## When NOT to use

- Input is already a grouped feature table or peak-area matrix — go straight to downstream statistics instead of re-running grouping/export
- Peaks have not yet been detected with a trained PeakBot CNN model — training (train.py) and detection (detect.py) must precede grouping (group.py)
- The goal is de novo peak picking on centroided or non-LC-HRMS data — PeakBot requires profile-mode LC-HRMS chromatograms

## Inputs

- Profile-mode LC-HRMS chromatograms already processed by PeakBot peak detection (CNN outputs peak-type, peak-center, bounding-box per local maximum)
- PeakBot model detections from multiple samples to be aligned and grouped
- LC-HRMS property settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, ppm tolerances) as required by the PeakBot scripts

## Outputs

- Tab-separated-values peak-area table (e.g. wheatear_peak_areas.tsv) with peak areas per feature per sample
- featureML files viewable in TOPPView (OpenMS toolbox)
- Grouped feature data matrix spanning several samples
- Example detection images for user verification

## How to apply

After running PeakBot peak detection (local-maxima extraction into standardized rt x m/z areas followed by CNN classification), use the group.py workflow to align and merge detections across samples: peakbot.cuda.KNNalignFeatures aligns detected features with a k-nearest-neighbor approach and peakbot.cuda.groupFeatures calculates the groups into a data matrix. Export the grouped peak areas as a tab-separated-values file (e.g. wheatear_peak_areas.tsv) and as featureML files, which are directly viewable in TOPPView from the OpenMS toolbox. Optionally export example detections to images so a user can visually verify that bounding-boxes and peak-centers were placed correctly. The resulting tsv peak-area table is the input for downstream per-feature statistics such as replicate RSD computed with pandas.

## Related tools

- **PeakBot** (Provides the grouping/export functions (peakbot.cuda.KNNalignFeatures, peakbot.cuda.groupFeatures) that turn CNN-detected peaks into a data matrix and featureML file) — https://github.com/christophuv/PeakBot
- **TOPPView (OpenMS)** (Used to view the exported featureML files and to interactively inspect m/z differences of signals in ppm)
- **Python** (Environment in which the group.py export workflow runs; pandas loads the exported tsv peak-area table for downstream per-feature statistics) — https://github.com/christophuv/PeakBot

## Examples

```
python group.py
```

## Evaluation signals

- The exported featureML files open without errors in TOPPView from the OpenMS toolbox
- The tsv peak-area table parses cleanly (e.g. with pandas) and contains a peak area for every feature x sample combination
- After KNN alignment, each detected peak is matched to exactly one feature group across samples (no orphan or duplicated groups)
- Exported example detection images show bounding-boxes and peak-centers correctly placed on true chromatographic peaks, not on walls/background
- Downstream validation: per-feature replicate RSD of exported peak areas below 10% on average, with at least 90% of features under 20% RSD (WheatEar technical-replicate criterion)

## Limitations

- featureML/data-matrix export requires the grouping step (KNN alignment across samples); it cannot be produced from a single ungrouped detection run
- The CNN outputs a suggested bounding-box and peak-center, which are approximations and should be visually verified via the exported example images
- The example data referenced by the PeakBot_Example repository is not hosted in the repository (the data folder is not hosted), so exports must be reproduced with user-supplied chromatograms

## Evidence

- [readme] This script shows how detected chromatographic peaks can be grouped into a data matrix and a featureML file.: [evidence span withheld — anchor status: external_doc]
- [readme] TOPPView from the OpenMS toolbox (https://pubmed.ncbi.nlm.nih.gov/19425593/) can be used as it allows the user to easily calculate the m/z difference in ppm: [evidence span withheld — anchor status: external_doc]
