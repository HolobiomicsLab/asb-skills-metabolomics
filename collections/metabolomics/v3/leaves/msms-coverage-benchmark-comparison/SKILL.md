---
name: msms-coverage-benchmark-comparison
description: Use when LC-HRMS/MS workflow needs to measure how many true chromatographic
  peaks receive MS/MS coverage before and after applying an exclusion list generated
  from background (wall) annotations, and the result must be compared against published
  baseline counts (Supplementary Table S2).
when_to_use_negative:
- The exclusion list has not yet been generated from PeakBot wall/background annotations
  (this skill benchmarks coverage, not list creation).
- Data are not LC-HRMS/MS acquisitions in both negative and positive ionization modes
  with paired with/without-exclusion conditions.
- No published or ground-truth MS/MS coverage counts exist to compare the reproduced
  numbers against (e.g. exploratory study without Supplementary Table S2-equivalent
  values).
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakBot
  role: Annotates background (wall) signals in LC-HRMS background/blank chromatograms,
    whose detections are compiled into the msms_exclusion_list.tsv used to boost MS/MS
    coverage in subsequent measurements.
  repo: https://github.com/christophuv/PeakBot
- name: Python 3.8
  role: Runtime language in which PeakBot and the benchmark computation are implemented.
- name: TensorFlow 2.5.0
  role: Runs PeakBot's CNN classification of local maxima as true peaks versus walls/backgrounds,
    which feeds the exclusion list.
- name: Anaconda
  role: Recommended virtual-environment manager for installing PeakBot with CUDA dependencies
    in a reproducible setup.
- name: CUDA toolkit
  role: GPU support required to run PeakBot's CNN on Nvidia graphics cards.
- name: cuDNN
  role: GPU deep-learning library required alongside CUDA toolkit for PeakBot CNN
    execution.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:46f5381284141d997efb10c61083545d986517e2cf88b0e32b6cf65fd72cf971
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
  - Python 3.8
  - TensorFlow 2.5.0
  - cuDNN
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# msms-coverage-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies MS/MS coverage of true chromatographic peaks with and without an MS/MS exclusion list derived from PeakBot-detected wall/background signals, in negative and positive ionization modes, benchmarking the counts against reported reference values.

## When to use

Use when LC-HRMS/MS workflow needs to measure how many true chromatographic peaks receive MS/MS coverage before and after applying an exclusion list generated from background (wall) annotations, and the result must be compared against published baseline counts (Supplementary Table S2).

## When NOT to use

- The exclusion list has not yet been generated from PeakBot wall/background annotations (this skill benchmarks coverage, not list creation).
- Data are not LC-HRMS/MS acquisitions in both negative and positive ionization modes with paired with/without-exclusion conditions.
- No published or ground-truth MS/MS coverage counts exist to compare the reproduced numbers against (e.g. exploratory study without Supplementary Table S2-equivalent values).

## Inputs

- LC-HRMS profile-mode chromatogram files (background/blank samples) annotated by PeakBot
- PeakBot output featureML files and tab-separated-values files containing wall/background signal annotations
- msms_exclusion_list.tsv — tab-separated exclusion list compiled from PeakBot wall detections
- LC-MS/MS raw chromatograms acquired in negative and positive ionization modes, with and without the exclusion list active
- Reference peak/feature list identifying true chromatographic peaks (ground-truth or PeakBot-detected features)
- Supplementary Table S2 reference values for expected MS/MS coverage counts (negative mode 1356→1605; positive mode 855→1810)

## Outputs

- MS/MS coverage count of true chromatographic peaks per ionization mode, without exclusion list
- MS/MS coverage count of true chromatographic peaks per ionization mode, with exclusion list
- Absolute and relative (%) coverage gain per ionization mode
- Comparison against Supplementary Table S2 baseline values to validate reproduction

## How to apply

Compile the wall/background signal annotations from PeakBot (classified as 'walls' — signals present throughout large parts of the chromatograms) into an MS/MS exclusion list saved as msms_exclusion_list.tsv. Run the LC-HRMS/MS acquisition twice for the same samples: once without the exclusion list and once with it loaded, measuring MS/MS coverage of true chromatographic peaks in each condition. Count how many true chromatographic peaks acquired MS/MS spectra in each ionization mode separately. Compare the reproduced counts against the published reference values from Supplementary Table S2: negative mode baseline 1356 versus 1605 with exclusion list (+18%); positive mode baseline 855 versus 1810 with exclusion list (+112%). The relative increase percentage is computed as (coverage_with_list − coverage_without_list) / coverage_without_list × 100, so the decision point is whether the observed gain is consistent with the expected mode-dependent magnitude.

## Related tools

- **PeakBot** (Annotates background (wall) signals in LC-HRMS background/blank chromatograms, whose detections are compiled into the msms_exclusion_list.tsv used to boost MS/MS coverage in subsequent measurements.) — https://github.com/christophuv/PeakBot
- **Python 3.8** (Runtime language in which PeakBot and the benchmark computation are implemented.)
- **TensorFlow 2.5.0** (Runs PeakBot's CNN classification of local maxima as true peaks versus walls/backgrounds, which feeds the exclusion list.)
- **Anaconda** (Recommended virtual-environment manager for installing PeakBot with CUDA dependencies in a reproducible setup.)
- **CUDA toolkit** (GPU support required to run PeakBot's CNN on Nvidia graphics cards.)
- **cuDNN** (GPU deep-learning library required alongside CUDA toolkit for PeakBot CNN execution.)

## Examples

```
conda create -f Windows10_python38_withGPU.yml; python detect.py  # run PeakBot to annotate walls, then compile msms_exclusion_list.tsv and re-run LC-MS/MS to compare coverage
```

## Evaluation signals

- Reproduced MS/MS coverage count in negative mode without list matches the published baseline of 1356 true chromatographic peaks.
- Reproduced MS/MS coverage count in negative mode with exclusion list matches the published value of 1605 (≈ +18% gain).
- Reproduced MS/MS coverage count in positive mode without list matches the published baseline of 855 true chromatographic peaks.
- Reproduced MS/MS coverage count in positive mode with exclusion list matches the published value of 1810 (≈ +112% gain).
- The compiled msms_exclusion_list.tsv contains only wall/background entries annotated by PeakBot, not true chromatographic peaks.

## Limitations

- The coverage gain is ionization-mode dependent: negative mode improved only 18% while positive mode improved 112%, so extrapolating a single gain factor across modes is invalid.
- The exclusion list must be derived specifically from PeakBot-annotated walls (background signals present throughout large parts of the chromatograms); arbitrary exclusion lists will not reproduce these numbers.
- PeakBot requires a CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN installed, or the benchmark cannot be run in the recommended GPU-accelerated configuration.
- Coverage counts reflect the specific LC-HRMS/MS setup used in the article; they are reproducible only when acquisition parameters and reference feature lists match the original study.

## Evidence

- [results] Using an MS/MS exclusion list generated from PeakBot-detected walls increased MS/MS coverage of true chromatographic peaks from 1356 to 1605 (negative mode) and 855 to 1810 (positive mode), increases of 18% and 112%.: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation: reproduced counts match Supplementary Table S2.: [evidence span withheld — anchor status: not_found]
- [readme] peakbot.runPeakBot is used to classify standardized areas as peaks or backgrounds.: [evidence span withheld — anchor status: external_doc]
