---
name: rassp-element-constraint-filtering
description: Use when preparing large SMILES libraries (e.g. scraped ZINC20 lists)
  as input for RASSP synthetic spectra generation, and you must restrict compounds
  to those RASSP can convert into EI-MS spectra.
when_to_use_negative:
- When generating synthetic spectra with NEIMS instead of RASSP — NEIMS does not impose
  the same 8-element or 4096-fragment-subformula constraints, so this filter would
  needlessly discard valid NEIMS-pretraining compounds.
- When the target library contains compounds with rare elements or isotopologues (e.g.
  deuterium) outside RASSP's supported element set — the filter would eliminate them
  before spectra generation can even attempt them.
- When building a candidate list for database search methods (SSS, HSS, BDC) rather
  than de novo pretraining, since the filter is tuned to RASSP's forward-model feasibility,
  not to search-tool compatibility.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: RASSP
  role: Defines the feasibility constraints (8 most common elements, max 4096 possible
    fragment subformulae) used to filter compounds; only RASSP-processable molecules
    are retained
- name: Jupyter Notebook
  role: Used to implement and run the filtering workflow on the scraped SMILES lists
- name: conda
  role: Provides the Python environment in which the filtering scripts run (trainSpectus_build.yaml)
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:821382639dce34f4b08705a5819b9d31ee2b561c1c33f7d619e13f6445f5b26a
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - RASSP
  - conda
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rassp-element-constraint-filtering

## Summary

Filters large ZINC20-derived SMILES candidate sets down to compounds that the RASSP forward spectra generator can process, using RASSP's 8-element and fragment-subformula constraints, yielding the synth1 and synth2 pretraining corpora.

## When to use

Use when preparing large SMILES libraries (e.g. scraped ZINC20 lists) as input for RASSP synthetic spectra generation, and you must restrict compounds to those RASSP can convert into EI-MS spectra.

## When NOT to use

- When generating synthetic spectra with NEIMS instead of RASSP — NEIMS does not impose the same 8-element or 4096-fragment-subformula constraints, so this filter would needlessly discard valid NEIMS-pretraining compounds.
- When the target library contains compounds with rare elements or isotopologues (e.g. deuterium) outside RASSP's supported element set — the filter would eliminate them before spectra generation can even attempt them.
- When building a candidate list for database search methods (SSS, HSS, BDC) rather than de novo pretraining, since the filter is tuned to RASSP's forward-model feasibility, not to search-tool compatibility.

## Inputs

- Scraped ZINC20 SMILES list underlying synth1 (~30 million compounds, 2D-standard-annotated-druglike query)
- Scraped ZINC20 SMILES list underlying synth2 (~30 million compounds, 2D-standard-annotated-druglike query)
- NIST 20 compound list for leakage exclusion

## Outputs

- rassp_filtered_compounds.csv with ~4.7 million (synth1) and ~4.8 million (synth2) compounds, each row tagged by source corpus

## How to apply

Load the two scraped ZINC20 SMILES lists (each ~30 million compounds, obtained with the 2D-standard-annotated-druglike query) that underlie the synth1 and synth2 corpora, and parse each SMILES string to recover its molecular formula. Retain only compounds composed exclusively of the 8 most common elements supported by RASSP, and additionally discard compounds whose number of possible fragment subformulae exceeds RASSP's limit of 4096 fragment formulae. Remove any compound that also appears in the NIST 20 library to prevent data leakage, since SpecTUS is subsequently finetuned on NIST 20 spectra. Write the surviving compounds for both sets to rassp_filtered_compounds.csv, tagging each row with its source corpus (synth1 or synth2), and validate the counts against the reported sizes.

## Related tools

- **RASSP** (Defines the feasibility constraints (8 most common elements, max 4096 possible fragment subformulae) used to filter compounds; only RASSP-processable molecules are retained)
- **Jupyter Notebook** (Used to implement and run the filtering workflow on the scraped SMILES lists)
- **conda** (Provides the Python environment in which the filtering scripts run (trainSpectus_build.yaml)) — https://github.com/hejjack/SpecTUS

## Examples

```
conda env create -f trainSpectus_build.yaml && jupyter lab notebooks/1_data_preprocessing.ipynb  # filter ZINC SMILES to RASSP-processable compounds (8 elements, ≤4096 subformulae), excluding NIST 20 overlaps, writing rassp_filtered_compounds.csv
```

## Evaluation signals

- Filtered set sizes match reported values: ~4.7 million compounds for synth1 and ~4.8 million for synth2.
- No compound in the filtered output appears in the NIST 20 library (leakage check).
- Every retained compound contains only the 8 most common elements supported by RASSP.
- Every retained compound has at most 4096 possible fragment subformulae.
- Output CSV rows are tagged with source corpus (synth1 or synth2) and are disjoint across the two sets.

## Limitations

- The published RASSP model is limited to 4096 fragment formulae, so any compound exceeding this constraint is irrecoverably dropped.
- Filtering is specific to RASSP's element and subformula constraints; compounds excluded here may still be processable by other spectra generators (e.g. NEIMS).
- Excluding all NIST 20 compounds reduces the pretraining pool and is only appropriate when NIST 20 spectra are used for later finetuning.

## Evidence
