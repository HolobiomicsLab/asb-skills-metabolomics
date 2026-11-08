---
name: carbon-shift-ambiguity-scoring
description: Decision support for ranking or weighting compound identifications in
  INADEQUATE-based NMR metabolomics when a simulated INADEQUATE database derived from
  BMRB contains metabolites whose carbons have multiple possible chemical shift assignments.
when_to_use_negative:
- The compound set is not amenable to 13C INADEQUATE simulation, e.g. metabolites
  without structural/carbon connectivity data or 1D 13C spectra in the reference database
- The goal is peak picking or spectral filtering of an experimental INADEQUATE spectrum
  rather than building or scoring the reference database
- Only unlabeled samples are available — the PyINETA approach relies on 13C labeling,
  so unlabeled workflows cannot use ambiguity scoring based on labeled carbon networks
tools:
- name: PyINETA
  role: Python package that reads and references INADEQUATE spectra, peak picks, clusters
    peaks into networks, and matches networks against the simulated INADEQUATE database;
    the ambiguity-scored simulated database is the reference it matches against
  repo: https://github.com/rhilt13/PyINETA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:70219963b315f5e5af56b8bcf638989b2b7e02e96ec395cf728912809645c62d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA
  tools:
  - PyINETA
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# carbon-shift-ambiguity-scoring

## Summary

Computes a per-compound ambiguity score (CA/CT) from a simulated INADEQUATE database built from BMRB 13C chemical shifts and structural information, quantifying how many carbons carry multiple chemical shift assignments relative to the total carbon count so downstream matching can account for assignment uncertainty.

## When to use

Decision support for ranking or weighting compound identifications in INADEQUATE-based NMR metabolomics when a simulated INADEQUATE database derived from BMRB contains metabolites whose carbons have multiple possible chemical shift assignments.

## When NOT to use

- The compound set is not amenable to 13C INADEQUATE simulation, e.g. metabolites without structural/carbon connectivity data or 1D 13C spectra in the reference database
- The goal is peak picking or spectral filtering of an experimental INADEQUATE spectrum rather than building or scoring the reference database
- Only unlabeled samples are available — the PyINETA approach relies on 13C labeling, so unlabeled workflows cannot use ambiguity scoring based on labeled carbon networks

## Inputs

- 13C chemical shift assignments for metabolites (from BMRB)
- Structural information / carbon connectivity for each metabolite
- Total metabolite set to include in the simulated INADEQUATE database (1209 metabolites, 1973 entries)

## Outputs

- Per-compound ambiguity score (CA/CT ratio) for each metabolite in the database
- Simulated INADEQUATE database emitted as a json file with 1973 entries covering 1209 metabolites

## How to apply

Load the 13C chemical shifts and structural information (connectivity) for each metabolite from BMRB, then simulate the INADEQUATE connectivity per compound to build the simulated INADEQUATE database entry. For each metabolite, count the number of carbons that carry more than one chemical shift assignment (CA) and divide by the total number of carbons (CT) to yield the ambiguity score; a higher ratio indicates greater assignment uncertainty for that compound. Store the resulting per-compound ambiguity alongside the simulated database — the build described in the source work contains 1973 entries covering 1209 metabolites and is emitted as a json file. This score can then be used downstream when PyINETA matches filtered networks of picked INADEQUATE peaks against the simulated database, so that metabolites with inherently ambiguous 13C assignments are not over-weighted in identification.

## Related tools

- **PyINETA** (Python package that reads and references INADEQUATE spectra, peak picks, clusters peaks into networks, and matches networks against the simulated INADEQUATE database; the ambiguity-scored simulated database is the reference it matches against) — https://github.com/rhilt13/PyINETA

## Examples

```
conda activate pyineta; python <path_to_pyineta_repo>/run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Every entry in the emitted json file carries both the simulated INADEQUATE connectivity and the computed CA/CT ambiguity score for its metabolite
- The database contains 1973 entries covering 1209 metabolites — a count mismatch signals incomplete build
- Ambiguity score is in the range [0,1]: 0 means no carbon has multiple shift assignments; a value approaching 1 means nearly every carbon is ambiguously assigned
- Cross-check that carbons counted as ambiguous actually have >1 distinct chemical shift assignment in the BMRB-derived shift list for that metabolite

## Limitations

- The ambiguity score reflects only the number of carbons with multiple shift assignments (CA/CT); it does not capture intensity, S/N, or spectral overlap of the experimental peaks
- PyINETA expects larger example2 input/temporary files that exceed 100 Mb and must be obtained from the Edison lab, so a full end-to-end rerun may not be self-contained from the repo alone
- The simulated database and ambiguity metric are only as good as the underlying BMRB shift/structure data; compounds missing BMRB entries will be absent from the 1973-entry build

## Evidence

- [results] Build database from BMRB with ambiguity score: [evidence span withheld — anchor status: not_found]
- [intro] pyINETA matches networks to simulated database: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [readme] pyINETA basic tasks: reading, referencing, peak picking: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [abstract] PyINETA tracks pipeline parameters for transparency: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
