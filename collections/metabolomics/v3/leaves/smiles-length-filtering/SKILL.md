---
name: smiles-length-filtering
description: Use when preprocessing raw EI-MS peak-list records (m/z / relative-intensity
  pairs with SMILES labels, e.g. from NIST 20 in msp format) whose SMILES strings
  may exceed the 100-character model input limit.
when_to_use_negative:
- The input SMILES strings are already verified to be within the model's length budget,
  so re-filtering would remove nothing
- Filtering synthetic ZINC20-derived datasets for spectra-generator training — that
  step uses RASSP-specific constraints (8 most common elements, number of possible
  subformulae), not the SMILES-length threshold
- Curating evaluation test sets for data leakage, where the relevant filters are training-set
  overlap and deuterium removal rather than sequence length
edam_operation: http://edamontology.org/operation_3695
tools:
- name: SpecTUS
  role: Source repository providing the documented preprocessing notebooks in which
    this SMILES-length filter is implemented
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Notebooks in the SpecTUS repo describe the data-preprocessing steps, including
    applying the SMILES-length threshold
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment setup (trainSpectus_build.yaml) required before running the preprocessing
    notebooks
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:e0e3c6daf95eaa4b4e432a2d0fb8782398154b70822e8319059ebe6837ff6f92
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  - conda
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# smiles-length-filtering

## Summary

A model-specific preprocessing filter from the SpecTUS pipeline that discards NIST 20 EI-MS spectral records whose associated SMILES string exceeds 100 characters, keeping sequences short enough for the transformer's input representation.

## When to use

Use when preprocessing raw EI-MS peak-list records (m/z / relative-intensity pairs with SMILES labels, e.g. from NIST 20 in msp format) whose SMILES strings may exceed the 100-character model input limit.

## When NOT to use

- The input SMILES strings are already verified to be within the model's length budget, so re-filtering would remove nothing
- Filtering synthetic ZINC20-derived datasets for spectra-generator training — that step uses RASSP-specific constraints (8 most common elements, number of possible subformulae), not the SMILES-length threshold
- Curating evaluation test sets for data leakage, where the relevant filters are training-set overlap and deuterium removal rather than sequence length

## Inputs

- Raw EI-MS spectra in msp format from NIST 20: peak lists (m/z, relative intensity) with associated SMILES labels

## Outputs

- Filtered spectra records (SMILES <= 100 characters) written to preprocessed_spectra.csv, ready for logarithmic intensity binning

## How to apply

Load the raw NIST 20 msp records with their SMILES labels, compute the character length of each SMILES string, and remove any record whose SMILES exceeds 100 characters — the maximum SMILES length threshold chosen for SpecTUS. This filter is applied jointly with the other model-specific thresholds: maximum m/z value of 500 and a maximum of 300 peaks per spectrum, which together exclude approximately 3% of the NIST dataset. Do not conflate this with other SpecTUS preprocessing filters (removal of overlaps with the NIST training set, compounds containing deuterium, or — for synthetic data — RASSP's element/subformula constraints). Rationale: the transformer decodes SMILES character-level, so overly long strings would exceed the model's sequence capacity; the surviving records are then intensity-binned (log base 1.28, 30 bins) and written to preprocessed_spectra.csv.

## Related tools

- **SpecTUS** (Source repository providing the documented preprocessing notebooks in which this SMILES-length filter is implemented) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Notebooks in the SpecTUS repo describe the data-preprocessing steps, including applying the SMILES-length threshold) — https://github.com/hejjack/SpecTUS
- **conda** (Environment setup (trainSpectus_build.yaml) required before running the preprocessing notebooks)

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && cd SpecTUS && conda env create -f trainSpectus_build.yaml  # then run the data-preprocessing notebook in notebooks/
```

## Evaluation signals

- Every retained record's SMILES string is at most 100 characters long
- Combined with the m/z <= 500 and <= 300 peaks filters, approximately 3% of the NIST dataset is excluded
- Each retained spectrum can subsequently be encoded into exactly 30 discrete intensity bins (log base 1.28, shift 29)
- The filtered records persist to preprocessed_spectra.csv without loss of the SMILES label column

## Limitations

- The 100-character threshold is a model-specific capacity limit, not a chemically motivated cutoff — records dropped may be valid spectra
- The ~3% exclusion figure is reported for all three thresholds combined, so the isolated effect of the SMILES-length filter is not quantified in the paper
- Reproducing the full filter pipeline on NIST 20 requires a license for the proprietary NIST20 dataset

## Evidence
