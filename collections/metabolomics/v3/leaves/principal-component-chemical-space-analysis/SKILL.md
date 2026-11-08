---
name: principal-component-chemical-space-analysis
description: Use when ML retention-time models must be shown to generalize across
  compound libraries, by checking that training, test, and validation sets (e.g. HILIC
  880/143/377 and RPLC 398/96/358 compounds) co-occupy chemical space.
when_to_use_negative:
- When the question is retention-time prediction accuracy itself (MAE, error windows)
  rather than structural representativeness of the training set
- When compounds are outside the chemical landscape of the training set — the paper
  warns 'if Retip is used for predictions outside the chemical landscape of the training
  set, results may be more erroneous', so PCA overlap is a screening step, not a fix
- When the compound class (e.g. complex high molecular weight lipids such as triacylglycerols
  or phosphatidylcholines) was excluded from training and cannot legitimately be assessed
  for overlap
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0091
tools:
- name: Retip
  role: R package providing getCD() descriptor calculation, cesc()/proc.data() preprocessing,
    and chem.space(db_rt, target, title) PCA chemical-space plotting
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: R CDK descriptor engine used to compute the 2D molecular descriptors feeding
    the PCA (rcdk 3.4.7.1 in the paper; 3.8.1 in Retip 2.0)
- name: ggplot2
  role: Graphics backend for the Retip chem.space PCA projection plots
- name: R
  role: Runtime environment for the Retip workflow (R 4.4.0 for Retip 2.0; RStudio
    recommended)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:8f2d9c1505e7852b4f666f243802b3f0f1801f257028329baff667c4ad700474
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
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# principal-component-chemical-space-analysis

## Summary

Assesses structural overlap between retention-time training, test, and validation compound sets by running PCA on standardized 2D CDK molecular descriptors, using Retip's chem.space() visualization of chemical space per chromatographic method (HILIC and RPLC).

## When to use

Use when ML retention-time models must be shown to generalize across compound libraries, by checking that training, test, and validation sets (e.g. HILIC 880/143/377 and RPLC 398/96/358 compounds) co-occupy chemical space.

## When NOT to use

- When the question is retention-time prediction accuracy itself (MAE, error windows) rather than structural representativeness of the training set
- When compounds are outside the chemical landscape of the training set — the paper warns 'if Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous', so PCA overlap is a screening step, not a fix
- When the compound class (e.g. complex high molecular weight lipids such as triacylglycerols or phosphatidylcholines) was excluded from training and cannot legitimately be assessed for overlap

## Inputs

- SMILES structures with RT metadata for HILIC training/test/validation sets (MoNA/MassBank, Metabolomics Workbench, RIKEN sources)
- SMILES structures with RT metadata for RPLC PlaSMA training/test/validation sets
- 2D CDK descriptor matrix computed per compound (286 descriptors via getCD())

## Outputs

- Per-method PCA score plots (ggplot2) superimposing training and test/validation compounds in chemical space
- Cumulative proportion of variance explained by the first two principal components per chromatographic method (reported: 41% HILIC, 69% RPLC)
- Qualitative overlap assessment (e.g. validation set slightly outside HILIC training space; RP validation co-distributed with training)

## How to apply

Load each chromatographic method's compound sets (HILIC: 880 training / 143 test / 377 Pathogen Box validation; RPLC PlaSMA: 398 training / 96 test / 358 validation) with SMILES structures, standardizing structures by removing salts and metal-containing compounds. Compute 2D CDK descriptors per compound (286 descriptors via Retip's getCD() using rcdk; the paper used rcdk 3.4.7.1, current README pins rcdk 3.8.1), dropping compounds whose descriptor calculation fails. Center and scale descriptor columns with Retip's cesc()/proc.data() handling so non-existent values and low-variance columns do not distort PCA inputs. Run PCA on the combined descriptor matrix per method using Retip's chem.space(db_rt, target, title) to superimpose training vs. external validation compounds in ggplot2-based PCA plots. Record cumulative variance explained by the first two PCs and inspect whether the validation set (drug-like Pathogen Box for HILIC; Ophiorrhiza pumila true positives for RPLC) overlaps the training cloud. The rationale is that models trained on molecular descriptors may mispredict for compounds outside the training chemical landscape.

## Related tools

- **Retip** (R package providing getCD() descriptor calculation, cesc()/proc.data() preprocessing, and chem.space(db_rt, target, title) PCA chemical-space plotting) — https://github.com/olobion/Retip
- **rcdk** (R CDK descriptor engine used to compute the 2D molecular descriptors feeding the PCA (rcdk 3.4.7.1 in the paper; 3.8.1 in Retip 2.0))
- **ggplot2** (Graphics backend for the Retip chem.space PCA projection plots)
- **R** (Runtime environment for the Retip workflow (R 4.4.0 for Retip 2.0; RStudio recommended))

## Examples

```
library(Retip); chem.space(dbRT, TARGET, "Chemical space HILIC")
```

## Evaluation signals

- First-two-PC cumulative variance reproduces reported values within tolerance: 41% (HILIC) and 69% (RPLC), ±5 percentage points
- PCA projection plots show HILIC validation (Pathogen Box) slightly outside, and RP validation (Ophiorrhiza pumila true positives) co-distributed with, the training cloud
- Descriptor matrices are centered and scaled; non-existent descriptor values and low-variance columns were removed before PCA
- Compounds failing CDK descriptor calculation are dropped and salt/metal-containing structures removed prior to descriptor computation
- Plots cover both chromatographic methods separately (HILIC and RPLC) with all three compound sets labeled

## Limitations

- Predictions for compounds outside the chemical landscape of the training set may be more erroneous, so limited overlap warns of degraded RT prediction rather than invalidating the PCA itself
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity
- Many chemicals in test and validation sets were not well represented by the training models
- Molecular descriptors alone may not fully explain complex LC interactions (buffer ionic strength, static aqueous phases, adsorption/desorption), so descriptor-based chemical space is an approximation

## Evidence

- [readme] tutorial in the Retip app as well as in the examples folder: [evidence span withheld — anchor status: not_found]
