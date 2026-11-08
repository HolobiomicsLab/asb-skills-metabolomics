---
name: pubchem-compound-retrieval
description: Use when an ion-mobility metabolomics workflow has a target m/z with
  an adduct type (e.g. [M+H]+, [M−H]−, [M+Na]+) and needs candidate structures from
  PubChem to predict and filter CCS values.
when_to_use_negative:
- When the training set must cover adducts not in the CCSP 2.0 training data, e.g.
  dimers such as [2M+H]+ or [2M−H]− or other less common adducts or in-source fragments
- When a focused metabolite database (e.g. HMDB) would yield fewer false positives
  and narrower candidate lists than PubChem's broad coverage
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Open-source Jupyter Notebook that orchestrates the PubChem query, candidate
    retrieval, and descriptor computation as the first stage of its CCS prediction
    and filtering workflow
  repo: https://github.com/facundof2016/CCSP2.0
- name: PubChemPy
  role: Fetches matching compound records and CIDs from PubChem, extracting SMILES
    for each mass-search hit
- name: Biopython
  role: Interfaces with PubChem's mass-search service via Entrez/E-utilities to retrieve
    compounds whose exact mass falls inside the ±10 ppm window
- name: Mordred
  role: Computes the library of 1613 molecular descriptors (including exact mass)
    for each candidate structure using its SMILES
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook (Google Colab or locally hosted
    Jupyter Lab)
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:71f8201d3921e8ba7eb538f85576c9fab34a89523dc116d466bb312ab9c909f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - Biopython
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - PubChem
  - PubChemPy
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# pubchem-compound-retrieval

## Summary

Retrieves candidate compound records from the PubChem database for a target ion mass and computes molecular descriptors (SMILES, exact mass, 1613 Mordred descriptors) to enable downstream CCS-based false-positive filtering in metabolomics annotation workflows.

## When to use

Use when an ion-mobility metabolomics workflow has a target m/z with an adduct type (e.g. [M+H]+, [M−H]−, [M+Na]+) and needs candidate structures from PubChem to predict and filter CCS values.

## When NOT to use

- When the training set must cover adducts not in the CCSP 2.0 training data, e.g. dimers such as [2M+H]+ or [2M−H]− or other less common adducts or in-source fragments
- When a focused metabolite database (e.g. HMDB) would yield fewer false positives and narrower candidate lists than PubChem's broad coverage

## Inputs

- Target ion m/z value from an IM-MS feature
- Adduct type (e.g. [M+H]+, [M−H]−, [M+Na]+, [M+K]+)
- Mass error tolerance in ppm (CCSP 2.0 uses 10 ppm)

## Outputs

- candidate_descriptors.csv containing columns for CID, SMILES, exact mass, and all 1613 Mordred molecular descriptors
- List of candidate compound CIDs within the ±10 ppm mass window

## How to apply

Convert the target m/z and adduct type into a neutral-molecule mass search window of ±10 ppm around the target. Query PubChem's mass-search service (via Biopython's Entrez/E-utilities interface) to retrieve compounds whose exact mass falls inside the window, then fetch matching compound records and CIDs through PubChemPy, extracting SMILES for each hit. Compute molecular descriptors for each candidate with the Mordred Python package, which provides a library of 1613 descriptors including exact mass. Assemble a table of CID, SMILES, exact mass, and descriptor columns and write it to candidate_descriptors.csv. The rationale is to generate a candidate list as broad as possible (PubChem was deliberately chosen over focused databases like HMDB to stress-test the filtering approach) so that downstream CCS prediction can remove incorrect structures while retaining correct annotations.

## Related tools

- **CCSP 2.0** (Open-source Jupyter Notebook that orchestrates the PubChem query, candidate retrieval, and descriptor computation as the first stage of its CCS prediction and filtering workflow) — https://github.com/facundof2016/CCSP2.0
- **PubChemPy** (Fetches matching compound records and CIDs from PubChem, extracting SMILES for each mass-search hit)
- **Biopython** (Interfaces with PubChem's mass-search service via Entrez/E-utilities to retrieve compounds whose exact mass falls inside the ±10 ppm window)
- **Mordred** (Computes the library of 1613 molecular descriptors (including exact mass) for each candidate structure using its SMILES)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook (Google Colab or locally hosted Jupyter Lab)) — https://github.com/facundof2016/CCSP2.0

## Examples

```
Download "CCSP 2.0 - CCS Prediction in Google Colab" from https://github.com/facundof2016/CCSP2.0, then upload the .ipynb in Google Colab and follow the embedded instructions to run the PubChem mass search at 10 ppm.
```

## Evaluation signals

- candidate_descriptors.csv exists and contains all 1613 Mordred descriptor columns
- Every listed candidate's exact mass lies within the ±10 ppm window of the target m/z
- Each candidate row has a non-empty SMILES string and a PubChem CID
- Retrieved records reflect the adduct type: e.g. for [M+Na]+, masses are converted to the neutral-molecule window before querying, not the sodium-adducted mass

## Limitations

- Current CCSP 2.0 training sets allow prediction of the most common adduct ion types including [M−H]−, [M+H]+, [M+Na]+, and [M+K]+, but not dimers such as [2M+H]+ or [2M−H]−, or other less common adducts or in-source fragments
- PubChem's breadth means many incorrect structures will be retrieved; the retrieval step alone does not discriminate — CCS prediction with a ΔCCS threshold of 2.8% and a mass error of 10 ppm is required downstream to filter false positives
- Google Colab execution is limited to ~12 hours of continuous notebook operation and disconnects after ~20 minutes if left idle

## Evidence
