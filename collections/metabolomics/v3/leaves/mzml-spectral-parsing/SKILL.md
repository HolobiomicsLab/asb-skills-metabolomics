---
name: mzml-spectral-parsing
description: Use when starting from raw untargeted LC/MS runs in mzML format and you
  need a zero-parameter MS1 signal-detection entry point that feeds downstream drift
  correction and feature-table generation.
when_to_use_negative:
- Input is already a feature table or intensity matrix rather than raw mzML files
  — mzLearn's signal-detection core is unnecessary; use the downstream mzEmbed VAE
  pretraining/fine-tuning modules instead
- Downstream tasks such as de-isotoping, annotation, or MS/MS compound identification
  are the goal — mzLearn outputs are limited to the ion MS1 level and do not perform
  these steps; use downstream workflows
- Analysis targets LC modes other than HILIC positive mode (e.g. the current pre-trained
  model was developed specifically for HILIC positive mode)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzLearn
  role: Data-driven MS1 signal-detection and alignment tool that runs from mzML files
    without user-set parameters; handles the mzML parsing and iterative signal-characteristic
    learning stage
  repo: http://mzlearn.com/
- name: mzEmbed
  role: Downstream framework that consumes the mzLearn feature output (2,736-feature
    matrices) for VAE pretraining and fine-tuning; depends on this mzML parsing/detection
    stage
  repo: https://github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:6e165fb60b2c888f8d1f12c4db0b61c67ae56ade4d99da5f93ba266a08d8501b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mzml-spectral-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses raw LC/MS data in the open-source mzML format as the entry point of the mzLearn pipeline, which then iteratively learns signal characteristics to detect metabolite signals without user-set parameters.

## When to use

Use when starting from raw untargeted LC/MS runs in mzML format and you need a zero-parameter MS1 signal-detection entry point that feeds downstream drift correction and feature-table generation.

## When NOT to use

- Input is already a feature table or intensity matrix rather than raw mzML files — mzLearn's signal-detection core is unnecessary; use the downstream mzEmbed VAE pretraining/fine-tuning modules instead
- Downstream tasks such as de-isotoping, annotation, or MS/MS compound identification are the goal — mzLearn outputs are limited to the ion MS1 level and do not perform these steps; use downstream workflows
- Analysis targets LC modes other than HILIC positive mode (e.g. the current pre-trained model was developed specifically for HILIC positive mode)

## Inputs

- Raw LC/MS data files in the open-source mzML format

## Outputs

- A two-dimensional table of detected features defined by median rt and m/z values, with normalized intensities across samples
- Feature matrices as CSV of floats, shape [n_samples, 2736] (the robust mzLearn peak set)

## How to apply

Load raw LC/MS data files in the open-source mzML format as the first step of the mzLearn pipeline. mzLearn then autonomously refines signal detection, iteratively learning signal characteristics to separate true peaks from noise and correcting for retention time (rt) and intensity drifts caused by batch effects and run order. The parsing and detection stage requires no prior knowledge or QC samples; all thresholds are learned from the data itself. The output is a two-dimensional table of detected features defined by median rt and m/z values with normalized intensities across samples. This mzLearn core is the prerequisite for downstream mzEmbed steps such as VAE pretraining on the resulting 2,736-feature robust peak set.

## Related tools

- **mzLearn** (Data-driven MS1 signal-detection and alignment tool that runs from mzML files without user-set parameters; handles the mzML parsing and iterative signal-characteristic learning stage) — http://mzlearn.com/
- **mzEmbed** (Downstream framework that consumes the mzLearn feature output (2,736-feature matrices) for VAE pretraining and fine-tuning; depends on this mzML parsing/detection stage) — https://github.com/ReviveMed/mzEmbed

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Output must be a feature table whose columns are median rt and m/z with normalized intensities across samples
- Detected signal counts should be in the range observed across 15 public datasets (mzLearn: 11,442 signals on average vs 7,100 for XCMS and 4,655 for ASARI) as a qualitative benchmark
- No user-set parameters or external QC samples should be required; all thresholds are learned internally and iteratively
- Downstream mzEmbed consumption requires CSV float matrices with shape [n_samples, 2736] and pandas.read_csv(index_col=0) semantics
- Retention-time (rt) and intensity drifts caused by batch effects and run order should be corrected within the output (validated against mzLearn's drift-correction claims)

## Limitations

- Iterative learning increases computational time (up to 2.5 days on a single machine); parallel processing on multiple compute nodes is not yet implemented
- mzLearn results are at the ion MS1 level; it does not perform de-isotoping or MS/MS identification
- The current pre-trained model downstream was developed specifically for HILIC positive mode; other LC modes and modalities are not covered
- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, which the algorithm must compensate for data-drivenly

## Evidence

- [readme] mzML format requirement: [evidence span withheld — anchor status: external_doc]
- [readme] Output table definition: [evidence span withheld — anchor status: external_doc]
- [readme] Iterative drift correction: [evidence span withheld — anchor status: external_doc]
- [readme] Downstream mzEmbed input shape: [evidence span withheld — anchor status: external_doc]
