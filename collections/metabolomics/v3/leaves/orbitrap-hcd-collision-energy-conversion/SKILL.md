---
name: orbitrap-hcd-collision-energy-conversion
description: Use when input MS/MS spectra were acquired on an Orbitrap with HCD fragmentation
  but the downstream spectral-prediction model (e.g. CFM-ID 4.0 trained on Metlin
  QTOF spectra) expects QTOF collision energies at 10, 20 and 40 eV.
when_to_use_negative:
- Spectra were already acquired on a QTOF instrument — no cross-instrument energy
  conversion is needed and applying it would distort the collision energies
- Data come from non-Orbitrap/non-HCD instruments outside the QTOF scope the model
  was built for (CFM-ID's authors note adapting it to other instruments remains future
  work)
- The downstream scoring model was trained on Orbitrap-HCD data directly, so the energy
  domains already match
tools:
- name: CFM-ID 4.0
  role: 'Consumes the QTOF-converted spectra: its MSML model predicts ESI-MS/MS spectra
    for candidate structures at the converted 10/20/40 eV levels for compound identification'
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Deployment vehicle for running CFM-ID 4.0 with the converted spectra
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: CFM-ID 3.0
  role: Provides the candidate structure database (10 ppm mass tolerance) from which
    candidates are scored against converted spectra
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:58b9b4c752a9a23cb755a3164543a07abd0be39d4bb0a10b4faffabd3173b556
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CASMI 2016 challenge
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  - Metlin
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Orbitrap HCD-to-QTOF collision energy conversion

## Summary

Converts Orbitrap HCD MS/MS spectra into QTOF-equivalent collision energy spectra (10, 20, 40 eV) using the Thermo Fisher Scientific conversion equation, so that spectra from one instrument family can be scored against models or libraries trained on QTOF ESI-MS/MS data.

## When to use

Use when input MS/MS spectra were acquired on an Orbitrap with HCD fragmentation but the downstream spectral-prediction model (e.g. CFM-ID 4.0 trained on Metlin QTOF spectra) expects QTOF collision energies at 10, 20 and 40 eV.

## When NOT to use

- Spectra were already acquired on a QTOF instrument — no cross-instrument energy conversion is needed and applying it would distort the collision energies
- Data come from non-Orbitrap/non-HCD instruments outside the QTOF scope the model was built for (CFM-ID's authors note adapting it to other instruments remains future work)
- The downstream scoring model was trained on Orbitrap-HCD data directly, so the energy domains already match

## Inputs

- Orbitrap HCD MS/MS spectra in positive [M+H]+ and negative [M-H]- mode (e.g. CASMI 2016 category 3, 208 spectra)
- Candidate chemical structures retrieved from CFM-ID 3.0's spectral database within a 10 ppm mass tolerance

## Outputs

- Per-spectrum QTOF-equivalent collision energy series (10, 20, 40 eV) suitable for MSML spectral prediction
- Predicted ESI-MS/MS spectra per candidate at the three QTOF energies, ranked by modified Dot Product score

## How to apply

Take each Orbitrap HCD tandem MS spectrum (e.g. the 208 CASMI 2016 category-3 spectra: 127 positive [M+H]+ and 81 negative [M-H]-) and apply the Thermo Fisher Scientific conversion equation to map the HCD normalized collision energy onto the corresponding QTOF collision energy levels of 10, 20 and 40 eV. The rationale is that the CFM-ID 4.0 MSML model was trained exclusively on QTOF ESI-MS/MS data (Metlin 2019: 12,165 [M+H]+ spectra over 4,055 metabolites; 6,120 [M-H]- spectra over 2,040 metabolites), so unconverted Orbitrap spectra would be out-of-distribution for the predictor. After conversion, predict ESI-MS/MS spectra for each candidate structure at the three QTOF energy levels and score candidates against the experimental spectrum with the modified Dot Product function from CFM-ID 2.0. Decision point: run the pipeline twice, once without and once with the CFM-ID 3.0 metadata scoring function (e.g. citation frequency), since these two conditions are evaluated separately.

## Related tools

- **CFM-ID 4.0** (Consumes the QTOF-converted spectra: its MSML model predicts ESI-MS/MS spectra for candidate structures at the converted 10/20/40 eV levels for compound identification) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Deployment vehicle for running CFM-ID 4.0 with the converted spectra) — https://hub.docker.com/r/wishartlab/cfmid
- **CFM-ID 3.0** (Provides the candidate structure database (10 ppm mass tolerance) from which candidates are scored against converted spectra) — https://bitbucket.org/wishartlab/cfm-id-code/

## Evaluation signals

- Every converted spectrum maps to exactly the three QTOF energy levels 10, 20 and 40 eV — no residual Orbitrap NCE values remain in the workflow
- Reproducing the CASMI 2016 benchmark after conversion yields 147/208 correct identifications without metadata and 162/208 with metadata
- The identification accuracy shows a 22.5% improvement over CFM-ID 2.0 on the same converted dataset
- Conversion occurs as pipeline step 1, before candidate retrieval and MSML prediction — scoring unconverted Orbitrap spectra against the QTOF-trained model invalidates the comparison
- Positive and negative adduct subsets (127 [M+H]+, 81 [M-H]-) must total 208 and be scored with the adduct-matched trained models

## Limitations

- The Thermo Fisher Scientific conversion equation's explicit coefficients are not reproduced in the provided article text; the exact mapping must be sourced from Thermo Fisher documentation before reuse
- CFM-ID 4.0 is trained on QTOF spectra only; the authors state they plan to adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments, so converted spectra remain an approximation
- Deeper fragmentation graphs and prediction add computational complexity, and runtime limitations are acknowledged without exact benchmarks

## Evidence

- [methods] Orbitrap HCD to QTOF energy conversion step: [evidence span withheld — anchor status: not_found]
