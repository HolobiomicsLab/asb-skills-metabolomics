---
name: msml-neural-network-retraining
description: Use when observed MS/MS spectra come from a non-QTOF instrument (e.g.
  Orbitrap HCD) and the stock CFM-ID 4.0 QTOF-trained MSML model must be recalibrated
  before in silico spectral prediction and candidate ranking.
when_to_use_negative:
- When the query spectra are already QTOF ESI-MS/MS data — the published CFM-ID 4.0
  model trained on QTOF Metlin data is directly applicable and retraining is unnecessary
- When EI-MS/MS spectral prediction is the goal — this skill concerns ESI-MS/MS MSML
  retraining, not CFM-ID 4.0's EI prediction improvements
- When compounds fall in classes covered by CFM-ID 4.0's enhanced hand-written rule-based
  predictor (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid
  glycosides) — rule-based prediction does not require neural network retraining
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: Provides the MSML neural network architecture and training pipeline that is
    retrained on the new instrument's data; also performs spectral prediction and
    candidate ranking via the modified Dot Product function
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 3.0
  role: Supplies the spectral database from which candidate structures are retrieved
    with a 10 ppm mass tolerance for the identification benchmark
- name: Docker
  role: Distribution mechanism for running the retrained CFM-ID 4.0 instance (wishartlab/cfmid
    image)
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:e63e1cf8190f6c4f0c99582d139cb97c0111c24cf168288414ee8c67ddbd3b6c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# msml-neural-network-retraining

## Summary

Retrains CFM-ID 4.0's MSML neural network on instrument-specific ESI-MS/MS training data (e.g. Orbitrap HCD instead of QTOF Metlin spectra) to improve compound identification on spectra from that instrument type, using the paper's CASMI 2016 category 3 benchmark as the evaluation harness.

## When to use

Use when observed MS/MS spectra come from a non-QTOF instrument (e.g. Orbitrap HCD) and the stock CFM-ID 4.0 QTOF-trained MSML model must be recalibrated before in silico spectral prediction and candidate ranking.

## When NOT to use

- When the query spectra are already QTOF ESI-MS/MS data — the published CFM-ID 4.0 model trained on QTOF Metlin data is directly applicable and retraining is unnecessary
- When EI-MS/MS spectral prediction is the goal — this skill concerns ESI-MS/MS MSML retraining, not CFM-ID 4.0's EI prediction improvements
- When compounds fall in classes covered by CFM-ID 4.0's enhanced hand-written rule-based predictor (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides) — rule-based prediction does not require neural network retraining

## Inputs

- Instrument-specific experimental ESI-MS/MS training spectra (e.g. Orbitrap HCD data from the Metlin database, in place of QTOF Metlin data)
- CASMI 2016 category 3 challenge dataset: 208 Orbitrap MS/MS spectra (127 [M+H]+, 81 [M-H]-)
- Candidate chemical structures retrieved from CFM-ID 3.0's spectral database within a 10 ppm mass tolerance
- CFM-ID source code from https://bitbucket.org/wishartlab/cfm-id-code/ (MSML training configuration: 2 hidden layers x 128 nodes, fragmentation graph depth 2, connectivity matrix capped at 10 atoms)

## Outputs

- Retrained MSML neural network model parameters matched to the target instrument type
- Predicted in silico MS/MS spectra for each candidate compound at 10, 20, and 40 eV collision energies
- Ranked candidate list per query spectrum scored with the modified Dot Product function
- Compound identification count out of 208, tabulated against the QTOF-trained baseline of 147/208

## How to apply

Train the CFM-ID 4.0 MSML neural network — two hidden layers of 128 nodes each, fragmentation graph depth 2, connectivity matrix features capped at 10 atoms — on ESI-MS/MS data from the target instrument type instead of the QTOF Metlin data used in the paper. For evaluation, download the CASMI 2016 category 3 dataset (208 Orbitrap MS/MS spectra: 127 [M+H]+ and 81 [M-H]-) and retrieve candidate structures from CFM-ID 3.0's spectral database with a 10 ppm mass tolerance. Since the model is now instrument-matched but the benchmark scoring pipeline is QTOF-oriented, map Orbitrap HCD spectra to QTOF 10/20/40 eV collision energies with the Thermo Fisher Scientific conversion equation before predicting in silico MS/MS spectra for all candidates with the retrained MSML model. Score each observed spectrum against all candidates using the modified Dot Product function introduced by CFM-ID 2.0, rank candidates, and count correct identifications out of 208 against the QTOF-trained baseline of 147/208.

## Related tools

- **CFM-ID 4.0** (Provides the MSML neural network architecture and training pipeline that is retrained on the new instrument's data; also performs spectral prediction and candidate ranking via the modified Dot Product function) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 3.0** (Supplies the spectral database from which candidate structures are retrieved with a 10 ppm mass tolerance for the identification benchmark)
- **Docker** (Distribution mechanism for running the retrained CFM-ID 4.0 instance (wishartlab/cfmid image)) — https://hub.docker.com/r/wishartlab/cfmid

## Evaluation signals

- The comparison table records both the retrained-model identification count and the paper's QTOF-trained baseline of 147/208 on CASMI 2016 category 3
- Identification counts fall in a plausible 0-208 range with no value fabricated beyond empirical evaluation
- Candidate structures were retrieved with exactly a 10 ppm mass tolerance from CFM-ID 3.0's spectral database, matching the paper's evaluation setup
- Predicted spectra were generated at all three QTOF collision energies (10, 20, 40 eV) after Thermo Fisher conversion of Orbitrap HCD spectra
- Ranking used the modified Dot Product function from CFM-ID 2.0, not raw peak matching

## Limitations

- The published CFM-ID 4.0 was trained only on QTOF ESI-MS/MS data from Metlin; the paper explicitly flags 'we plan to adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments', so Orbitrap support requires the retraining described here rather than a supported feature
- Mapping Orbitrap HCD spectra to QTOF 10/20/40 eV collision energies via the Thermo Fisher conversion equation is an approximation that may limit achievable identification gains
- Deeper fragmentation graphs introduce extra computational complexity, and the authors note 'there is still room for further improvement, both in terms of accuracy and runtime'
- Without metadata, in silico approaches historically 'could only correctly identify about 25% of the testing compounds', so absolute counts should be interpreted with that ceiling in mind

## Evidence
