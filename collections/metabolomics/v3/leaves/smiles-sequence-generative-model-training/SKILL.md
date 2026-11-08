---
name: smiles-sequence-generative-model-training
description: Reference for training a fingerprint-to-SMILES encoder–decoder generative
  model (as in MSNovelist) from SMILES-only chemical databases, with 10-fold cross-validation
  on GPU.
when_to_use_negative:
- The goal is annotating compounds already present in a compound database — use database
  search (CSI:FingerID) rather than training a generative model.
- Input is an annotated MS2 spectral library training set (e.g. GNPS/MoNA spectra
  with spectral annotations) — this skill specifically trains on SMILES-only compound
  structures to remain independent of spectral libraries.
- Only negative-mode MS2 spectra will be analyzed — the model is trained for positive-mode
  data.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3047
tools:
- name: MSNovelist
  role: Encoder–decoder RNN framework trained to generate SMILES structure sequences
    from fingerprints under formula constraints; provided as a Docker/Singularity
    container (msnovelist6 image).
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS 6
  role: Upstream fingerprint prediction system; MSNovelist was retrained on the new
    data from SIRIUS 6, and training weights were uploaded to S3 for the SIRIUS 6
    backend.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:49c21a6ac1806f6a8fb6a67d40eab6b64a32e6e4ecc74dba0ba0cc185de1053c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - MSNovelist
  - SIRIUS 6
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3047
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# smiles-sequence-generative-model-training

## Summary

Train an encoder–decoder RNN that generates SMILES structure strings from predicted molecular fingerprints and formula constraints, with training data drawn from compound databases rather than spectral libraries so structure generation stays library-independent.

## When to use

Reference for training a fingerprint-to-SMILES encoder–decoder generative model (as in MSNovelist) from SMILES-only chemical databases, with 10-fold cross-validation on GPU.

## When NOT to use

- The goal is annotating compounds already present in a compound database — use database search (CSI:FingerID) rather than training a generative model.
- Input is an annotated MS2 spectral library training set (e.g. GNPS/MoNA spectra with spectral annotations) — this skill specifically trains on SMILES-only compound structures to remain independent of spectral libraries.
- Only negative-mode MS2 spectra will be analyzed — the model is trained for positive-mode data.

## Inputs

- SMILES sequences of ~1.23M chemical structures from HMDB 4.0, COCONUT, DSSTox
- 14,047 predicted fingerprints for fingerprint-simulation parametrization
- dataset-s6-202311 S3 dataset (sirius-novelist)
- MSNovelist Docker/Singularity image (stravsm/msnovelist6)
- GPU allocation for training jobs

## Outputs

- Trained encoder–decoder RNN weights for SMILES generation
- 10-fold cross-validated model weights under MSNovelist-results/weights
- Trained weights set ready for upload to S3 (e.g. s3://sirius-novelist)

## How to apply

Compile the training set from SMILES-only compound databases — MSNovelist used 1,232,184 structures from HMDB 4.0, COCONUT and DSSTox, together with 14,047 predicted fingerprints used to parametrize fingerprint simulation with added error, deliberately excluding MS2 spectra and spectral libraries. Set up the MSNovelist container environment: build the msnovelist6 Singularity image on a job node (builds fail on login nodes) with SINGULARITY_CACHEDIR pointed at /cluster/scratch, download the training dataset s3://sirius-novelist/dataset-s6-202311 with an AWS CLI Singularity image into $TMPDIR, then wire paths via a .env file defining DATA_LOC, SIF_LOC, CODE_LOC (e.g. MSNovelist-private), and RESULTS_LOC. Write a training script (train.py) that consumes the dataset and trains the encoder–decoder RNN to predict structures as SMILES sequences from the fingerprint input under constraints imposed by the molecular formula; submit with sbatch run_singularity.sh using run_train.sh, one GPU per job. Submit 'sbatch --array=0-9' so train.py independently trains each of 10 cross-validation folds. Correctness is judged by all 10 training array jobs completing and a trained weights directory (e.g. m-36719628-msnovelist-sirius6-weights) appearing under MSNovelist-results/weights.

## Related tools

- **MSNovelist** (Encoder–decoder RNN framework trained to generate SMILES structure sequences from fingerprints under formula constraints; provided as a Docker/Singularity container (msnovelist6 image).) — https://github.com/meowcat/MSNovelist
- **SIRIUS 6** (Upstream fingerprint prediction system; MSNovelist was retrained on the new data from SIRIUS 6, and training weights were uploaded to S3 for the SIRIUS 6 backend.)

## Examples

```
sbatch --array=0-9 run_singularity.sh run_train.sh
```

## Evaluation signals

- All 10 'sbatch --array=0-9' training jobs and all 10 evaluation jobs complete successfully.
- A trained weights directory is present under MSNovelist-results/weights ready for upload to S3.
- Held-out fold evaluation yields retrieval performance comparable to reported benchmarks (25% ranked first / 45% retrieved on GNPS; 26% ranked first / 57% on CASMI 2016).
- Generated SMILES decode to valid structures consistent with the formula constraint xM.
- Fingerprint input includes simulated error from the 14,047 predicted fingerprints used to parametrize fingerprint simulation.

## Limitations

- The RNN generation module remains independent of spectral libraries, which limits coverage to chemistry represented in the SMILES training databases (HMDB 4.0, COCONUT, DSSTox).
- MSNovelist is currently only trained for positive-mode data; negative-mode performance is expected to be lower.
- Model may be unable to discover chemistry extremely different from known molecules, as chemical space is conservative.
- Formula determination error rates rise to >50% for compounds up to m/z 800 on individual spectra, propagating into generation quality.
- Processing requires a minimum of three fragment ions per MS2 spectrum.

## Evidence

- [results] RNN training data composition: [evidence span withheld — anchor status: not_found]
- [results] Fingerprint simulation parametrization: [evidence span withheld — anchor status: not_found]
- [abstract] SMILES sequence generation objective: "we trained an encoder–decoder RNN model to predict structures in the form of a SMILES sequence from the fingerprint xF under the constraints imposed by the formula xM"
- [readme] Training workflow on cluster: [evidence span withheld — anchor status: not_found]
- [readme] 10-fold cross-validation: [evidence span withheld — anchor status: not_found]
- [abstract] Rationale for library-independent training: "MSNovelist, which combines fingerprint prediction with an encoder–decoder neural network to generate structures de novo solely from tandem mass spectrometry (MS2) spectra"
