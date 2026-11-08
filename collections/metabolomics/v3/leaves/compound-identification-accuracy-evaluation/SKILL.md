---
name: compound-identification-accuracy-evaluation
description: Use when you must quantify top-hit compound identification accuracy of
  an in silico MS2C tool on a labeled benchmark (e.g. CASMI 2016 category 3, 208 compounds)
  and compare it against prior versions or competing tools.
when_to_use_negative:
- Input spectra were collected on instruments other than QTOF and cannot be collision-energy
  converted — the authors note they still 'plan to adapt CFM-ID to handle MS spectra
  collected using instruments other than QTOF instruments'
- The goal is spectral prediction fidelity (peak matching) rather than compound identification
  accuracy over a ranked candidate list
- No candidate structure database or precursor-mass filter is available, so top-hit
  identification accuracy cannot be defined against a fixed candidate set
tools:
- name: CFM-ID 4.0
  role: Predicts ESI-MS/MS spectra for each candidate structure with the trained MSML
    model and performs the top-hit identification being benchmarked
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 3.0
  role: Source of the candidate spectral database (10 ppm mass tolerance) and the
    optional metadata scoring function (citation frequency)
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Provides the modified Dot Product scoring function used to rank candidates,
    and serves as the baseline for the 22.5% accuracy comparison
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Deployment container for running CFM-ID 4.0 in the benchmark pipeline
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: SIRIUS 4
  role: Competing in silico identification tool used as an external baseline in the
    CASMI 2016 comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:190760c8d62f10980e1d4617b18d02ae010e26633ca6be906258bb80617732f1
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
  - SIRIUS 4
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# compound-identification-accuracy-evaluation

## Summary

Benchmark an in silico MS/MS-based compound identification tool (e.g. CFM-ID 4.0) on a labeled challenge set such as CASMI 2016 category 3, by scoring predicted spectra against experimental spectra and ranking candidate structures, with and without metadata-assisted scoring.

## When to use

Use when you must quantify top-hit compound identification accuracy of an in silico MS2C tool on a labeled benchmark (e.g. CASMI 2016 category 3, 208 compounds) and compare it against prior versions or competing tools.

## When NOT to use

- Input spectra were collected on instruments other than QTOF and cannot be collision-energy converted — the authors note they still 'plan to adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments'
- The goal is spectral prediction fidelity (peak matching) rather than compound identification accuracy over a ranked candidate list
- No candidate structure database or precursor-mass filter is available, so top-hit identification accuracy cannot be defined against a fixed candidate set

## Inputs

- CASMI 2016 category-3 MS/MS spectra (208 Orbitrap HCD spectra; 127 positive [M+H]+, 81 negative [M-H]-) with known correct compound identities
- Candidate chemical structure lists retrieved per spectrum from a spectral database (CFM-ID 3.0 spectral DB, 10 ppm mass tolerance)
- Trained MSML prediction model for the relevant adduct (Metlin 2019 training sets)
- Optional metadata scores (e.g. citation frequency from CFM-ID 3.0 metadata scoring function)

## Outputs

- Per-spectrum ranked candidate list with modified Dot Product scores and top-hit identification
- Aggregate identification accuracy counts (correct/total) with and without metadata (e.g. 147/208 and 162/208)
- Comparative accuracy table against baselines (CFM-ID 2.0, CFM-ID 3.0, MS-Finder, SIRIUS 4), including percent improvement

## How to apply

Load the benchmark MS/MS spectra (208 CASMI 2016 category-3 Orbitrap HCD spectra: 127 positive [M+H]+ and 81 negative [M-H]-) and convert each Orbitrap spectrum to the corresponding QTOF collision energy levels (10, 20, 40 eV) using the Thermo Fisher Scientific conversion equation, since the predictor is QTOF-trained. Retrieve candidate structures for each spectrum from the CFM-ID 3.0 spectral database using a 10 ppm mass tolerance, then use the trained MSML model (fully connected neural network, two hidden layers of 128 nodes, connectivity matrix features capped at 10 atoms; trained on Metlin 2019 [M+H]+ 12,165 spectra/4055 metabolites and [M-H]- 6120 spectra/2040 metabolites) to predict ESI-MS/MS spectra per candidate. Apply post-processing that iteratively selects predicted peaks until 80% of total input intensity is reached or 30 peaks are output, score each candidate against the experimental spectrum with the modified Dot Product function from CFM-ID 2.0, and rank compounds to take the top hit. Compute identification accuracy twice — without metadata and with the CFM-ID 3.0 metadata scoring function (e.g. citation frequency) — and compare against baselines (CFM-ID 2.0/3.0, MS-Finder, SIRIUS 4). Validate against the reported counts: 147/208 correct without metadata, 162/208 with metadata, and a 22.5% accuracy improvement over CFM-ID 2.0.

## Related tools

- **CFM-ID 4.0** (Predicts ESI-MS/MS spectra for each candidate structure with the trained MSML model and performs the top-hit identification being benchmarked) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 3.0** (Source of the candidate spectral database (10 ppm mass tolerance) and the optional metadata scoring function (citation frequency)) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Provides the modified Dot Product scoring function used to rank candidates, and serves as the baseline for the 22.5% accuracy comparison) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Deployment container for running CFM-ID 4.0 in the benchmark pipeline) — https://hub.docker.com/r/wishartlab/cfmid
- **SIRIUS 4** (Competing in silico identification tool used as an external baseline in the CASMI 2016 comparison)

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Without-metadata run reproduces 147/208 correct identifications; with-metadata run reproduces 162/208
- Computed accuracy improvement over CFM-ID 2.0 equals 22.5%
- Post-processing invariant: predicted peak lists stop when 80% of total input intensity is reached or exactly 30 peaks are output
- All candidate lookups use a 10 ppm mass tolerance; spectrum adduct split (127 [M+H]+, 81 [M-H]-) sums to the 208 benchmark spectra
- Ranking uses the modified Dot Product function from CFM-ID 2.0 — a different similarity score indicates a protocol deviation

## Limitations

- CFM-ID 4.0 is QTOF-trained; non-QTOF instrument data require conversion (Orbitrap HCD to QTOF 10/20/40 eV via the Thermo Fisher equation) and native handling is future work
- Metadata-assisted scoring (citation frequency) inflates identification accuracy, so results must always be reported both with and without metadata
- Pure in silico identification without metadata help can be weak (~25% correct in earlier evaluations), so candidate-list coverage of the spectral database strongly bounds achievable accuracy
- Deeper fragmentation graphs and accuracy/runtime trade-offs remain open; the authors note 'there is still room for further improvement, both in terms of accuracy and runtime'

## Evidence
