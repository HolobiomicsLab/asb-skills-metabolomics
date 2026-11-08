---
name: msms-exclusion-list-generation
description: Use when PeakBot has annotated walls/background signals in blank or background
  LC-HRMS samples and you need a targeted DDA exclusion list to boost MS/MS coverage
  of true chromatographic peaks in subsequent runs.
when_to_use_negative:
- The task is standard peak picking/feature detection only and no MS/MS acquisition
  is planned — use PeakBot's detection/grouping workflow (detect.py, group.py) without
  compiling an exclusion list.
- Chromatograms are already reduced to a feature table or peak list without profile-mode
  data — PeakBot operates on profile-mode LC-HRMS data (rt × m/z local maxima), so
  no wall annotations can be extracted.
- No blank/background samples are available, or walls are absent from the sample background
  — the exclusion list is derived from PeakBot's wall-type background annotations,
  so it cannot be built.
- If you need MS/MS exclusion lists from sources other than PeakBot wall annotations
  (e.g. in-source ions, reporter ions), this workflow is not designed for that.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: PeakBot
  role: CNN-based peak picker that classifies local maxima as true chromatographic
    peaks vs. background (walls); its wall annotations are the source of the exclusion
    list
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow
  role: Implements the PeakBot CNN model used to classify wall vs. peak local maxima
  repo: https://www.tensorflow.org/
- name: Anaconda
  role: Provides the virtual environment (with CUDA dependencies) in which PeakBot
    is run for wall annotation
- name: CUDA toolkit
  role: GPU prerequisite for fast instance generation and detection on background
    chromatograms
- name: cuDNN
  role: GPU library required alongside CUDA for the TensorFlow CNN inference
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:b2e19aff9fbc9b882f704f7c183a66a07220b8bcc4e9276e89ec479ebdde0933
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
  - TensorFlow
  - cuDNN
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# msms-exclusion-list-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates an MS/MS exclusion list (msms_exclusion_list.tsv) from PeakBot's CNN-detected wall/background annotations of LC-HRMS blank samples, so that co-eluting background signals are excluded from fragmentation and MS/MS coverage of true chromatographic peaks increases. The workflow was validated against the paper's Supplementary Table S2, where coverage rose from 1356 to 1605 (negative mode) and 855 to 1810 (positive mode).

## When to use

Use when PeakBot has annotated walls/background signals in blank or background LC-HRMS samples and you need a targeted DDA exclusion list to boost MS/MS coverage of true chromatographic peaks in subsequent runs.

## When NOT to use

- The task is standard peak picking/feature detection only and no MS/MS acquisition is planned — use PeakBot's detection/grouping workflow (detect.py, group.py) without compiling an exclusion list.
- Chromatograms are already reduced to a feature table or peak list without profile-mode data — PeakBot operates on profile-mode LC-HRMS data (rt × m/z local maxima), so no wall annotations can be extracted.
- No blank/background samples are available, or walls are absent from the sample background — the exclusion list is derived from PeakBot's wall-type background annotations, so it cannot be built.
- If you need MS/MS exclusion lists from sources other than PeakBot wall annotations (e.g. in-source ions, reporter ions), this workflow is not designed for that.

## Inputs

- LC-HRMS profile-mode background/blank chromatograms (e.g. raw mzML/centroided profile data consumed by PeakBot)
- A trained PeakBot CNN model (from peakbot.trainPeakBotModel) or the paper's published model
- LC-HRMS run properties: noiseLevel, minIntensity, minRT, maxRT, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM

## Outputs

- msms_exclusion_list.tsv containing wall/background signals (m/z, retention-time window, m/z deviation)
- PeakBot featureML files and TSV annotations of detected walls/backgrounds
- Rendered images of detection examples for user verification
- MS/MS coverage counts (number of true chromatographic peaks with MS/MS spectra) with vs. without the exclusion list, per ionization mode

## How to apply

Run PeakBot (Python 3.8, TensorFlow 2.5.0) inside an Anaconda environment built from the repository's yml files (e.g. Windows10_python38_withGPU.yml), with a CUDA-enabled Nvidia GPU plus CUDA toolkit and cuDNN when GPU mode is used. Load the LC-HRMS background/blank chromatograms in profile mode and run the trained CNN via peakbot.cuda.preProcessChromatogram and peakbot.runPeakBot to classify each local maximum as true chromatographic peak versus irrelevant background such as walls (signals present throughout large parts of the chromatograms). Export the wall/background annotations as featureML files and tab-separated-values files, optionally rendering detection examples to images for user verification. Compile the wall annotations into msms_exclusion_list.tsv, following the wall tsv schema (columns MZ, Rtstart, Rtend, MZdeviation), and use that list as the exclusion list when acquiring MS/MS on subsequent samples. Finally, compare MS/MS coverage of true peaks with and without the exclusion list in both ionization modes; the paper's benchmark values are 1356→1605 (+18%) negative and 855→1810 (+112%) positive per Supplementary Table S2.

## Related tools

- **PeakBot** (CNN-based peak picker that classifies local maxima as true chromatographic peaks vs. background (walls); its wall annotations are the source of the exclusion list) — https://github.com/christophuv/PeakBot
- **TensorFlow** (Implements the PeakBot CNN model used to classify wall vs. peak local maxima) — https://www.tensorflow.org/
- **Anaconda** (Provides the virtual environment (with CUDA dependencies) in which PeakBot is run for wall annotation)
- **CUDA toolkit** (GPU prerequisite for fast instance generation and detection on background chromatograms)
- **cuDNN** (GPU library required alongside CUDA for the TensorFlow CNN inference)

## Examples

```
conda env create -f Windows10_python38_withGPU.yml; python detect.py  # then peakbot.runPeakBot on blanks, export walls, and compile msms_exclusion_list.tsv
```

## Evaluation signals

- msms_exclusion_list.tsv is produced with rows matching the wall tsv schema (MZ, Rtstart, Rtend, MZdeviation) for each annotated wall.
- FeatureML and TSV wall annotations exported from PeakBot cover all wall-type local maxima classified as background in the blank chromatograms.
- MS/MS coverage of true chromatographic peaks increases relative to the no-exclusion baseline; paper reference values: 1356→1605 (+18%) in negative mode and 855→1810 (+112%) in positive mode (Supplementary Table S2).
- Detection examples rendered to images show walls annotated across large retention-time spans, consistent with the wall definition (signals present throughout the entire or large parts of the chromatograms).
- Run environment resolves to Python 3.8 / TensorFlow 2.5.0 (or the yml-declared equivalents) before detection starts.

## Limitations

- Requires a minimum of 100 reference features to train a CNN model with high-quality peak-picking results before wall annotation can be trusted.
- PeakBot runs on profile-mode LC-HRMS data; centroid-only inputs without local-maxima information are unsuitable.
- GPU is effectively required for practical use: it requires a CUDA-enabled graphics card from Nvidia as well as the CUDA toolkit and cuDNN libraries; a CPU-only Anaconda yml exists but is slower.
- Wall/background tsv columns must be correctly specified (MZ, Rtstart, Rtend, MZdeviation); incorrect Rtstart/Rtend windows will misplace exclusion regions.
- Coverage gains depend on the ionization mode and dataset composition — the reported +18%/+112% gains are specific to the paper's negative/positive mode benchmark.

## Evidence

- [results] Coverage gain claim (source card finding): [evidence span withheld — anchor status: not_found]
- [readme] Workflow steps (README): [evidence span withheld — anchor status: external_doc]
