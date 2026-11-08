---
name: metadata-augmented-compound-ranking
description: Use when ranking candidate structures retrieved from a spectral database
  against an experimental MS/MS spectrum and optional metadata (e.g. compound citation
  frequency) is available to boost identification accuracy beyond spectrum-only dot-product
  scoring.
when_to_use_negative:
- No candidate structures can be retrieved from the CFM-ID 3.0 spectral database within
  the 10 ppm mass tolerance, so there is nothing to rank or metadata-score
- The spectra were acquired on instruments other than QTOF — CFM-ID's ESI-MS/MS models
  are trained and validated on QTOF collision-energy data, and the authors note they
  'plan to adapt CFM-ID to handle MS spectra collected using instruments other than
  QTOF instruments'
- The goal is spectral prediction (compound-to-MS) rather than spectrum-to-compound
  identification ranking; use the prediction workflow instead of the metadata ranking
  workflow
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: Predicts ESI-MS/MS spectra for each candidate structure (MSML model) whose
    spectra are then scored and ranked against the experimental spectrum
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: CFM-ID 3.0
  role: Provides the metadata scoring function (e.g. citation frequency) applied on
    top of the spectral similarity score, and the spectral database used for 10 ppm
    candidate retrieval
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Source of the modified Dot Product function used to score each candidate against
    the experimental spectrum
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:da88d96dc8734174893ca7b39eab133c60ecdc8b726ed79fc1683176ad0385da
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# metadata-augmented-compound-ranking

## Summary

Ranks candidate compounds identified from ESI-MS/MS spectra by combining CFM-ID's modified Dot Product spectral similarity score with the CFM-ID 3.0 metadata scoring function (e.g. citation frequency), the configuration under which CFM-ID identified 162/208 CASMI 2016 category-3 compounds versus 147/208 without metadata.

## When to use

Use when ranking candidate structures retrieved from a spectral database against an experimental MS/MS spectrum and optional metadata (e.g. compound citation frequency) is available to boost identification accuracy beyond spectrum-only dot-product scoring.

## When NOT to use

- No candidate structures can be retrieved from the CFM-ID 3.0 spectral database within the 10 ppm mass tolerance, so there is nothing to rank or metadata-score
- The spectra were acquired on instruments other than QTOF — CFM-ID's ESI-MS/MS models are trained and validated on QTOF collision-energy data, and the authors note they 'plan to adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments'
- The goal is spectral prediction (compound-to-MS) rather than spectrum-to-compound identification ranking; use the prediction workflow instead of the metadata ranking workflow

## Inputs

- Experimental ESI-MS/MS spectra with precursor m/z and adduct type ([M+H]+ or [M-H]-), e.g. CASMI 2016 category-3 Orbitrap HCD spectra converted to QTOF collision energies (10, 20, 40 eV)
- Candidate chemical structures retrieved from the CFM-ID 3.0 spectral database within a 10 ppm mass tolerance
- Predicted spectra per candidate from the CFM-ID 4.0 MSML model (post-processed to 80% cumulative intensity or max 30 peaks)
- Compound metadata such as citation frequency used by the CFM-ID 3.0 metadata scoring function

## Outputs

- Ranked list of candidate compounds per input spectrum with combined spectral + metadata scores
- Top-hit compound identification per spectrum
- Identification accuracy counts with and without metadata (e.g. 162/208 vs 147/208 on CASMI 2016)

## How to apply

First generate candidate structures by searching the CFM-ID 3.0 spectral database within a 10 ppm mass tolerance of the precursor m/z, then use the trained MSML model of CFM-ID 4.0 to predict QTOF ESI-MS/MS spectra (10, 20, 40 eV) for each candidate, post-processed by iteratively selecting peaks until 80% of total input intensity is reached or 30 peaks are output. Score each candidate against the experimental spectrum with the modified Dot Product function from CFM-ID 2.0 and combine this spectral score with the metadata scoring function carried over from CFM-ID 3.0, which rewards compounds with higher citation frequency. Rank candidates by the combined score and take the top hit as the identification. Because metadata changes rankings, always compute accuracy both with and without metadata to quantify its contribution (on CASMI 2016: 147/208 without metadata vs 162/208 with metadata).

## Related tools

- **CFM-ID 4.0** (Predicts ESI-MS/MS spectra for each candidate structure (MSML model) whose spectra are then scored and ranked against the experimental spectrum) — https://hub.docker.com/r/wishartlab/cfmid
- **CFM-ID 3.0** (Provides the metadata scoring function (e.g. citation frequency) applied on top of the spectral similarity score, and the spectral database used for 10 ppm candidate retrieval) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Source of the modified Dot Product function used to score each candidate against the experimental spectrum) — https://bitbucket.org/wishartlab/cfm-id-code/

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Identification counts reproduce the reported benchmarks: 147/208 correct without metadata and 162/208 correct with metadata on CASMI 2016 category 3
- Metadata-augmented ranking improves over spectrum-only ranking (22.5% accuracy improvement over CFM-ID 2.0 is the reported baseline delta)
- Candidate retrieval step returns non-empty candidate sets under the 10 ppm mass tolerance for each input precursor
- Predicted spectra satisfy the post-processing invariant: peak selection stops once 80% of total input intensity is reached or 30 peaks have been output
- Ablation shows accuracy computed with metadata exceeds accuracy computed without metadata (162/208 vs 147/208)

## Limitations

- Inability to handle non-QTOF instruments: the ESI-MS/MS models are trained on QTOF data, and adapting to other instruments is future work
- Runtime and computational complexity remain open issues; deeper fragmentation graphs 'introduce extra computational complexity'
- Metadata scoring depends on compound citation frequency, which biases ranking toward well-studied compounds; without metadata accuracy drops (147/208 vs 162/208)

## Evidence

- [methods] Score each candidate against the experimental spectrum with the modified Dot Product function from CFM-ID 2.0 and rank compounds to identify the top hit.: [evidence span withheld — anchor status: not_found]
- [methods] Compute identification accuracy with and without the CFM-ID 3.0 metadata scoring function (e.g. citation frequency).: [evidence span withheld — anchor status: not_found]
- [results] On the CASMI 2016 set, CFM-ID 4.0 without metadata correctly identified 147 of 208 compounds: [evidence span withheld — anchor status: not_found]
- [results] with metadata it identified 162 of 208.: [evidence span withheld — anchor status: not_found]
- [methods] Retrieve candidate chemical structures for each spectrum from CFM-ID 3.0's spectral database using a 10 ppm mass tolerance.: [evidence span withheld — anchor status: not_found]
