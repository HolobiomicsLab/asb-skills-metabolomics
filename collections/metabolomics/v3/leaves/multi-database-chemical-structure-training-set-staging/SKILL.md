---
name: multi-database-chemical-structure-training-set-staging
description: Use when training a structure-generative model (e.g. MSNovelist RNN)
  that must consume a large merged chemical-structure corpus independent of spectral
  libraries.
when_to_use_negative:
- The input is already a processed spectral library (e.g. GNPS or MoNA spectra) rather
  than a chemical-structure corpus — MSNovelist training is deliberately independent
  of spectral libraries
- You only need to run inference/prediction on new MS2 spectra with the released SIRIUS
  6 GUI or the prebuilt Docker container, not retrain the RNN
- 'Working on the login node only: the Singularity build and AWS dataset download
  ''freak out on login node'' and must run on job nodes'
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: MSNovelist
  role: Encoder–decoder RNN framework whose training corpus is being staged; the staged
    structures and fingerprints feed train.py for the 10-fold cross-validation
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: 'Downstream integration target: MSNovelist was integrated into SIRIUS 6, and
    the staged dataset (dataset-s6-202311) reflects retraining on the SIRIUS 6 data'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:f2ca155f3c6ecec824444d5e2bc4a6dc9026ceb2f7bbd34712ee2b1f2c57a5e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - MSNovelist
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# multi-database-chemical-structure-training-set-staging

## Summary

Stages the 1,232,184-structure training corpus for the MSNovelist encoder–decoder RNN by compiling chemical structures from HMDB 4.0, COCONUT and DSSTox plus 14,047 predicted fingerprints, via an AWS S3 dataset and Singularity workflow on an Euler HPC cluster.

## When to use

Use when training a structure-generative model (e.g. MSNovelist RNN) that must consume a large merged chemical-structure corpus independent of spectral libraries.

## When NOT to use

- The input is already a processed spectral library (e.g. GNPS or MoNA spectra) rather than a chemical-structure corpus — MSNovelist training is deliberately independent of spectral libraries
- You only need to run inference/prediction on new MS2 spectra with the released SIRIUS 6 GUI or the prebuilt Docker container, not retrain the RNN
- Working on the login node only: the Singularity build and AWS dataset download 'freak out on login node' and must run on job nodes

## Inputs

- AWS S3 bucket dataset (s3://sirius-novelist/dataset-s6-202311) containing merged HMDB 4.0 / COCONUT / DSSTox structures
- Predicted fingerprints (14,047) used to parametrize fingerprint simulation with input error
- MSNovelist git repository (MSNovelist-private code location)
- Euler HPC scratch space at /cluster/scratch

## Outputs

- Staged training dataset directory under /cluster/scratch wired via DATA_LOC
- Staged msnovelist.sif Singularity container image under SIF_LOC
- Configured .env file pointing to DATA_LOC, SIF_LOC, CODE_LOC, RESULTS_LOC
- Per-fold RNN training workspace for 10 cross-validation folds under RESULTS_LOC

## How to apply

Compile the training set from three structural databases — HMDB (4.0), COCONUT and DSSTox — yielding 1,232,184 chemical structures, and additionally gather 14,047 predicted fingerprints used to parametrize fingerprint simulation (adding error to the RNN input). On an Euler HPC cluster, checkout the MSNovelist git repository on the login node (it fails on job nodes), then build the msnovelist6 Singularity image on a job node with SINGULARITY_CACHEDIR pointed at /cluster/scratch. Build the AWS CLI Singularity image on a job node and use it to download the dataset s3://sirius-novelist/dataset-s6-202311 into $TMPDIR, copying it to /cluster/scratch. Write a .env file defining DATA_LOC, SIF_LOC, CODE_LOC, and RESULTS_LOC to wire the staged dataset, container image, code, and results directories together before launching train.py in the container for the 10 cross-validation folds.

## Related tools

- **MSNovelist** (Encoder–decoder RNN framework whose training corpus is being staged; the staged structures and fingerprints feed train.py for the 10-fold cross-validation) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Downstream integration target: MSNovelist was integrated into SIRIUS 6, and the staged dataset (dataset-s6-202311) reflects retraining on the SIRIUS 6 data)

## Examples

```
sbatch --array=0-9 run_singularity.sh run_train.sh  # stages DATA_LOC/SIF_LOC via .env then trains the 10 MSNovelist RNN folds
```

## Evaluation signals

- The staged structure corpus contains 1,232,184 chemical structures compiled from HMDB 4.0, COCONUT and DSSTox
- The fingerprint-simulation corpus contains 14,047 predicted fingerprints
- A trained weights directory (e.g. m-36719628-msnovelist-sirius6-weights) appears under MSNovelist-results/weights after training
- All 10 training array jobs (sbatch --array=0-9) and 10 evaluation array jobs complete successfully
- The .env file resolves DATA_LOC, SIF_LOC, CODE_LOC and RESULTS_LOC to existing paths under /cluster/scratch

## Limitations

- The released meowcat/MSNovelist repo backend relied on an old SIRIUS version that is no longer running; direct use requires retraining with different fingerprint data (e.g. the 'mist' branch work on Morgan 4096-bit fingerprints was not completed)
- MSNovelist is trained only for positive-mode MS2 data; negative-mode performance is expected to be lower
- Processing requires a minimum of three fragment ions per MS2 spectrum
- The model may be unable to discover chemistry very different from known molecules because its chemical space is conservative

## Evidence

- [methods] RNN model trained on 1,232,184 chemical structures compiled from HMDB (4.0), COCONUT and DSSTox, together with 14,047 predicted fingerprints: "the RNN model was trained on a dataset of 1,232,184 chemical structures compiled from the databases HMDB (4.0)29, COCONUT30 and DSSTox31"
- [methods] 14,047 predicted fingerprints parametrize fingerprint simulation with input error: [evidence span withheld — anchor status: not_found]
- [methods] Staging workflow: git checkout on login node, Singularity build on job node with SINGULARITY_CACHEDIR at /cluster/scratch: "Checkout git repo on login node. Seemingly doesn't work on job node."
- [methods] AWS CLI container downloads s3://sirius-novelist/dataset-s6-202311 into $TMPDIR: "singularity build $SCRATCH_PATH/MSNovelist-image/msnovelist.sif docker://stravsm/msnovelist6"
- [readme] README notes SIRIUS 6 integration and the obsolete old-backend repo: "with the **release of SIRIUS 6, MSNovelist was integrated into SIRIUS**. You can now use MSNovelist de novo structure suggestions directly with the SIRIUS GUI"
