---
name: mzml-to-npy-conversion
description: Use when raw untargeted LC-MS metabolomics data arrive as mzML files
  and the goal is end-to-end DeepMSProfiler analysis, which requires .npy spectral
  arrays plus a datalist mapping each spectrum to its disease/cancer-type label.
when_to_use_negative:
- Input is already in .npy format (the demo data are delivered as .npy, so conversion
  is unnecessary and files can go straight to run_train)
- The workflow requires annotated metabolite tables or peak-picked feature matrices
  — DeepMSProfiler is designed to bypass conventional peak extraction, alignment,
  and annotation, not to produce them
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0102
tools:
- name: DeepMSProfiler
  role: Provides the automatic mzML-to-npy conversion script and the downstream run_train/run_predict/run_feature
    functions that consume the converted arrays
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Liquid chromatography mass spectrometry (LC-MS)
  role: Instrument platform producing the raw mzML metabolomic signals that are converted
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:9921d26a97ef1c9bd5878d1f8b1109e2305f1d066c5c0ca7505540d8f55e79d7
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
  - Liquid chromatography mass spectrometry (LC-MS)
  - PIUMet
  - Python (>= 3.6)
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# mzml-to-npy-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic conversion of raw LC-MS metabolomics files in mzML format into .npy array files so they can be ingested end-to-end by DeepMSProfiler's DenseNet121 ensemble without any peak picking, alignment, or annotation steps.

## When to use

Use when raw untargeted LC-MS metabolomics data arrive as mzML files and the goal is end-to-end DeepMSProfiler analysis, which requires .npy spectral arrays plus a datalist mapping each spectrum to its disease/cancer-type label.

## When NOT to use

- Input is already in .npy format (the demo data are delivered as .npy, so conversion is unnecessary and files can go straight to run_train)
- The workflow requires annotated metabolite tables or peak-picked feature matrices — DeepMSProfiler is designed to bypass conventional peak extraction, alignment, and annotation, not to produce them

## Inputs

- Raw LC-MS metabolomics files in .mzML format (e.g. CCLE lipidomics of 928 cell lines, MetaboLights ST001142, or the MetaboLights MTBLS1129 colon cancer dataset)
- Label/datalist file mapping each spectrum file to its disease or cancer-type label (e.g. ../example/label.txt or datalist.txt with FilePath, Label, Dataset columns)

## Outputs

- Per-sample .npy spectral arrays readable by DeepMSProfiler's run_train/run_predict/run_feature pipeline
- Input directory and datalist prepared for DenseNet121 ensemble training (optionally pre-trained model weights if obtained from the author)

## How to apply

Install DeepMSProfiler (pip install DeepMSProfiler, or git clone the yjdeng9/DeepMSProfiler repo and run install_dependencies.sh) under Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1, with CUDA 10.1 or a CPU-only environment. Rather than manually converting spectra, point the -data argument at a directory containing mzML files; DeepMSProfiler's script automatically converts each mzML file to .npy format so the tensors can be fed directly to run_train (e.g. DenseNet121, adam optimizer, lr 1e-04, 10 runs). The converted .npy files are keyed through a datalist/label file (e.g. example/datalist.txt with FilePath, Label, Dataset columns) mapping each spectrum to its class such as cancer/health/nodule or one of 23 CCLE cancer types. This conversion is what enables the paper's end-to-end claim: raw LC-MS signals enter the model with no conventional peak extraction, identification, or annotation, so unannotated metabolite signals are retained for downstream RISE saliency analysis.

## Related tools

- **DeepMSProfiler** (Provides the automatic mzML-to-npy conversion script and the downstream run_train/run_predict/run_feature functions that consume the converted arrays) — https://github.com/yjdeng9/DeepMSProfiler
- **Liquid chromatography mass spectrometry (LC-MS)** (Instrument platform producing the raw mzML metabolomic signals that are converted)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_train -run_pred -run_feature
```

## Evaluation signals

- Every input mzML file has a corresponding .npy file in the data directory before run_train starts
- The datalist/label file enumerates all converted files with correct labels (demo log shows a 27-row table of C/H/N .npy files with train/test Dataset splits)
- Converted arrays reach the model without intermediate peak-extraction artifacts — the pipeline log goes directly from file loading to '[INFO] run model with args: DenseNet121 adam lr 1.0e-04 0'
- Downstream training completes on the converted data (demo: 10 DenseNet121 runs then 'Step2 Train Done'), confirming the arrays are well-formed for the network

## Limitations

- The conversion is coupled to DeepMSProfiler's own input conventions (datalist plus data directory); it is not a general-purpose mzML converter like msconvert
- Pre-trained model weights are not bundled — access requires contacting the author (<maintainer e-mail, see the repository README>), so converted data alone cannot reproduce the published 96.1%-accuracy lung adenocarcinoma prediction out of the box
- Demo heatmaps are fixed-size arrays (e.g. shape (6, 341, 341)), so spectra from unusual acquisition settings may need checking for compatibility with the expected tensor dimensions

## Evidence
