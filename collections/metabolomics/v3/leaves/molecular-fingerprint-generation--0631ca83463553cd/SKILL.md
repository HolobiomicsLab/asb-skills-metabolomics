---
name: molecular-fingerprint-generation
description: Use when small-molecule structures (SMILES, SDF, mol, mol2, hin, InChI-referenced
  CSVs) must be turned into fingerprint/descriptor feature matrices to train or apply
  ML regressors, e.g. for retention-time prediction.
when_to_use_negative:
- Input data is already a fingerprint/descriptor feature table (e.g. a precomputed
  alvaDesc export) — skip generation and go straight to preprocessing/training
- An alvaDesc license is unavailable — fall back to the RDKit path (train_with_rdkit.ipynb)
  instead of attempting alvaDesc generation
- No machine-learning step follows and only a small number of candidate molecules
  need simple mass-based lookup, e.g. an accurate mass search at 10 ppm against HMDB
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3316
tools:
- name: alvaDesc
  role: Licensed software used to generate the 5,666 molecular descriptors and 2,214
    fingerprints (ECFP, MACCSFP, PFP) from each compound structure
  repo: https://www.alvascience.com/alvadesc/
- name: cmmrt
  role: Python package providing build_data_cmm.py / build_data_smrt.py with generate_vector_fingerprints()
    and generate_vector_fps_descs() to drive alvaDesc and produce the fingerprint
    strings
  repo: https://github.com/constantino-garcia/cmmrt
- name: RDKit
  role: Alternative fingerprint source for training a DNN on SMRT without alvaDesc,
    as illustrated in the train_with_rdkit.ipynb notebook
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:721e3f74a0e98525a5abedf884ceb0f976a62c64f3928540f21b1e219eaa0327
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# molecular-fingerprint-generation

## Summary

Generate ECFP, MACCSFP and PFP fingerprints (plus optional molecular descriptors) for small-molecule structures using alvaDesc, producing the fixed-length feature vectors consumed by RT-prediction models such as the SMRT-trained DNN in the cmmrt pipeline.

## When to use

Use when small-molecule structures (SMILES, SDF, mol, mol2, hin, InChI-referenced CSVs) must be turned into fingerprint/descriptor feature matrices to train or apply ML regressors, e.g. for retention-time prediction.

## When NOT to use

- Input data is already a fingerprint/descriptor feature table (e.g. a precomputed alvaDesc export) — skip generation and go straight to preprocessing/training
- An alvaDesc license is unavailable — fall back to the RDKit path (train_with_rdkit.ipynb) instead of attempting alvaDesc generation
- No machine-learning step follows and only a small number of candidate molecules need simple mass-based lookup, e.g. an accurate mass search at 10 ppm against HMDB

## Inputs

- Chemical structure files in SMILES, mol, SDF, mol2 or hin format
- CSV files with PubChem IDs plus SMILES or InChI (as in build_data_cmm.py / build_data_smrt.py)
- SMILES strings passed directly via the smiles parameter of generate_vector_fingerprints()

## Outputs

- Per-molecule string of ECFP, MACCSFP and PFP fingerprint values sequentially joined (2,214 fingerprint features total)
- Descriptor + fingerprint feature table (5,666 descriptors + 2,214 fingerprints) for model training
- Feature matrix preprocessed with standardization, median imputation with missing indicators, zero-variance removal, and correlation (>0.9) filtering

## How to apply

Process each compound structure with alvaDesc via the cmmrt build_data scripts: generate_vector_fingerprints() returns a single string of ECFP, MACCSFP and PFP fingerprint values sequentially joined, while generate_vector_fps_descs() additionally emits 5,666 molecular descriptors alongside the 2,214 fingerprint bits. The reference application (build_data_cmm.py, build_data_smrt.py) reads a CSV keyed by PubChem ID with either SMILES or InChI and resolves each entry to an SDF structure before fingerprinting. Before model fitting, standardize descriptors, apply median imputation with a missing indicator when alvaDesc cannot generate a descriptor, remove zero-variance features and features with correlation >0.9. Prefer fingerprints-only features when choosing between feature blocks, since results in the paper suggest 'fingerprints tend to perform better' (the best model was a heavily regularized fingerprint DNN reaching 39.2±1.2 s MAE on SMRT).

## Related tools

- **alvaDesc** (Licensed software used to generate the 5,666 molecular descriptors and 2,214 fingerprints (ECFP, MACCSFP, PFP) from each compound structure) — https://www.alvascience.com/alvadesc/
- **cmmrt** (Python package providing build_data_cmm.py / build_data_smrt.py with generate_vector_fingerprints() and generate_vector_fps_descs() to drive alvaDesc and produce the fingerprint strings) — https://github.com/constantino-garcia/cmmrt
- **RDKit** (Alternative fingerprint source for training a DNN on SMRT without alvaDesc, as illustrated in the train_with_rdkit.ipynb notebook) — https://github.com/constantino-garcia/cmmrt

## Examples

```
from cmmrt.rt.build_data_cmm import generate_vector_fingerprints; fps = generate_vector_fingerprints(aDesc, smiles='CC(=O)O')
```

## Evaluation signals

- Each output fingerprint string concatenates exactly the three types ECFP, MACCSFP and PFP, totaling 2,214 fingerprint values per molecule (5,666 descriptors if descriptors=True)
- Downstream features show zero-variance columns removed and no pairs of features with correlation >0.9
- Any alvaDesc generation failures are imputed with the median plus a missing-indicator column, not silently dropped
- Trained regressors on fingerprint features should be competitive with the paper's best result: ~39.2±1.2 s MAE and 17.2±0.9 s median absolute error on SMRT (DNN benchmark)

## Limitations

- alvaDesc is commercial software under license, so the pipeline cannot reproduce fingerprints without it; the RDKit notebook offers a non-identical alternative
- alvaDesc may fail to generate some descriptors, requiring median imputation with a missing indicator before training
- For exact reproduction of paper results, the repository README specifies using branch 'paper', since the code continues to be developed for CEU Mass Mediator integration

## Evidence

- [readme] 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software: "5,666 molecular descriptors and 2,214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with"
- [readme] This function processes an instance of the alvaDesc software, and the input file path representing the compound of interest in the format SMILES, mol, SDF, mol2 or hin.: "This function processes an instance of the alvaDesc software, and the input file path representing the compound of interest in the format SMILES, mol, SDF, mol2 or hin."
- [methods] Descriptors and fingerprints were generated with the alvaDesc software.: "Descriptors and fingerprints were generated with the alvaDesc software."
- [methods] Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor.: "Descriptors were first standardized and imputed using median imputation when alvaDesc was not able to generate some descriptor."
