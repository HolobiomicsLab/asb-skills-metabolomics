---
name: inadequate-connectivity-simulation
description: Use when building or extending a reference database of metabolite 13C-13C
  connectivity networks (INADEQUATE-style) from public chemical shift data (BMRB),
  or when computing per-compound ambiguity (CA/CT) to weight downstream compound annotation
  confidence.
when_to_use_negative:
- The query data are already picked, clustered peak networks ready to be matched —
  use pyINETA's match step, not the database simulation.
- The compounds of interest have no BMRB structural information or no 13C chemical
  shifts, so no connectivity simulation can be grounded.
- The study requires quantitation or isotope-tracking of 13C enrichment rather than
  database construction for connectivity matching.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0584
tools:
- name: PyINETA
  role: Open-source Python platform whose pipeline matches query INADEQUATE peak networks
    against the simulated INADEQUATE database built by this skill
  repo: edisonomics/PyINETA
- name: Python
  role: Language used to implement the database simulation and ambiguity-score computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:dc45782b15f166ebc70c18c5ed45346fab35185311bcc0c023e755ffd225de9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0584
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# inadequate-connectivity-simulation

## Summary

Generates a simulated INADEQUATE database of metabolites from BMRB structural information and 1D 13C chemical shifts, with a per-compound ambiguity score (CA/CT) reflecting assignment uncertainty. This simulated database is the reference that pyINETA matches peak networks against to identify metabolites in query INADEQUATE spectra.

## When to use

Use when building or extending a reference database of metabolite 13C-13C connectivity networks (INADEQUATE-style) from public chemical shift data (BMRB), or when computing per-compound ambiguity (CA/CT) to weight downstream compound annotation confidence.

## When NOT to use

- The query data are already picked, clustered peak networks ready to be matched — use pyINETA's match step, not the database simulation.
- The compounds of interest have no BMRB structural information or no 13C chemical shifts, so no connectivity simulation can be grounded.
- The study requires quantitation or isotope-tracking of 13C enrichment rather than database construction for connectivity matching.

## Inputs

- BMRB entries providing structural information for metabolites
- BMRB 1D 13C chemical shift lists per metabolite

## Outputs

- Simulated INADEQUATE database (JSON) with 1973 entries covering 1209 metabolites
- Per-compound ambiguity score CA/CT (carbons with multiple chemical shift assignments over total carbons)

## How to apply

Load each metabolite's structural information and its 1D 13C chemical shifts from BMRB; simulate the 13C-13C connectivity (INADEQUATE-style pair network) implied by the structure and those shifts. For each compound, count CT = total carbons and CA = carbons with multiple chemical shift assignments, then compute the ambiguity score CA/CT to quantify assignment uncertainty. Emit the simulated database as a JSON file — the published build contains 1973 entries covering 1209 metabolites — which pyINETA later uses to match peak networks from query INADEQUATE spectra. Rationale: matching against structurally grounded, shift-annotated pairs (rather than raw shift lists) lets pyINETA identify compounds and even assign fragments to unknowns, and the CA/CT score propagates annotation uncertainty from the database into downstream calls.

## Related tools

- **PyINETA** (Open-source Python platform whose pipeline matches query INADEQUATE peak networks against the simulated INADEQUATE database built by this skill) — edisonomics/PyINETA
- **Python** (Language used to implement the database simulation and ambiguity-score computation)

## Evaluation signals

- Output JSON contains 1973 entries covering 1209 metabolites (the published build size).
- For every compound in the database, 0 ≤ CA ≤ CT and CT equals the compound's total carbon count from BMRB structure.
- Each simulated connectivity is consistent with the compound's known structure — carbons only linked to bonded neighbors implied by the BMRB structural information.
- Chemical shifts in the simulated pairs reproduce the BMRB 13C shift values with correct pairing to their bonded carbon partner.
- Downstream check: pyINETA match step runs against this JSON and returns identified metabolites for known query spectra.

## Limitations

- Build scope is limited to metabolites with BMRB structural information and 1D 13C spectra — compounds absent from BMRB are excluded from the 1973-entry database.
- Ambiguity CA/CT reflects only multiple chemical shift assignments per compound, not experimental uncertainty in the query spectra.
- The skill name references simulated INADEQUATE connectivity; it does not substitute for acquiring or peak-picking actual INADEQUATE spectra.

## Evidence

- [methods] BMRB database: [evidence span withheld — anchor status: not_found]
- [methods] database details: [evidence span withheld — anchor status: not_found]
- [methods] ambiguity score: [evidence span withheld — anchor status: not_found]
- [methods] database file format: [evidence span withheld — anchor status: not_found]
