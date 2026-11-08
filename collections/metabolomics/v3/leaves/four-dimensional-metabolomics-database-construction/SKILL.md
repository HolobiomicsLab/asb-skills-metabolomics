---
name: four-dimensional-metabolomics-database-construction
description: 'Use when building an in-silico LC-IM-MS/MS reference library in which
  each derivatized sterol (or other C=C-bearing lipid) entry must carry all four 4D
  fields: m/z, retention time, CCS, and fragment ions.'
when_to_use_negative:
- The target molecules lack C=C bonds or are outside the sterol lipid class — the
  MS/MS calculation script recognizes double bond positions and was only tested on
  sterol lipids, though theoretically applicable to all molecules containing C=C bonds
- An experimentally measured spectral/RT/CCS library already exists for the analytes
  of interest — the 4D database is an in-silico prediction resource, not a replacement
  for measured standards
- The analysis does not involve N-Me derivatization — the fragment ion prediction
  logic is specific to N-Me fragmentation patterns
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RDkit
  role: 'Basis of the MS/MS calculation script: parses sterol structures, recognizes
    double bond positions, and generates fragment ions according to N-Me fragmentation
    patterns'
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: scikit-learn
  role: 'Implements the CCS prediction model: LASSO feature selection plus an SVR
    model with cross-validated hyperparameters, yielding the CCS field of the 4D database'
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: seokhokang/retention_time_gnn
  role: Source of the pretrained graph neural network used for the retention time
    prediction layer of the 4D database
  repo: https://github.com/seokhokang/retention_time_gnn/
- name: Jupyter Notebook
  role: All database-construction functions (MS/MS calculation, CCS prediction, data
    processing) are implemented in Jupyter notebooks in QCCAssisted4DSterol
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:4162bb793e21e958b652d367dcad782170f3200763cd486cfed5e6d58d74feae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  - RDkit
  - scikit-learn
  - seokhokang/retention_time_gnn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# four-dimensional-metabolomics-database-construction

## Summary

Assembles a large-scale 4D information database of derivatized sterol lipids (n = 4891) by combining per-molecule m/z, retention time (pretrained GNN), collision cross section (QCC-assisted scikit-learn SVR), and RDkit-predicted N-Me fragment ions into a single searchable table. It enables LC-IM-MS/MS identification of derivatized sterols at the isomer level without requiring experimental standards for every candidate.

## When to use

Use when building an in-silico LC-IM-MS/MS reference library in which each derivatized sterol (or other C=C-bearing lipid) entry must carry all four 4D fields: m/z, retention time, CCS, and fragment ions.

## When NOT to use

- The target molecules lack C=C bonds or are outside the sterol lipid class — the MS/MS calculation script recognizes double bond positions and was only tested on sterol lipids, though theoretically applicable to all molecules containing C=C bonds
- An experimentally measured spectral/RT/CCS library already exists for the analytes of interest — the 4D database is an in-silico prediction resource, not a replacement for measured standards
- The analysis does not involve N-Me derivatization — the fragment ion prediction logic is specific to N-Me fragmentation patterns

## Inputs

- N-Me derivatized sterol lipid candidate structures (SMILES/SDF parsed with RDkit, including C=C double bond positions)
- Quantum-chemistry-derived molecular descriptors/features for CCS prediction (input to the LASSO-selected, cross-validated SVR model)
- Structures amenable to GNN retention time prediction (molecular graphs, per seokhokang/retention_time_gnn)

## Outputs

- sterol_4d_database.csv: one row per derivatized sterol lipid (n = 4891) with four 4D fields — m/z, predicted retention time, predicted CCS, and predicted fragment ions
- Per-molecule predicted fragment ion lists based on N-Me fragmentation patterns
- Combined 4D table used for LC-IM-MS/MS based sterolomics data processing and identification

## How to apply

First generate the four prediction layers per derivatized sterol candidate: (1) m/z from the N-Me derivatized molecular formula; (2) retention time via the pretrained graph neural network approach from seokhokang/retention_time_gnn (Kwon et al., pretrained on METLIN-SMRT, transfer-learned with run_transfer.py); (3) CCS via the quantum-chemistry-calculation-assisted scikit-learn SVR model, which uses LASSO feature selection and cross-validated hyperparameters; and (4) fragment ions from the RDkit-based MS/MS script, which recognizes double bond positions and generates fragments according to N-Me fragmentation patterns. Then key all four fields by sterol lipid identity and combine them into a single table (e.g. sterol_4d_database.csv), covering all derivatized sterol lipids. Finally validate that the assembled database contains the reported n = 4891 entries and that every entry carries all four 4D fields before using it for LC-IM-MS/MS search against ./Search/tissue sample files.

## Related tools

- **RDkit** (Basis of the MS/MS calculation script: parses sterol structures, recognizes double bond positions, and generates fragment ions according to N-Me fragmentation patterns) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **scikit-learn** (Implements the CCS prediction model: LASSO feature selection plus an SVR model with cross-validated hyperparameters, yielding the CCS field of the 4D database) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **seokhokang/retention_time_gnn** (Source of the pretrained graph neural network used for the retention time prediction layer of the 4D database) — https://github.com/seokhokang/retention_time_gnn/
- **Jupyter Notebook** (All database-construction functions (MS/MS calculation, CCS prediction, data processing) are implemented in Jupyter notebooks in QCCAssisted4DSterol) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Examples

```
import pandas as pd; db = pd.DataFrame({'mz': mz, 'RT': rt_pred, 'CCS': ccs_pred, 'fragments': frag_ions}); db.to_csv('sterol_4d_database.csv', index=False)
```

## Evaluation signals

- Row-count invariant: the assembled database contains exactly the reported n = 4891 derivatized sterol lipid entries
- Schema check: every entry carries all four 4D fields (m/z, retention time, CCS, fragment ions) with no missing per-molecule values
- Fragment ion plausibility: predicted MS/MS fragments follow N-Me fragmentation patterns and are consistent with the recognized double bond positions of each sterol
- Prediction provenance: CCS values come from the LASSO/cross-validated scikit-learn SVR model and RT values from the pretrained GNN (seokhokang/retention_time_gnn), not ad hoc estimates
- Downstream usability: the database can be searched against LC-IM-MS/MS sample files organized under ./Search/tissue for sterol identification

## Limitations

- The RDkit MS/MS script is 'theoretically applicable to all molecules including C=C bond (only test sterol lipids)' — fragment rules were validated only on sterol lipids, so extension to other lipid classes is untested
- RT predictions rely on transfer learning from a small training dataset with a pretrained GNN, so accuracy depends on the similarity of target sterols to the pretrained/transfer data (METLIN-SMRT, PredRet, MoNA)
- CCS values are predictions from a machine learning model assisted by quantum chemistry calculations, not experimentally measured drift times

## Evidence
