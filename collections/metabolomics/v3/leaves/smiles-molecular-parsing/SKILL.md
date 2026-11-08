---
name: smiles-molecular-parsing
description: Use when building synthetic-spectrum pretraining corpora from a large
  SMILES library and you must decide, compound by compound, whether a spectra generator
  (RASSP) can actually process that molecule.
when_to_use_negative:
- You need the full unfiltered ZINC candidate set (e.g. for hybrid database search
  baselines) — this skill deliberately discards compounds RASSP cannot process.
- Your downstream spectra generator is not RASSP and accepts a different element set
  or subformula limit, so the 8-element and 4096-fragment filters would be inappropriate.
- The SMILES list is small and curated (e.g. an experimental library like NIST 20
  itself) rather than a 30-million-scale scraped set — per-compound filtering overhead
  is unnecessary.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: RASSP
  role: 'Defines the processability constraints this filter enforces: the 8 most common
    elements and the 4096 possible fragment subformulae limit'
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Used to run the SMILES parsing and filtering pipeline described in the notebooks/
    directory
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment manager for reproducing the Python environment (trainSpectus_build.yaml)
    in which the filtering notebooks run
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:ec723e2ca78c1678cbeb8d7a75298437ae597ce88dcac4d9b8f978f652131e64
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# smiles-molecular-parsing

## Summary

Parse ZINC20-derived SMILES strings to determine which compounds RASSP can process — restricting to the 8 most common elements and the 4096-fragment-formula limit — to shrink ~30-million-compound candidate sets into the synth1/synth2 pretraining corpora.

## When to use

Use when building synthetic-spectrum pretraining corpora from a large SMILES library and you must decide, compound by compound, whether a spectra generator (RASSP) can actually process that molecule.

## When NOT to use

- You need the full unfiltered ZINC candidate set (e.g. for hybrid database search baselines) — this skill deliberately discards compounds RASSP cannot process.
- Your downstream spectra generator is not RASSP and accepts a different element set or subformula limit, so the 8-element and 4096-fragment filters would be inappropriate.
- The SMILES list is small and curated (e.g. an experimental library like NIST 20 itself) rather than a 30-million-scale scraped set — per-compound filtering overhead is unnecessary.

## Inputs

- ~30M-compound ZINC20 SMILES lists scraped with the 2D-standard-annotated-druglike query (one list per target corpus: synth1, synth2)
- NIST 20 compound list used for the de-overlap step

## Outputs

- rassp_filtered_compounds.csv — one row per surviving compound, tagged with its source corpus (synth1 or synth2), totalling ~4.7M (synth1) and ~4.8M (synth2) compounds

## How to apply

Load the scraped ZINC20 SMILES lists (each ~30 million compounds, scraped with the 2D-standard-annotated-druglike query), parse each SMILES string, and keep only compounds composed exclusively of the 8 most common elements supported by RASSP. Next enforce RASSP's structural limit of at most 4096 possible fragment subformulae per compound, since the published RASSP model cannot enumerate more. Then remove any compound that also appears in the NIST 20 library to prevent data leakage, because SpecTUS is later finetuned on NIST 20 spectra. Tag each surviving row with its source corpus (synth1 or synth2) and write the result to rassp_filtered_compounds.csv. Expect roughly 4.7 million survivors for synth1 and 4.8 million for synth2.

## Related tools

- **RASSP** (Defines the processability constraints this filter enforces: the 8 most common elements and the 4096 possible fragment subformulae limit) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Used to run the SMILES parsing and filtering pipeline described in the notebooks/ directory) — https://github.com/hejjack/SpecTUS
- **conda** (Environment manager for reproducing the Python environment (trainSpectus_build.yaml) in which the filtering notebooks run) — https://github.com/hejjack/SpecTUS

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git; conda env create -f trainSpectus_build.yaml; jupyter lab notebooks/1_data_preprocessing.ipynb
```

## Evaluation signals

- synth1 filtered set size ≈ 4.7 million compounds (paper reports 4.7)
- synth2 filtered set size ≈ 4.8 million compounds (paper reports 4.8)
- Every retained compound contains only the 8 most common elements and yields ≤ 4096 possible fragment subformulae under RASSP's enumeration
- Zero overlap between retained compounds and the NIST 20 library
- rassp_filtered_compounds.csv rows are tagged with their source corpus so synth1/synth2 provenance is unambiguous

## Limitations

- Filtering drops roughly 83% of each 30-million-compound set, so compounds outside RASSP's element/subformula scope are permanently excluded from pretraining
- The 4096-fragment-formula cap is a property of the published RASSP model — if you train a different generator, the threshold must be re-derived
- De-overlap is against NIST 20 specifically; other test libraries (SWGDRUG, Cayman, MONA) require their own overlap checks

## Evidence

- [results] RASSP's fragment-formula limit is the hard constraint that makes the filter necessary.: [evidence span withheld — anchor status: not_found]
