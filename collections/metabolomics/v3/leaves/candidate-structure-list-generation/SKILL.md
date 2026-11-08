---
name: candidate-structure-list-generation
description: Use when you have a measured precursor m/z and assigned adduct (e.g.
  [M+H]+, [M−H]−, [M+Na]+) and need a structured candidate-compound list with exact
  masses and descriptors as the input table for CCS-based false-positive filtering
  in metabolomics.
when_to_use_negative:
- You already have a curated candidate structure list (e.g. SMILES or CIDs from a
  focused database such as HMDB) — regenerating candidates via a broad PubChem 10
  ppm search would reintroduce false positives the CCS step is meant to remove.
- You need CCS predictions for adducts outside the training set, such as dimers ([2M+H]+,
  [2M−H]−) or in-source fragments, since the descriptor-generation pipeline is built
  around the supported common adduct types.
- You are annotating compounds whose exact mass falls outside PubChem's coverage —
  the mass search relies on PubChem as the structure source, and broader mass windows
  change the false-positive filtering conditions evaluated in the paper.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: CCSP 2.0
  role: Jupyter notebook framework whose candidate-list cells implement the 10 ppm
    mass search, PubChem retrieval, descriptor calculation, and export to candidate_descriptors.csv
  repo: https://github.com/facundof2016/CCSP2.0
- name: PubChem
  role: Structure database queried via its mass-search service to retrieve compounds
    whose exact mass falls within the ±10 ppm window
- name: Biopython
  role: Python package used to access PubChem's E-utilities/Entrez interface for the
    mass search
- name: PubChemPy
  role: Fetches matching compound records and CIDs and extracts canonical/isomeric
    SMILES for each hit
- name: Mordred
  role: Computes all 1613 molecular descriptors for each candidate from its isomeric
    SMILES
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook cells that perform the candidate-list
    workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:a5de627467e0b312f03f3fd902f6e78d3bf5ea3f348d89b5a0d809d5bfc8bdec
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# candidate-structure-list-generation

## Summary

Build the CCS-predictor candidate pool by converting a target m/z and adduct type into a ±10 ppm neutral-mass window, querying PubChem via Biopython's Entrez/E-utilities mass search, retrieving each candidate's SMILES through PubChemPy, and computing all 1613 Mordred molecular descriptors for each candidate before CCS prediction.

## When to use

Use when you have a measured precursor m/z and assigned adduct (e.g. [M+H]+, [M−H]−, [M+Na]+) and need a structured candidate-compound list with exact masses and descriptors as the input table for CCS-based false-positive filtering in metabolomics.

## When NOT to use

- You already have a curated candidate structure list (e.g. SMILES or CIDs from a focused database such as HMDB) — regenerating candidates via a broad PubChem 10 ppm search would reintroduce false positives the CCS step is meant to remove.
- You need CCS predictions for adducts outside the training set, such as dimers ([2M+H]+, [2M−H]−) or in-source fragments, since the descriptor-generation pipeline is built around the supported common adduct types.
- You are annotating compounds whose exact mass falls outside PubChem's coverage — the mass search relies on PubChem as the structure source, and broader mass windows change the false-positive filtering conditions evaluated in the paper.

## Inputs

- target precursor m/z value (user-supplied, high-accuracy mass measurement)
- assigned adduct ion type (e.g. [M−H]−, [M+H]+, [M+Na]+, [M+K]+)
- CCSP 2.0 Jupyter notebook (Google Colab or Jupyter Lab variant, .ipynb)

## Outputs

- candidate_descriptors.csv containing CID, SMILES, exact mass, and 1613 Mordred descriptor columns per candidate
- list of PubChem CIDs falling within the ±10 ppm mass window

## How to apply

