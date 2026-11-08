---
name: bmrb-chemical-shift-retrieval
description: Use when building or updating a simulated INADEQUATE database of metabolites
  that requires 13C chemical shift assignments and structural connectivity, as PyINETA's
  matching step expects.
when_to_use_negative:
- When the goal is to annotate compounds directly from raw INADEQUATE spectra — that
  is PyINETA's peak picking/clustering pipeline, not database construction.
- When 13C chemical shift assignments for the target metabolites are not present in
  BMRB; the simulated database cannot cover compounds missing from the source.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0593
tools:
- name: PyINETA
  role: Python package that consumes the simulated INADEQUATE database built from
    BMRB data to match networks of picked peaks against metabolites
  repo: https://github.com/rhilt13/PyINETA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:7301d77d6131b9fb29644281473186c32137de1bcd375a465bbacab62e20d7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA
  tools:
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# bmrb-chemical-shift-retrieval

## Summary

Retrieve 13C chemical shifts and structural information from BMRB to build the simulated INADEQUATE reference database that PyINETA matches picked peak networks against.

## When to use

Use when building or updating a simulated INADEQUATE database of metabolites that requires 13C chemical shift assignments and structural connectivity, as PyINETA's matching step expects.

## When NOT to use

- When the goal is to annotate compounds directly from raw INADEQUATE spectra — that is PyINETA's peak picking/clustering pipeline, not database construction.
- When 13C chemical shift assignments for the target metabolites are not present in BMRB; the simulated database cannot cover compounds missing from the source.

## Inputs

- BMRB entries containing 13C chemical shift assignments
- Structural information (connectivity) for each metabolite from BMRB

## Outputs

- Simulated INADEQUATE database as a json file (1973 entries, 1209 metabolites)
- Per-compound ambiguity score CA/CT

## How to apply

Load per-compound 13C chemical shift assignments from BMRB along with the corresponding structural (connectivity) information for each metabolite. Simulate the INADEQUATE connectivity expected for each compound from the shifts and structure, computing the ambiguity score CA/CT — the number of carbons with multiple chemical shift assignments divided by the total number of carbons — to reflect assignment uncertainty. The build in the article produced 1973 entries covering 1209 metabolites, emitted as a json file that PyINETA's 'match' step consumes to identify metabolites present in query INADEQUATE spectra.

## Related tools

- **PyINETA** (Python package that consumes the simulated INADEQUATE database built from BMRB data to match networks of picked peaks against metabolites) — https://github.com/rhilt13/PyINETA

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Output json contains 1973 entries spanning 1209 metabolites, matching the article's build size.
- Each entry includes a CA/CT ambiguity score; verify CA ≤ CT for every compound.
- Every entry's simulated INADEQUATE connectivity is consistent with the BMRB structural information for that metabolite.
- Database loads successfully into PyINETA's 'match' step (e.g. via the -d DBNAME option) and returns matches for example1 query spectra.
- Compounds with multiple chemical shift assignments for the same carbon have higher CA/CT ambiguity scores than those with unique assignments.

## Limitations

- Coverage is limited to metabolites present in BMRB; the build contains 1973 entries for 1209 metabolites.
- Per-compound ambiguity is captured only through the CA/CT ratio — carbons with multiple chemical shift assignments over total carbons — and does not model other sources of uncertainty.

## Evidence

- [methods] The database is built from BMRB structural information and 1D 13C spectra; per-compound ambiguity is computed as CA/CT: [evidence span withheld — anchor status: not_found]
- [readme] which it then matches to a simulated INADEQUATE database of metabolites to identify: "which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra."
