---
name: smrt-fingerprint-data-loading
description: Use when a task needs the METLIN SMRT retention time table with ECFP
  fingerprints loaded and randomly split 75% training / 25% validation before training
  or benchmarking an RT prediction model.
when_to_use_negative:
- The data are already loaded as a disjoint train/validation feature matrix — re-splitting
  would break comparability with the published 75/25 evaluation
- The task is a downstream comparison (e.g., Tanimoto-thresholded k-NN vs. DLM error
  testing), which consumes the outputs of this skill rather than performing the loading
  itself
- The input dataset is not the METLIN SMRT RT dataset (e.g., MS/MS spectra or other
  chromatographic modalities), since SMRT covers reverse-phase chromatography retention
  times specifically
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: Keras
  role: Consumes the 75/25 split of SMRT RT values and ECFP fingerprints as training
    and validation inputs for the deep-learning RT regression model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:8e781f077cdb4e44d80aa482dc04452e0443daf99598d4cae11092b0c4596ad5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# smrt-fingerprint-data-loading

## Summary

Ingests the METLIN SMRT dataset — experimentally acquired reverse-phase chromatography retention times (in seconds) for up to 80,038 small molecules paired with ECFP fingerprints — and randomly partitions molecules into a 75% training / 25% validation split for downstream RT-prediction modeling.

## When to use

Use when a task needs the METLIN SMRT retention time table with ECFP fingerprints loaded and randomly split 75% training / 25% validation before training or benchmarking an RT prediction model.

## When NOT to use

- The data are already loaded as a disjoint train/validation feature matrix — re-splitting would break comparability with the published 75/25 evaluation
- The task is a downstream comparison (e.g., Tanimoto-thresholded k-NN vs. DLM error testing), which consumes the outputs of this skill rather than performing the loading itself
- The input dataset is not the METLIN SMRT RT dataset (e.g., MS/MS spectra or other chromatographic modalities), since SMRT covers reverse-phase chromatography retention times specifically

## Inputs

- METLIN SMRT dataset table with retention times in seconds for up to 80,038 small molecules from the METLIN library
- Per-molecule ECFP fingerprint feature block aligned to the RT rows

## Outputs

- Training-set dataframe/feature matrix (75% of molecules: RT in seconds + ECFP fingerprints)
- Validation-set dataframe/feature matrix (25% of molecules: RT in seconds + ECFP fingerprints)

## How to apply

Load the METLIN SMRT dataset containing RT values recorded in seconds plus per-molecule ECFP fingerprints, exactly as described in the Domingo-Almenara et al. workflow. Perform a random molecule-level split reserving 75% of molecules for training and 25% for validation; the split is by molecule, not by feature, so every molecule's RT and fingerprint stay together. Preserve the seconds unit convention for RT, since downstream relative-error comparisons (e.g., naive k-NN median-RT baseline vs. the DLM) are computed against experimental RT. The resulting training partition feeds Keras model training (4 fully connected hidden layers of 1000, 500, 200, 100 units, relu, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch size 35) while the validation partition is used for prediction and for Tanimoto-similarity thresholding against training molecules.

## Related tools

- **Keras** (Consumes the 75/25 split of SMRT RT values and ECFP fingerprints as training and validation inputs for the deep-learning RT regression model)

## Examples

```
import pandas as pd; df = pd.read_csv('metlin_smrt_rt_ecfp.csv'); train = df.sample(frac=0.75, random_state=0); val = df.drop(train.index)
```

## Evaluation signals

- Training/validation partition sizes match the 75%/25% molecule-level split with no molecule appearing in both partitions
- RT values retain the seconds unit convention expected by the article's error calculations
- Every retained molecule carries both an RT value and a complete ECFP fingerprint vector (row-wise alignment intact)
- Molecule count does not exceed the 80,038 small molecules reported for the SMRT dataset
- Downstream reproduction check: the split supports recovering the published significance pattern (P < 0.0001 at 90/80/70/50% Tanimoto thresholds; not significant at 95%)

## Limitations

- The split is random at the molecule level, so reproducing the exact paper comparison requires the same randomization procedure; stochastic splits will perturb threshold-group membership and hence Wilcoxon P values
- SMRT retention times were acquired under reverse-phase chromatography, so the loaded data do not by themselves guarantee transfer to other chromatographic methods
- The dataset's availability was motivated by a historical gap — the lack of publicly available datasets to train ML models — so no substitute dataset should be swapped in without re-validating the benchmark

## Evidence

- [abstract] SMRT dataset scope and RT provenance: "Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules."
- [intro] Dataset source library: "a large-scale dataset consisting of experimentally acquired chromatographic RT covering 80,038 small molecules from the METLIN library"
- [other] Loading and split procedure from the surfaced workflow: [evidence span withheld — anchor status: not_found]
