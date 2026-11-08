---
name: molecular-descriptor-calculation
description: Use when you have a list of candidate compound structures (SMILES from
  PubChem or similar) and need molecular descriptors as numeric features for machine
  learning models like SVR CCS prediction.
when_to_use_negative:
- Input is already a numeric feature/identifier table with no structural information
  — descriptor calculation requires SMILES or comparable structure representation.
- You only need a single physicochemical property (e.g. exact mass) rather than the
  full 1613-descriptor feature set; a simpler tool would suffice.
- Structures are not available as computable SMILES (e.g. only formulas or InChIKeys
  without conversion) since Mordred needs a molecular representation.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mordred
  role: Computes the 1613 molecular descriptors from each candidate structure's SMILES;
    the descriptor matrix is used to train and apply the linear SVR CCS prediction
    models in CCSP 2.0
- name: PubChemPy
  role: Fetches matching compound records and CIDs from PubChem, extracting canonical
    SMILES that serve as the structural input for descriptor calculation
- name: CCSP 2.0
  role: Open-source Jupyter Notebook (Google Colab or Jupyter Lab) that orchestrates
    the descriptor computation as part of the CCS prediction and filtering workflow
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:7086b376d48b7f2ffee7b7b62fa21067a85490f8ce533549591a47d155e91e6a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - Biopython
  - CCSP 2.0
  - Mordred
  - PubChemPy
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# molecular-descriptor-calculation

## Summary

Compute a full panel of 1613 molecular descriptors from candidate compound structures (SMILES) with the Mordred Python package, as used in CCSP 2.0 to support CCS prediction and annotation filtering.

## When to use

Use when you have a list of candidate compound structures (SMILES from PubChem or similar) and need molecular descriptors as numeric features for machine learning models like SVR CCS prediction.

## When NOT to use

- Input is already a numeric feature/identifier table with no structural information — descriptor calculation requires SMILES or comparable structure representation.
- You only need a single physicochemical property (e.g. exact mass) rather than the full 1613-descriptor feature set; a simpler tool would suffice.
- Structures are not available as computable SMILES (e.g. only formulas or InChIKeys without conversion) since Mordred needs a molecular representation.

## Inputs

- Target m/z and adduct type
- Isomeric or canonical SMILES strings for each candidate compound (retrieved via PubChemPy from PubChem CIDs)

## Outputs

- candidate_descriptors.csv with CID, SMILES, exact mass, and 1613 molecular descriptor columns per candidate

## How to apply

Retrieve the canonical/isomeric SMILES for each candidate compound (e.g. via PubChemPy from PubChem records) and pass each structure to the Mordred Python package to compute all 1613 molecular descriptors. In CCSP 2.0 these descriptors are computed inside the Jupyter notebook for both training-set structures and for all PubChem candidates retrieved within the 10 ppm mass-error window of the target m/z. Assemble the results into a table with CID, SMILES, exact mass, and descriptor columns (e.g. candidate_descriptors.csv) so the linear SVR model can consume the feature matrix. Validate that the output file contains the 1613 descriptor columns and that every candidate's exact mass lies within the 10 ppm window. Mordred is chosen because it exposes a large library of descriptors via a Python API that integrates with scikit-learn-compatible SVR workflows.

## Related tools

- **Mordred** (Computes the 1613 molecular descriptors from each candidate structure's SMILES; the descriptor matrix is used to train and apply the linear SVR CCS prediction models in CCSP 2.0)
- **PubChemPy** (Fetches matching compound records and CIDs from PubChem, extracting canonical SMILES that serve as the structural input for descriptor calculation)
- **CCSP 2.0** (Open-source Jupyter Notebook (Google Colab or Jupyter Lab) that orchestrates the descriptor computation as part of the CCS prediction and filtering workflow) — https://github.com/facundof2016/CCSP2.0

## Examples

```
import pandas as pd; from rdkit import Chem; from mordred import Calculator, descriptors; calc = Calculator(descriptors, ignore_3D=False); df = pd.read_csv('candidate_smiles.csv'); df.join(pd.DataFrame([calc(Chem.MolFromSmiles(s)) for s in df.smiles])).to_csv('candidate_descriptors.csv', index=False)
```

## Evaluation signals

- Output table (e.g. candidate_descriptors.csv) exists and contains exactly the 1613 Mordred descriptor columns.
- Every listed candidate's exact mass lies within the 10 ppm window around the target m/z.
- Descriptor values are populated (not all-NaN) for each candidate SMILES.
- For training-set structures, the resulting SVR model achieves the paper's reported median relative CCS prediction errors (1.25% for [M−H]−, 1.73% for [M+H]+, 1.87% for [M+Na]+).
- Applying ΔCCS 2.8% threshold retains 100% of correct annotations and filters ~36.1% of incorrect class-matched structures.

## Limitations

- Requires scikit-learn v1.0.2 or later; full requirements listed in the repository's requirements.txt.
- Google Colab sessions are limited to ~12 h of continuous operation and disconnect after ~20 min idle — use the locally hosted Jupyter Lab version for large (>10,000 molecules) prediction sets.
- Descriptor calculation is only as good as the SMILES retrieved from PubChem; candidate records must be accessible for every query hit.
- The 1613 descriptors are fixed by the Mordred library; custom descriptor sets are not part of the standard CCSP 2.0 workflow.
- Current CCSP 2.0 training sets cover [M−H]−, [M+H]+, [M+Na]+, and [M+K]+ but not dimers ([2M+H]+, [2M−H]−) or other less common adducts/in-source fragments.

## Evidence

- [methods] Workflow from source card (task_004): [evidence span withheld — anchor status: not_found]
- [methods] Validation from source card (task_004): [evidence span withheld — anchor status: not_found]
- [methods] SMILES retrieval via PubChemPy (task_004): [evidence span withheld — anchor status: not_found]
- [results] Training-set descriptor computation (task_006): [evidence span withheld — anchor status: not_found]
