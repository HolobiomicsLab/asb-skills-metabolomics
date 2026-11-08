---
name: casmi2016-compound-identification-scoring
description: Use when benchmarking an MS-to-compound (MS2C) identification method
  on the 208-spectrum CASMI 2016 category 3 Orbitrap dataset, or when a QTOF-trained
  model needs an instrument-transfer evaluation against the published 147/208 baseline.
when_to_use_negative:
- When the query spectra come from instruments other than QTOF or Orbitrap HCD and
  no energy-conversion step (e.g., the Thermo Fisher Scientific conversion equation)
  can be applied — the models are instrument-specific
- When experimental reference spectral library (RSL) matching is available, since
  this skill targets in silico compound-to-MS matching rather than direct library
  search
- When metadata (e.g., molecular formula or retention time) is intended to assist
  identification, which would break comparability with the metadata-free category
  3 benchmark
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CFM-ID 4.0
  role: Predicts in silico ESI-MS/MS spectra for candidate structures and ranks them
    against the query spectra for compound identification on the CASMI 2016 benchmark
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 3.0
  role: Source of the spectral database from which candidate structures are retrieved
    at 10 ppm mass tolerance for each CASMI query spectrum
- name: Docker
  role: Distribution mechanism for running CFM-ID 4.0 via the wishartlab/cfmid image
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:73f16050681ab4c1378354cb6adb64d4cdd3c47a25304cd2fe61b87a5b62e84f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# casmi2016-compound-identification-scoring

## Summary

Evaluates in silico MS/MS-based compound identification on the CASMI 2016 challenge (category 3) benchmark, where CFM-ID 4.0 correctly identified 147 of 208 Orbitrap spectra without metadata. Provides a fixed, reproducible protocol: candidate retrieval, spectral prediction, modified Dot Product scoring, and ranked identification counting.

## When to use

Use when benchmarking an MS-to-compound (MS2C) identification method on the 208-spectrum CASMI 2016 category 3 Orbitrap dataset, or when a QTOF-trained model needs an instrument-transfer evaluation against the published 147/208 baseline.

## When NOT to use

- When the query spectra come from instruments other than QTOF or Orbitrap HCD and no energy-conversion step (e.g., the Thermo Fisher Scientific conversion equation) can be applied — the models are instrument-specific
- When experimental reference spectral library (RSL) matching is available, since this skill targets in silico compound-to-MS matching rather than direct library search
- When metadata (e.g., molecular formula or retention time) is intended to assist identification, which would break comparability with the metadata-free category 3 benchmark

## Inputs

- CASMI 2016 category 3 Orbitrap MS/MS spectra (208 total: 127 [M+H]+, 81 [M-H]-)
- Candidate chemical structures retrieved from the CFM-ID 3.0 spectral database using the precursor m/z and a 10 ppm mass tolerance

## Outputs

- Ranked candidate compound list per query spectrum, scored by the modified Dot Product function
- Aggregate identification count out of 208, comparable to the reported 147/208 CFM-ID 4.0 baseline

## How to apply

Download the CASMI 2016 category 3 dataset (208 Orbitrap MS/MS spectra: 127 [M+H]+ and 81 [M-H]-) and retrieve candidate structures from CFM-ID 3.0's spectral database with a 10 ppm mass tolerance. For each of the 208 spectra, predict in silico MS/MS spectra (10/20/40 eV collision energies) for every candidate using CFM-ID 4.0's MSML network (two hidden layers of 128 nodes, fragmentation graph depth 2, connectivity matrix features capped at 10 atoms); if the model was trained on QTOF data but the query spectra are Orbitrap HCD, first map collision energies using the Thermo Fisher Scientific conversion equation. Score each experimental spectrum against all candidates with the modified Dot Product function introduced by CFM-ID 2.0 and rank candidates to make the identification call. Count correct identifications out of 208 and compare against the QTOF-trained baseline of 147/208, judging any retrained or modified pipeline by its reported count and its position relative to that baseline. The evaluation must exclude metadata assistance so results remain comparable to the paper's category 3 setting.

## Related tools

- **CFM-ID 4.0** (Predicts in silico ESI-MS/MS spectra for candidate structures and ranks them against the query spectra for compound identification on the CASMI 2016 benchmark) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 3.0** (Source of the spectral database from which candidate structures are retrieved at 10 ppm mass tolerance for each CASMI query spectrum)
- **Docker** (Distribution mechanism for running CFM-ID 4.0 via the wishartlab/cfmid image) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- The comparison table records both the new pipeline's identification count and the published QTOF-trained baseline of 147/208
- The reported identification count falls in a plausible 0-208 range and is not fabricated beyond the empirical evaluation
- Candidate lists were retrieved with a 10 ppm mass tolerance against the CFM-ID 3.0 spectral database, matching the paper's evaluation
- The correct number of query spectra (208 total: 127 [M+H]+ and 81 [M-H]-) is processed, with no metadata used during identification
- Scoring is performed with the modified Dot Product function of CFM-ID 2.0, not an ad hoc similarity metric

## Limitations

- CFM-ID 4.0 was trained on QTOF Metlin data; the authors explicitly note plans to adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments
- In silico approaches could only correctly identify about 25% of testing compounds without the help of metadata, so absolute identification counts on hard categories may be low
- Runtime remains a limitation, and deeper fragmentation graphs introduce extra computational complexity

## Evidence
