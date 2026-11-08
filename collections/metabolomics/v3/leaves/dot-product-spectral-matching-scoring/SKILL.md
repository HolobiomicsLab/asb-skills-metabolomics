---
name: dot-product-spectral-matching-scoring
description: Use when matching an unknown ESI-MS/MS spectrum against a set of predicted
  spectra from candidate chemical structures to produce a compound-identification
  ranking (e.g. CASMI challenge submissions or library search with CFM-ID).
when_to_use_negative:
- The spectrum is from a non-QTOF instrument (e.g. Orbitrap collision energy curves)
  unless the energy levels are first converted to the QTOF levels CFM-ID was trained
  on (10, 20, 40 eV).
- The input is a peak table or extracted ion chromatogram rather than a per-fragment
  MS/MS spectrum, since the Dot Product requires an intensity-vector representation
  of the fragmentation pattern.
- Metadata (e.g. citation frequency) is unavailable and the candidates are all chemically
  plausible — the skill will still rank but with lower discrimination.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CFM-ID 4.0
  role: computes Dot Product scores between predicted and experimental spectra and
    performs compound identification
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: source of the modified Dot Product scoring function used for ranking
- name: CFM-ID 3.0
  role: provides candidate structures and optional metadata scoring function (e.g.
    citation frequency)
- name: Docker
  role: container image for running CFM-ID 4.0 scoring pipeline locally
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:60e1e5ddd9ec3029d148bca6bdc1fdecdc6ad3d813683fdb13d98d3315d67d8a
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# dot-product-spectral-matching-scoring

## Summary

A scoring skill that compares an experimentally acquired ESI-MS/MS spectrum against in-silico predicted spectra from candidate structures using the modified Dot Product function from CFM-ID, ranking candidates by spectral similarity to identify the top-hit compound.

## When to use

Use when matching an unknown ESI-MS/MS spectrum against a set of predicted spectra from candidate chemical structures to produce a compound-identification ranking (e.g. CASMI challenge submissions or library search with CFM-ID).

## When NOT to use

- The spectrum is from a non-QTOF instrument (e.g. Orbitrap collision energy curves) unless the energy levels are first converted to the QTOF levels CFM-ID was trained on (10, 20, 40 eV).
- The input is a peak table or extracted ion chromatogram rather than a per-fragment MS/MS spectrum, since the Dot Product requires an intensity-vector representation of the fragmentation pattern.
- Metadata (e.g. citation frequency) is unavailable and the candidates are all chemically plausible — the skill will still rank but with lower discrimination.

## Inputs

- experimental ESI-MS/MS spectrum (e.g. Orbitrap HCD converted to QTOF 10/20/40 eV energy levels)
- list of candidate chemical structures (e.g. retrieved from CFM-ID 3.0 spectral database with 10 ppm mass tolerance)
- predicted in-silico MS/MS spectra for each candidate (from CFM-ID 4.0's MSML model)
- optional metadata per candidate (e.g. citation frequency from CFM-ID 3.0)

## Outputs

- ranked list of candidate compounds with Dot Product similarity scores
- top-hit compound identification per spectrum
- identification accuracy counts (e.g. 147/208 or 162/208 on CASMI 2016)

## How to apply

Generate predicted spectra for each candidate structure across the relevant collision energies (e.g. 10, 20, 40 eV for QTOF data), then post-process each predicted spectrum by iteratively selecting peaks until 80% of the total input intensity is reached or 30 peaks are output. Score each experimental spectrum against the candidate's predicted spectrum with the modified Dot Product function from CFM-ID 2.0, optionally weighting the score with metadata (e.g. citation frequency) from CFM-ID 3.0. Rank all candidates by combined score and select the top-scoring compound as the identification. Validate the ranking against known answer keys (e.g. on the 208 CASMI 2016 category-3 compounds, 147/208 correct without metadata, 162/208 with metadata).

## Related tools

- **CFM-ID 4.0** (computes Dot Product scores between predicted and experimental spectra and performs compound identification) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (source of the modified Dot Product scoring function used for ranking)
- **CFM-ID 3.0** (provides candidate structures and optional metadata scoring function (e.g. citation frequency))
- **Docker** (container image for running CFM-ID 4.0 scoring pipeline locally) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
python cfm_id.py --spectrum unknown.mzML --candidates candidates.sdf --energy-levels 10,20,40 --scoring dot_product
```

## Evaluation signals

- Reproduces reported accuracy on the CASMI 2016 category-3 set: 147/208 correct identifications without metadata, 162/208 with metadata.
- Each candidate's predicted spectrum must be post-processed to ≤30 peaks or ≥80% of total input intensity before scoring.
- Dot Product scores must be computed against spectra generated at the same collision energy levels (10, 20, 40 eV) as the experimental spectrum.
- Ranking must place the true compound at the top hit for a reproducible fraction of benchmark compounds matching published numbers.
- Adding the CFM-ID 3.0 metadata score should increase the number of correct identifications (e.g. 147→162 on CASMI 2016).

## Limitations

- Pure in silico approaches correctly identified only about 25% of testing compounds without metadata help in earlier comparisons, so accuracy depends heavily on candidate-set quality.
- Limited coverage of experimental reference spectral libraries is a major bottleneck since they contain spectra for only a few thousand compounds.
- Deeper fragmentation graphs introduce extra computational complexity, and runtime still has room for improvement.

## Evidence

- [methods] 147/208 correct identifications without metadata: [evidence span withheld — anchor status: not_found]
- [results] 162/208 with metadata: [evidence span withheld — anchor status: not_found]
- [methods] post-processing that iteratively selects peaks until 80% of total input intensity: [evidence span withheld — anchor status: not_found]
- [methods] collision energy conversion for QTOF levels: [evidence span withheld — anchor status: not_found]
- [methods] candidate retrieval from CFM-ID 3.0 spectral database: [evidence span withheld — anchor status: not_found]
