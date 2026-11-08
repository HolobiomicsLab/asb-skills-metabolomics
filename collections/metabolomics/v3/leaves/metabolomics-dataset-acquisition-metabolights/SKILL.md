---
name: metabolomics-dataset-acquisition-metabolights
description: Use when a study requires an external, publicly archived LC-MS metabolomics
  cohort (raw mzML files with disease labels) to train or benchmark a classifier,
  and the dataset of interest has a MetaboLights accession ID.
when_to_use_negative:
- The data is already available locally as feature tables or .npy arrays (e.g. the
  bundled demo data or Baidu Netdisk example files) — no archive download is needed
- The target dataset is not archived in MetaboLights (e.g. CCLE lipid metabolomic
  data or the 859-sample pre-trained serum cohort, which requires contacting the author
  for access)
- Only prediction or feature extraction on an existing trained model is required —
  use run_predict/run_feature directly rather than re-acquiring the cohort
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DeepMSProfiler
  role: Consumes the downloaded mzML files and label file; auto-converts mzML to .npy
    and reads the datalist/label paths during run_train
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: MetaboLights
  role: Public repository from which the raw colon-cancer LC-MS dataset (MTBLS1129,
    236 samples) is obtained
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:8aa35dcbf146258602d2881cd929aa0e13aff112592c274f3933e9b121744eba
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
  - MetaboLights
  - Python (>= 3.6)
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolomics-dataset-acquisition-metabolights

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Acquire public LC-MS metabolomics data from the MetaboLights repository (e.g. colon-cancer dataset MTBLS1129, 236 samples) and format a sample label file with disease-type labels and a train/test split, ready for downstream deep-learning training.

## When to use

Use when a study requires an external, publicly archived LC-MS metabolomics cohort (raw mzML files with disease labels) to train or benchmark a classifier, and the dataset of interest has a MetaboLights accession ID.

## When NOT to use

- The data is already available locally as feature tables or .npy arrays (e.g. the bundled demo data or Baidu Netdisk example files) — no archive download is needed
- The target dataset is not archived in MetaboLights (e.g. CCLE lipid metabolomic data or the 859-sample pre-trained serum cohort, which requires contacting the author for access)
- Only prediction or feature extraction on an existing trained model is required — use run_predict/run_feature directly rather than re-acquiring the cohort

## Inputs

- MetaboLights accession ID (e.g. MTBLS1129) for a public LC-MS metabolomics dataset
- Raw LC-MS mzML files from the downloaded cohort (236 samples for MTBLS1129)
- Sample disease-type label file assigning labels and a 4:1 discovery/testing split

## Outputs

- Local copy of raw mzML files organized in a data directory
- Formatted sample label file (filepath, disease label, train/test assignment)
- Auto-converted .npy spectral arrays ready for model training

## How to apply

Identify the MetaboLights accession for the target cohort — in this workflow the colon-cancer LC-MS dataset with ID MTBLS1129 containing 236 samples — and download the raw mzML files. Construct a sample label file mapping each file (e.g. C01.npy-style identifiers after conversion) to a disease-type label and a discovery/testing split; this workflow uses a 4:1 discovery-to-testing split. Place the raw mzML files in the tool's data directory so the pipeline auto-converts them to .npy format before training. Verify the downloaded cohort size and label distribution before proceeding, since the independent testing set is what reproduces the reported metrics (accuracy 97.9%, precision 98.7%, recall 93.4%, F1 95.8% on MTBLS1129).

## Related tools

- **DeepMSProfiler** (Consumes the downloaded mzML files and label file; auto-converts mzML to .npy and reads the datalist/label paths during run_train) — https://github.com/yjdeng9/DeepMSProfiler
- **MetaboLights** (Public repository from which the raw colon-cancer LC-MS dataset (MTBLS1129, 236 samples) is obtained)

## Examples

```
run_train(datalist_path='DeepMSProfiler/example/datalist.txt', data_dir='DeepMSProfiler/example/data', job_dir='DeepMSProfiler/example/out/jobs007', epoch=2)
```

## Evaluation signals

- Downloaded cohort matches the published size (236 samples for MTBLS1129)
- Label file assigns a disease-type label and train/test split to every sample, with a 4:1 discovery-to-testing ratio
- All mzML files are auto-converted to .npy format without errors before training begins
- Downstream test metrics on the independent split land near the published values: accuracy 97.9%, precision 98.7%, recall 93.4%, F1 95.8% (small run-to-run variation allowed)

## Limitations

- Only datasets deposited in MetaboLights can be acquired this way; the pre-trained 859-sample serum model is not public and requires contacting the author (<maintainer e-mail, see the repository README>)
- Run-to-run variation means test metrics will only approximately reproduce the reported values
- The demo/example data distributed via Baidu Netdisk is a separate, smaller subset and is not equivalent to the full MTBLS1129 cohort

## Evidence

- [results] Re-trained model metrics on the MTBLS1129 independent test set that the acquired data must reproduce: [evidence span withheld — anchor status: not_found]