Convert the user-supplied target m/z and adduct type into a neutral-molecule mass search window of ±10 ppm around the target. Query PubChem's mass-search service through Biopython's Entrez/E-utilities interface to retrieve compounds whose exact mass falls inside the 10 ppm window, then fetch the matching compound records and CIDs via PubChemPy and extract canonical SMILES for each hit. Compute all 1613 molecular descriptors for each candidate structure with the Mordred Python package, using each compound's isomeric SMILES. Assemble a table of CID, SMILES, exact mass, and descriptor columns and write it to candidate_descriptors.csv, which downstream CCSP 2.0 cells use for CCS prediction and filtering. Validate that the output file exists, contains the 1613 descriptor columns, and that every listed candidate's exact mass lies within the 10 ppm window of the target m/z.

## Related tools

- **CCSP 2.0** (Jupyter notebook framework whose candidate-list cells implement the 10 ppm mass search, PubChem retrieval, descriptor calculation, and export to candidate_descriptors.csv) — https://github.com/facundof2016/CCSP2.0
- **PubChem** (Structure database queried via its mass-search service to retrieve compounds whose exact mass falls within the ±10 ppm window)
- **Biopython** (Python package used to access PubChem's E-utilities/Entrez interface for the mass search)
- **PubChemPy** (Fetches matching compound records and CIDs and extracts canonical/isomeric SMILES for each hit)
- **Mordred** (Computes all 1613 molecular descriptors for each candidate from its isomeric SMILES)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook cells that perform the candidate-list workflow)

## Examples

```
In Google Colab: File > Open Notebook > Upload > Select "CCSP 2.0 - CCS Prediction for Google Colab.ipynb"; then run the mass-search and Mordred descriptor cells to produce candidate_descriptors.csv.
```

## Evaluation signals

- candidate_descriptors.csv exists and parses with the expected schema (CID, SMILES, exact mass, descriptor columns).
- The descriptor table contains the full library of 1613 molecular descriptors produced by Mordred.
- Every candidate's exact mass lies within the ±10 ppm window of the target m/z — spot-check a few rows against the measured m/z and adduct conversion.
- SMILES strings are non-empty and valid for each CID retrieved via PubChemPy; Mordred descriptor computation completed without errors for each candidate.
- Downstream CCSP 2.0 cells can load candidate_descriptors.csv and run SVR-based CCS prediction and filtering without schema errors.

## Limitations

- The broad PubChem search is deliberately unfavorable — the paper chose PubChem over focused databases such as HMDB to test filtering under the most adverse conditions, so candidate lists can be very large.
- Google Colab will only allow continuous notebook operation for up to 12 hours and disconnects after ~20 minutes if left idle, so large candidate sets may be better run in the locally hosted Jupyter Lab version.
- Descriptor/CCS prediction is tied to common adduct types ([M−H]−, [M+H]+, [M+Na]+, [M+K]+); dimers such as [2M+H]+ or [2M−H]− and less common adducts or in-source fragments are not supported by current training sets.
- Requires Sklearn V1.0.2 or later (plus the requirements.txt package list), so version mismatches can break the notebook cells used for retrieval and descriptor computation.

## Evidence

- [methods] Mass search window definition: [evidence span withheld — anchor status: not_found]
- [abstract] Descriptor library size: "inclusion of a large library of 1613 molecular descriptors via the Mordred Python package"
- [abstract] Filtering thresholds applied downstream of the candidate list: "retaining a total of 100% of the correct annotations using a ΔCCS threshold of 2.8% and a mass error of 10 ppm"
- [intro] PubChem chosen as structure source: "We chose PubChem vs more focused databases such as HMDB,31 so as to test the CCS filtering approach in the most adverse conditions"
- [readme] How to obtain and run the notebook: "Download "CCSP 2.0 - CCS Prediction in Google Colab" from this GitHub repository. It should save to your computer as a Python notebook file with extension ".ipynb"."
- [readme] Runtime constraint of the Colab route: "Google Colab will only allow continuous notebook operation for up to 12 hours and will disconnect after ~20 minutes if left idle."
