---
name: retention-time-database-construction
description: Use when you need a reference set of predicted retention times for known
  metabolites (e.g. HMDB entries) to pair with m/z filtering and RT-based candidate
  ranking in untargeted metabolomics where MS/MS spectra are unavailable.
when_to_use_negative:
- You already have experimental RTs for the molecules of interest — the projection
  step works directly on measured values and no SMRT-trained prediction is needed
- Fragmentation spectra are available and confident structural identification (not
  just m/z + RT annotation) is possible — full MS/MS identification supersedes RT-only
  scoring
- Your chromatography is not reversed-phase LC-MS similar to the SMRT/PredRet collection
  — the trained model's RT scale and the projection prior will not transfer
edam_operation: http://edamontology.org/operation_0417
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3520
tools:
- name: cmmrt
  role: Python package implementing RT prediction and the fingerprint/build_data helpers;
    installed via `make install` and driven through the Makefile rules (train_predictor,
    test_predictor)
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the 2,214 fingerprints (MACCS166, ECFP, PFP) and 5,666 descriptors
    that are the model's input features; required (under license) before training
    or predicting
- name: RDKit
  role: Alternative fingerprint backend for a lighter-weight reproduction via train_with_rdkit.ipynb,
    training a DNN on SMRT using RDKit fingerprints
- name: Python
  role: Runtime for the cmmrt package, training scripts and notebooks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:07bb3ea0f31d8a30625d33d341dd04ac300fe98e462c393b93afa285e8a53785
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_0417
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# retention-time-database-construction

## Summary

Builds a predicted retention-time (RT) database by computing ECFP, MACCSFP and PFP fingerprints with alvaDesc for 6,823 HMDB molecules and scoring their RTs with a DNN regressor trained on the 80,038-RT SMRT dataset, to support downstream metabolite annotation by m/z + RT.

## When to use

Use when you need a reference set of predicted retention times for known metabolites (e.g. HMDB entries) to pair with m/z filtering and RT-based candidate ranking in untargeted metabolomics where MS/MS spectra are unavailable.

## When NOT to use

- You already have experimental RTs for the molecules of interest — the projection step works directly on measured values and no SMRT-trained prediction is needed
- Fragmentation spectra are available and confident structural identification (not just m/z + RT annotation) is possible — full MS/MS identification supersedes RT-only scoring
- Your chromatography is not reversed-phase LC-MS similar to the SMRT/PredRet collection — the trained model's RT scale and the projection prior will not transfer

## Inputs

- SMILES, mol, SDF, mol2 or hin structure files for the compounds to annotate (e.g. the 6,823 HMDB molecules with KEGG numbers)
- METLIN SMRT dataset: 80,038 experimental RTs from reversed-phase LC-MS (training data)
- CSV input files mapping pubchem id to InChI or SMILES (build_data_smrt.py / build_data_cmm.py examples)
- alvaDesc-generated ECFP, MACCSFP and PFP fingerprint strings (and optionally descriptors)

## Outputs

- A predicted-RT database: RT predictions for each candidate molecule (e.g. all 6,823 HMDB/KEGG compounds)
- Trained RT predictor model saved to the saved_models folder (blender plus base regressors)
- Feature-preprocessing results: median-imputed, zero-variance- and correlation-filtered descriptor/fingerprint matrix
- Hyperparameter-search logs in results/optuna/*.db and cross-validation summaries in results/rt/rt_cv.csv

## How to apply

Compute 5,666 descriptors and 2,214 fingerprints (MACCS166, ECFP, PFP) for the target compound set (SMILES, mol, SDF, mol2 or hin input) using the licensed alvaDesc software; when alvaDesc cannot emit some descriptor, standardize and apply median imputation with a missing indicator, drop zero-variance features and features correlated >0.9. Train the RT regressor (best model: heavily regularized DNN with cosine annealing warm restarts and stochastic weight averaging) on the 80,038 experimental RTs from the METLIN SMRT dataset, including non-retained molecules (RT < 5 min) with a quantile transformation applied to RTs before fitting. Run the trained model over the 6,823 molecules carrying KEGG numbers in HMDB to produce the predicted-RT database, applying the log(1+x) transform and robust median/IQR scaling (factor 0.741) if the database will feed the meta-learned GP projection stage. Use the `paper` branch of the cmmrt repository for exact reproducibility; the Makefile rules run smoke tests by default and must have the --smoke_test flag removed for full training.

## Related tools

- **cmmrt** (Python package implementing RT prediction and the fingerprint/build_data helpers; installed via `make install` and driven through the Makefile rules (train_predictor, test_predictor)) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the 2,214 fingerprints (MACCS166, ECFP, PFP) and 5,666 descriptors that are the model's input features; required (under license) before training or predicting)
- **RDKit** (Alternative fingerprint backend for a lighter-weight reproduction via train_with_rdkit.ipynb, training a DNN on SMRT using RDKit fingerprints)
- **Python** (Runtime for the cmmrt package, training scripts and notebooks)

## Examples

```
python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models  # remove --smoke_test for full training
```

## Evaluation signals

- Best DNN achieves mean absolute error 39.2±1.2 s and median absolute error 17.2±0.9 s on the SMRT dataset — predictions within these ranges indicate a correctly trained predictor
- Fingerprint-only and fingerprints+descriptors variants should outperform descriptor-only features, per the paper's observation that 'fingerprints tend to perform better'
- Non-retained molecules (RT < 5 min) must be included in training/evaluation, unlike the original SMRT evaluation which excluded them
- Before projection, database RTs must be log(1+x)-transformed and robustly scaled with median/IQR (factor 0.741) into the [0,1]/[-1,1] range required by the GP software
- Reproduction should use branch `paper` and complete training (remove --smoke_test); results land in results/rt/rt_cv.csv and results/optuna/*.db

## Limitations

- alvaDesc is commercial software (under license); without it, fingerprints for new compounds cannot be generated as in the paper (RDKit notebook is a partial workaround)
- The Makefile ships with a --smoke_test line that trains on a subset of regressors and data — full training requires editing the Makefile
- Models trained on SMRT reversed-phase RTs do not directly give experimental RTs for a new chromatographic method; they must pass through the meta-learned GP projection stage
- Branch `paper` should be used as reference for reproducing results since the codebase continues to evolve toward CEU Mass Mediator integration

## Evidence

- [intro] Steps 1-2: a RTs database is created using a predictive model trained on the SMRT dataset.: "Steps 1-2: a RTs database is created using a predictive model trained on the SMRT dataset."
- [methods] HMDB database scope: "for the 6,823 molecules with KEGG number in the Human Metabolome DataBase (HMDB)"
- [methods] Feature preprocessing: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
- [methods] Non-retained definition: "a molecule was considered non-retained if its Retention Time (RT) was smaller than 5 minutes."
- [readme] Fingerprint generation prerequisite: "To train your own model or to predict the RT of your own set of compounds it is necessary to generate the fingerprints using alvaDesc software"
