---
name: qtof-vs-orbitrap-fragmentation-behavior-analysis
description: Use when evaluating or retraining CFM-ID 4.0 on Orbitrap (or other non-QTOF)
  MS/MS data, or when benchmarking in silico MS-to-compound identification on the
  CASMI 2016 category 3 Orbitrap dataset.
when_to_use_negative:
- The input spectra were already collected on a QTOF instrument, where the paper's
  own QTOF-trained CFM-ID 4.0 model and its 147/208 baseline apply directly
- The analysis targets EI-MS/MS spectral prediction rather than ESI-MS/MS (the paper
  reports EI improvements separately, and this skill concerns ESI-QTOF/Orbitrap transfer)
- No candidate structure database is available, since identification requires scoring
  predicted spectra against candidates from CFM-ID's spectral database
tools:
- name: CFM-ID 4.0
  role: MSML neural network (two hidden layers, 128 nodes, fragmentation graph depth
    2, connectivity matrix features capped at 10 atoms) retrained on Orbitrap data
    and used for spectral prediction and compound identification
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 3.0
  role: Source of the candidate structures / spectral database used for the CASMI
    2016 evaluation with 10 ppm mass tolerance
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Deployment container for running CFM-ID 4.0 spectral prediction and identification
    locally
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:86458f104eb5513fef50a77ae42a5979b91224947113cfa2e6a60bf2ce3905e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# qtof-vs-orbitrap-fragmentation-behavior-analysis

## Summary

A benchmarking skill for testing whether CFM-ID 4.0's MS/MS spectral prediction generalizes across instrument types: retrain the MSML model on Orbitrap HCD MS/MS data instead of the QTOF Metlin data used in the paper, then evaluate compound identification on the CASMI 2016 category 3 benchmark against the reported QTOF-trained baseline of 147/208.

## When to use

Use when evaluating or retraining CFM-ID 4.0 on Orbitrap (or other non-QTOF) MS/MS data, or when benchmarking in silico MS-to-compound identification on the CASMI 2016 category 3 Orbitrap dataset.

## When NOT to use

- The input spectra were already collected on a QTOF instrument, where the paper's own QTOF-trained CFM-ID 4.0 model and its 147/208 baseline apply directly
- The analysis targets EI-MS/MS spectral prediction rather than ESI-MS/MS (the paper reports EI improvements separately, and this skill concerns ESI-QTOF/Orbitrap transfer)
- No candidate structure database is available, since identification requires scoring predicted spectra against candidates from CFM-ID's spectral database

## Inputs

- CASMI 2016 category 3 Orbitrap MS/MS spectra (208 spectra: 127 [M+H]+, 81 [M-H]-)
- Candidate chemical structures retrieved from the CFM-ID 3.0 spectral database using a 10 ppm mass tolerance
- Orbitrap MS/MS training data for retraining the MSML model (as a substitute for the Metlin QTOF training set)

## Outputs

- In silico predicted ESI-MS/MS spectra for each candidate compound at QTOF-mapped 10/20/40 eV collision energies
- Ranked candidate lists per CASMI spectrum (scored with the modified Dot Product function)
- Compound identification counts out of 208, tabulated against the QTOF-trained baseline of 147/208

## How to apply

Download the CASMI 2016 category 3 dataset (208 Orbitrap MS/MS spectra: 127 [M+H]+ and 81 [M-H]-) and retrieve candidate structures from CFM-ID 3.0's spectral database with a 10 ppm mass tolerance. Train the CFM-ID 4.0 MSML neural network — two hidden layers of 128 nodes each, fragmentation graph depth 2, connectivity matrix features capped at 10 atoms — on Orbitrap MS/MS data instead of the QTOF Metlin data used in the paper. For each of the 208 CASMI spectra, use the Thermo Fisher Scientific conversion equation to map Orbitrap HCD spectra onto the QTOF 10/20/40 eV collision energies, then predict in silico MS/MS spectra for every candidate with the Orbitrap-trained MSML model. Score each spectrum against all candidates using the modified Dot Product function introduced by CFM-ID 2.0, rank candidates, and count correct identifications out of 208. Report both the Orbitrap-trained count and the paper's QTOF-trained baseline of 147/208 in a comparison table; do not fabricate improvements beyond the empirical evaluation.

## Related tools

- **CFM-ID 4.0** (MSML neural network (two hidden layers, 128 nodes, fragmentation graph depth 2, connectivity matrix features capped at 10 atoms) retrained on Orbitrap data and used for spectral prediction and compound identification) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 3.0** (Source of the candidate structures / spectral database used for the CASMI 2016 evaluation with 10 ppm mass tolerance) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Deployment container for running CFM-ID 4.0 spectral prediction and identification locally) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- A comparison table records both the QTOF-trained baseline (147/208 correct identifications) and the Orbitrap-trained count, with counts in a plausible 0-208 range
- All 208 CASMI 2016 category 3 spectra (127 [M+H]+ and 81 [M-H]-) are processed, not a subset
- Candidate retrieval uses the same 10 ppm mass tolerance as the paper's evaluation against CFM-ID 3.0's spectral database
- HCD spectra are mapped to all three QTOF collision energies (10/20/40 eV) via the Thermo Fisher Scientific conversion equation before prediction
- Candidate scoring consistently uses the modified Dot Product function from CFM-ID 2.0 for ranking

## Limitations

- The published CFM-ID 4.0 was trained only on QTOF Metlin data; Orbitrap retraining is an unvalidated extension the paper only flags as future work
- Runtime limitations are acknowledged without exact benchmarks; deeper fragmentation graphs introduce extra computational complexity
- Results depend on the Thermo Fisher conversion equation's assumption that Orbitrap HCD spectra are comparable to QTOF spectra at 10/20/40 eV — a cross-instrument mapping that may not hold for all compound classes

## Evidence
