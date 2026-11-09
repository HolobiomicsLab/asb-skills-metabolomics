---
name: metabolite-database-json-construction
description: Use when PyINETA needs a reference metabolite database for INADEQUATE-based
  compound identification, or when reconstructing how per-compound ambiguity (CA/CT)
  quantifies assignment uncertainty in BMRB-derived shift data.
when_to_use_negative:
- The goal is identifying metabolites in existing INADEQUATE spectra rather than constructing
  the reference database
- The data are unlabeled 1D 13C spectra without the structural connectivity information
  needed to simulate INADEQUATE networks
- A complementary 13C-JRES integration workflow is required instead of database construction
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: PyINETA
  role: Python package whose simulated INADEQUATE database of metabolites is produced
    by this skill and used to match filtered peak networks
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Implementation language for database construction and json emission
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:e5bdf2da695303707d72e4a9033c8ed0b1507bbf3b5f1c5241c000555d08017b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolite-database-json-construction

## Summary

Builds a simulated INADEQUATE reference database by loading 13C chemical shifts and structural information from BMRB, simulating INADEQUATE connectivity per metabolite, and emitting a json file (1973 entries covering 1209 metabolites) that PyINETA matches query spectra against.

## When to use

Use when PyINETA needs a reference metabolite database for INADEQUATE-based compound identification, or when reconstructing how per-compound ambiguity (CA/CT) quantifies assignment uncertainty in BMRB-derived shift data.

## When NOT to use

- The goal is identifying metabolites in existing INADEQUATE spectra rather than constructing the reference database
- The data are unlabeled 1D 13C spectra without the structural connectivity information needed to simulate INADEQUATE networks
- A complementary 13C-JRES integration workflow is required instead of database construction

## Inputs

- BMRB 13C chemical shift entries
- BMRB structural information per metabolite

## Outputs

- Simulated INADEQUATE database as a json file (reference build: 1973 entries, 1209 metabolites)
- Per-compound ambiguity score CA/CT (carbons with multiple assignments / total carbons)

## How to apply

Load the 13C chemical shifts and structural information for each metabolite from BMRB, then simulate the INADEQUATE connectivity implied by the shift and structure data. For each compound compute the ambiguity score CA/CT — the number of carbons with multiple chemical shift assignments divided by the total number of carbons — to capture assignment uncertainty. Emit the full simulated INADEQUATE database as a json file; the reference build contains 1973 entries covering 1209 metabolites. This json database is what PyINETA matches filtered peak networks against during compound identification, so any custom build must preserve the connectivity and shift structure it expects.

## Related tools

- **PyINETA** (Python package whose simulated INADEQUATE database of metabolites is produced by this skill and used to match filtered peak networks) — https://github.com/edisonomics/PyINETA
- **Python** (Implementation language for database construction and json emission)

## Examples

```
git clone https://github.com/edisonomics/PyINETA.git; conda env create -f environment.yml; python run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Output json parses and contains 1973 entries covering 1209 metabolites
- Each compound carries an ambiguity score CA/CT where carbons with multiple assignments ≤ total carbons (score in [0,1])
- Per-metabolite INADEQUATE connectivity is consistent with the BMRB structural information and 13C shifts
- Database matches metabolites present in query INADEQUATE spectra when run through PyINETA's match step

## Limitations

- Ambiguity score CA/CT rises for metabolites whose carbons carry multiple chemical shift assignments, weakening unambiguous identification
- Database construction requires BMRB structural plus 1D 13C shift data; compounds lacking either cannot be included
- Example2 raw input files exceed 100 Mb and must be requested from the Edison lab
- PyINETA matches filtered peak networks to the database to identify metabolites 'ideally' coming from the same compound — peak-network clustering is imperfect

## Evidence

- [readme] Simulated INADEQUATE database build from BMRB: "matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra."
- [results] CA/CT ambiguity computation: [evidence span withheld — anchor status: not_found]
- [results] Database scale: 1973 entries / 1209 metabolites: [evidence span withheld — anchor status: not_found]
- [results] Database construction workflow: [evidence span withheld — anchor status: not_found]
