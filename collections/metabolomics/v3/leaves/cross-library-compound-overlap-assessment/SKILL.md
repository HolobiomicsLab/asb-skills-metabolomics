---
name: cross-library-compound-overlap-assessment
description: Use when you must decide whether ML models trained on molecular descriptors
  (e.g., CDK descriptors for retention-time prediction) will generalize to a test
  or validation library with a different chemical profile.
when_to_use_negative:
- Compounds lie outside the training chemical landscape by design (e.g., complex high-molecular-weight
  lipids such as triacylglycerols or phosphatidylcholines were excluded from training,
  so overlap conclusions would not apply to them)
- The goal is model building or error estimation (MAE), not structural generalizability
  screening — this skill is a diagnostic precursor, not a prediction workflow
- Descriptors cannot be computed for a large fraction of compounds (e.g., salts or
  metal-containing structures not first standardized), making the PCA basis unrepresentative
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: Retip
  role: Provides chem.space(db_rt, target, title) for PCA chemical-space superposition
    plots and cesc()/proc.data() for descriptor centering/scaling and low-variance
    filtering
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: Computes the 286 2D CDK molecular descriptors per compound (original paper
    used rcdk 3.4.7.1)
- name: ggplot2
  role: Renders the PCA chemical-space projection plots comparing training, test,
    and validation libraries
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:69df62a81f1977deb31c4f086dbd8f0fb92d93ab910ff69bf7c39383afcb2c83
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - R
  - Retip
  - ggplot2
  - rcdk
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# cross-library-compound-overlap-assessment

## Summary

Assesses whether training, test, and external validation compound sets (e.g., HILIC 880/143/377 and RPLC PlaSMA 398/96/358 compounds) occupy overlapping chemical space by running PCA on standardized 2D CDK molecular descriptors, so that retention-time prediction models can be judged generalizable before being built.

## When to use

Use when you must decide whether ML models trained on molecular descriptors (e.g., CDK descriptors for retention-time prediction) will generalize to a test or validation library with a different chemical profile.

## When NOT to use

- Compounds lie outside the training chemical landscape by design (e.g., complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines were excluded from training, so overlap conclusions would not apply to them)
- The goal is model building or error estimation (MAE), not structural generalizability screening — this skill is a diagnostic precursor, not a prediction workflow
- Descriptors cannot be computed for a large fraction of compounds (e.g., salts or metal-containing structures not first standardized), making the PCA basis unrepresentative

## Inputs

- SMILES structures with set labels (train/test/validation) for each chromatographic method (HILIC from MoNA/MassBank, metabolomicsworkbench, RIKEN; RPLC from PlaSMA)
- Molecular descriptor matrix (286 2D CDK descriptors per compound, computed via Retip getCD())

## Outputs

- PCA projection plots (ggplot2) superimposing training and test/validation compounds per chromatographic method
- Cumulative proportion of variance explained by the first two principal components per method (reported: 41% HILIC, 69% RPLC)
- Qualitative overlap verdict (e.g., validation set slightly outside HILIC training space; RP validation co-distributed with training)

## How to apply

Load each chromatographic method's compound sets (HILIC: 880 training, 143 test, 377 Pathogen Box validation; RPLC: 398 training, 96 test, 358 validation) with SMILES, removing salts and metal-containing compounds. Compute 286 2D CDK descriptors per compound with Retip's getCD() function (rcdk), dropping compounds whose descriptor calculation fails. Center and scale descriptor columns (Retip's cesc()/proc.data() also handles non-existent values and low-variance columns) so PCA inputs are comparable across libraries. Run PCA per chromatographic method on the combined descriptor matrix using Retip's chem.space(db_rt, target, title) to superimpose training and test/validation libraries in PCA space (ggplot2 plots). Record the cumulative variance explained by the first two PCs per method and visually inspect whether validation compounds fall inside or outside the training chemical landscape.

## Related tools

- **Retip** (Provides chem.space(db_rt, target, title) for PCA chemical-space superposition plots and cesc()/proc.data() for descriptor centering/scaling and low-variance filtering) — https://github.com/olobion/Retip
- **rcdk** (Computes the 286 2D CDK molecular descriptors per compound (original paper used rcdk 3.4.7.1))
- **ggplot2** (Renders the PCA chemical-space projection plots comparing training, test, and validation libraries)

## Examples

```
devtools::install_github('olobion/Retip'); chem.space(db_rt, 'HILIC', 'chemical_space_overlap')
```

## Evaluation signals

- Cumulative variance explained by the first two PCs reproduces the reported values within tolerance: 41% for HILIC and 69% for RPLC (±5 percentage points)
- PCA projection plots show the described overlap pattern: Pathogen Box validation compounds slightly outside the HILIC training space, RPLC Ophiorrhiza pumila validation true positives co-distributed with training
- Compounds failing CDK descriptor calculation are dropped and salts/metal-containing structures removed before PCA
- Descriptor columns are centered and scaled (and non-existent / low-variance columns handled) before PCA; unscaled raw descriptors are a red flag

## Limitations

- If Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous — PCA overlap only flags, and does not remove, this risk
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity; there are still many chemicals not well represented by the training models
- Visual inspection of PCA overlap is partly qualitative; the reported first-two-PC variance totals (41%/69%) leave substantial unexplained variance

## Evidence

- [other] Compute 286 2D CDK descriptors per compound with Retip's getCD() function: [evidence span withheld — anchor status: not_found]
