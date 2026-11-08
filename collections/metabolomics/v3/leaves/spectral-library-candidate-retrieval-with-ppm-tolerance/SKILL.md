---
name: spectral-library-candidate-retrieval-with-ppm-tolerance
description: Use when identifying compounds from precursor-mass-filtered ESI-MS/MS
  spectra (e.g. CASMI category 3 challenges) where candidates must be pulled from
  a spectral/structure database within a ppm window before in silico prediction and
  scoring.
when_to_use_negative:
- The observed spectrum already matches an entry in an experimentally acquired reference
  spectral library (RSL) — direct library matching is preferable to in silico candidate
  retrieval and scoring.
- The MS/MS data come from instruments CFM-ID 4.0 does not handle (non-QTOF instruments)
  and cannot be converted — the article acknowledges the 'inability to handle non-QTOF
  instruments'.
- The task is EI-MS/MS or spectral prediction alone, with no unknown-compound candidate
  search — then use the predictor directly rather than the retrieval/scoring loop.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 3.0
  role: Provides the spectral database from which candidate structures are retrieved
    at the 10 ppm tolerance, plus the optional metadata scoring function (citation
    frequency)
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 4.0
  role: Predicts ESI-MS/MS spectra for each retrieved candidate (MSML model trained
    on Metlin 2019 QTOF data) so candidates can be scored against the experimental
    spectrum
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Source of the modified Dot Product scoring function used to rank candidates
    against the observed spectrum
- name: Docker (wishartlab/cfmid image)
  role: Deployment vehicle for running the CFM-ID candidate retrieval and spectral
    prediction pipeline reproducibly
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: Metlin
  role: 'Experimental ESI-MS/MS training data (Metlin 2019: 12,165 [M+H]+ spectra
    / 4055 metabolites; 6120 [M-H]- spectra / 2040 metabolites) behind the predictor
    that scores retrieved candidates'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:438ac06278ab23402af4be4f22836c0104840247c2fd6f9a2c8e4f867ef94e80
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CASMI 2016 challenge
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  - Docker (wishartlab/cfmid image)
  - Metlin
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-library-candidate-retrieval-with-ppm-tolerance

## Summary

Retrieves candidate chemical structures for an unknown ESI-MS/MS spectrum by querying CFM-ID 3.0's spectral database with the precursor m/z at a 10 ppm mass tolerance, producing a ranked candidate set that is then scored against in silico predicted spectra for compound identification.

## When to use

Use when identifying compounds from precursor-mass-filtered ESI-MS/MS spectra (e.g. CASMI category 3 challenges) where candidates must be pulled from a spectral/structure database within a ppm window before in silico prediction and scoring.

## When NOT to use

- The observed spectrum already matches an entry in an experimentally acquired reference spectral library (RSL) — direct library matching is preferable to in silico candidate retrieval and scoring.
- The MS/MS data come from instruments CFM-ID 4.0 does not handle (non-QTOF instruments) and cannot be converted — the article acknowledges the 'inability to handle non-QTOF instruments'.
- The task is EI-MS/MS or spectral prediction alone, with no unknown-compound candidate search — then use the predictor directly rather than the retrieval/scoring loop.

## Inputs

- Experimental precursor-ion ESI-MS/MS spectra with precursor m/z and adduct annotation ([M+H]+ or [M-H]-), e.g. 208 CASMI 2016 category 3 Orbitrap spectra
- Candidate source database: CFM-ID 3.0 spectral database / known-compound structure database
- Mass tolerance parameter (10 ppm)
- Predicted or reference spectra per candidate for scoring (CFM-ID 4.0 MSML model outputs)

## Outputs

- Per-spectrum ranked list of candidate compounds scored by the modified Dot Product function
- Top-hit compound identification per spectrum, reported with and without CFM-ID 3.0 metadata scoring
- Aggregate identification accuracy counts (e.g. correct/total over the 208-compound benchmark)

## How to apply

For each experimental MS/MS spectrum (CASMI 2016 category 3 Orbitrap HCD spectra converted to QTOF collision energies of 10, 20, and 40 eV), take the precursor m/z and the adduct assignment (127 [M+H]+ and 81 [M-H]- in the benchmark) and retrieve candidate structures from CFM-ID 3.0's spectral database using a 10 ppm mass tolerance. Predict ESI-MS/MS spectra for every retrieved candidate with the trained CFM-ID 4.0 MSML model, applying the post-processing that iteratively selects peaks until 80% of total input intensity is reached or 30 peaks are output. Score each candidate against the experimental spectrum using the modified Dot Product function from CFM-ID 2.0, rank candidates, and take the top hit as the identification. Optionally rescore with the CFM-ID 3.0 metadata function (e.g. citation frequency), reporting results both with and without metadata so the effect of database priors is not conflated with spectral-match accuracy.

## Related tools

- **CFM-ID 3.0** (Provides the spectral database from which candidate structures are retrieved at the 10 ppm tolerance, plus the optional metadata scoring function (citation frequency)) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 4.0** (Predicts ESI-MS/MS spectra for each retrieved candidate (MSML model trained on Metlin 2019 QTOF data) so candidates can be scored against the experimental spectrum) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Source of the modified Dot Product scoring function used to rank candidates against the observed spectrum)
- **Docker (wishartlab/cfmid image)** (Deployment vehicle for running the CFM-ID candidate retrieval and spectral prediction pipeline reproducibly) — https://hub.docker.com/r/wishartlab/cfmid
- **Metlin** (Experimental ESI-MS/MS training data (Metlin 2019: 12,165 [M+H]+ spectra / 4055 metabolites; 6120 [M-H]- spectra / 2040 metabolites) behind the predictor that scores retrieved candidates)

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- On the CASMI 2016 category 3 benchmark (208 compounds), the retrieval-plus-scoring loop reproduces 147/208 correct identifications without metadata and 162/208 with metadata.
- Accuracy improvement over the CFM-ID 2.0 baseline reaches ~22.5%, exceeding MS-Finder and SIRIUS 4.
- Every retrieved candidate's precursor mass falls within the 10 ppm mass tolerance of the query spectrum's precursor m/z.
- Ranking uses the modified Dot Product function with spectra post-processed to 80% cumulative intensity or a maximum of 30 peaks.
- Both metadata-on and metadata-off results are reported so citation-frequency priors are not mistaken for spectral-match gains.

## Limitations

- CFM-ID 4.0 (and thus this retrieval-scoring loop) is limited to QTOF-type ESI-MS/MS; non-QTOF instruments require collision-energy conversion, and adapting to other instruments is listed as future work.
- Experimental reference spectral libraries' 'limited coverage is a major bottleneck' — in silico retrieval is needed when no RSL entry exists, and deeper fragmentation graphs add computational complexity with acknowledged runtime headroom.

## Evidence
