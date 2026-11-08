---
name: lcms-raw-spectra-preprocessing-to-npy
description: Use when starting from raw LC-MS metabolomics files (e.g. mzML from MetaboLights
  studies such as MTBLS1129) that must be auto-converted to .npy arrays for end-to-end
  deep-learning classification with DeepMSProfiler.
when_to_use_negative:
- Input data are already in .npy format (e.g. the DeepMSProfiler example/demo data)
  — no conversion step is needed, proceed directly to run_train
- Input is a conventional peak-picked, aligned, or annotated metabolomics feature
  table rather than raw mzML spectra — this skill targets raw untargeted signals only
- The goal is downstream modeling (training/prediction/feature extraction) on already-converted
  arrays — use run_train/run_predict/run_feature instead
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DeepMSProfiler
  role: Performs automatic mzML-to-.npy conversion of raw LC-MS spectra and parses
    the label/datalist file into a train/test sample table for downstream training
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Python (>= 3.6)
  role: Runtime environment in which the DeepMSProfiler conversion script and its
    dependencies execute
  repo: https://pypi.org/project/DeepMSProfiler/
- name: TensorFlow 2.2.0
  role: Pinned deep-learning backend whose expected .npy input format the conversion
    step must satisfy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:390b7e0c023a8a5585f6574985f355c48944a5ebfef0fc8b313a26575213175f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - CUDA
  - DeepMSProfiler
  - Keras
  - Python (>= 3.6)
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lcms-raw-spectra-preprocessing-to-npy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts raw LC-MS metabolomics spectra (mzML) into fixed-size NumPy (.npy) image-like arrays that DeepMSProfiler can ingest directly, eliminating manual peak picking, alignment, and annotation. It is the mandatory entry step of the end-to-end DeepMSProfiler workflow.

## When to use

Use when starting from raw LC-MS metabolomics files (e.g. mzML from MetaboLights studies such as MTBLS1129) that must be auto-converted to .npy arrays for end-to-end deep-learning classification with DeepMSProfiler.

## When NOT to use

- Input data are already in .npy format (e.g. the DeepMSProfiler example/demo data) — no conversion step is needed, proceed directly to run_train
- Input is a conventional peak-picked, aligned, or annotated metabolomics feature table rather than raw mzML spectra — this skill targets raw untargeted signals only
- The goal is downstream modeling (training/prediction/feature extraction) on already-converted arrays — use run_train/run_predict/run_feature instead

## Inputs

- Raw LC-MS metabolomics files in mzML format (e.g. 236 samples from MetaboLights MTBLS1129)
- Sample label file assigning disease-type labels and the train/test (discovery/testing) split per sample
- Environment: Python >= 3.6, TensorFlow == 2.2.0, Keras == 2.3.1, CUDA 10.1 (or CPU-only)

## Outputs

- Auto-converted .npy spectral arrays in the data directory, referenced by the datalist (e.g. C01.npy, H01.npy, N01.npy)
- Parsed sample table (FilePath, Label, Dataset) mapping each .npy to its disease label and train/test split

## How to apply

Install DeepMSProfiler (pip install DeepMSProfiler, or git clone plus bash install_dependencies.sh) in an environment with Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1, and CUDA 10.1 (CPU-only also works). Prepare a sample label file that assigns disease-type labels and the discovery/testing split (e.g. 4:1) to each file, and point -data at the directory of raw mzML files; the script auto-converts mzML to .npy format on the fly. For each dataset (e.g. the 236-sample MTBLS1129 colon-cancer LC-MS dataset), the datalist/label file maps each converted .npy to a label and a train/test designation, as in the demo log where C01.npy–N09.npy are marked 'train' or 'test'. No peak extraction, identification, or alignment is performed by design — the model consumes the raw converted spectra directly. Decision point: if data are already .npy (e.g. the Baidu Netdisk demo data), skip conversion and pass the directory straight to run_train.

## Related tools

- **DeepMSProfiler** (Performs automatic mzML-to-.npy conversion of raw LC-MS spectra and parses the label/datalist file into a train/test sample table for downstream training) — https://github.com/yjdeng9/DeepMSProfiler
- **Python (>= 3.6)** (Runtime environment in which the DeepMSProfiler conversion script and its dependencies execute) — https://pypi.org/project/DeepMSProfiler/
- **TensorFlow 2.2.0** (Pinned deep-learning backend whose expected .npy input format the conversion step must satisfy)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_train -run_pred -run_feature
```

## Evaluation signals

- Each raw mzML file in the -data directory has a corresponding .npy file after the conversion step (no orphan mzML files left unprocessed)
- The parsed sample table prints FilePath/Label/Dataset columns matching the label file, with every .npy assigned to 'train' or 'test' (as in the demo log rows C01.npy … N09.npy)
- Converted arrays are loadable in the pinned environment (TensorFlow 2.2.0 / Keras 2.3.1) and reach training without shape errors; the demo pipeline proceeds to produce heatmaps of shape (6, 341, 341)
- Training with run_train launches 10 independent DenseNet121 runs (adam, lr 1.0e-04) on the converted discovery split without I/O errors
- Downstream test metrics on MTBLS1129 (accuracy 97.9%, precision 98.7%, recall 93.4%, F1 95.8%, allowing small run-to-run variation) confirm the converted inputs were correctly formatted

## Limitations

- The automatic conversion supports mzML input; data already in other vendor formats must be converted to mzML beforehand
- Requires a pinned legacy stack (Python >= 3.6, TensorFlow == 2.2.0, Keras == 2.3.1, CUDA 10.1); newer TensorFlow versions are not specified as compatible
- Only a datalist/label file separates discovery vs testing samples; mislabeling the split in the label file silently changes the evaluation

## Evidence
