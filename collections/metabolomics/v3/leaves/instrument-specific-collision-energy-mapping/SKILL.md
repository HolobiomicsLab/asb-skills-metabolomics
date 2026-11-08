---
name: instrument-specific-collision-energy-mapping
description: Use when evaluating a QTOF-trained MS/MS spectral prediction model (e.g.
  CFM-ID 4.0 trained on Metlin QTOF data) against Orbitrap MS/MS query spectra, such
  as the CASMI 2016 category 3 dataset of 208 Orbitrap spectra.
when_to_use_negative:
- The query spectra were already collected on a QTOF instrument, so no cross-instrument
  energy mapping is needed and predictions can be requested at QTOF 10/20/40 eV directly.
- A model trained on the target instrument's data (e.g. an Orbitrap-trained MSML model)
  is available; direct prediction at native energies is preferable to energy conversion.
- The spectra are EI-MS/MS rather than ESI-MS/MS; the CFM-ID ESI pipeline and QTOF/HCD
  energy mapping do not apply.
edam_topics:
- http://edamontology.org/topic_3521
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: QTOF-trained MSML prediction model whose 10/20/40 eV output levels receive
    the Orbitrap-mapped collision energies; run via web server or Docker
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: CFM-ID 3.0
  role: Source of the spectral database from which candidate structures are retrieved
    at a 10 ppm mass tolerance for scoring
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Modified Dot Product scoring function used to compare mapped-energy predicted
    spectra against the Orbitrap query spectra
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:df4d4625fa50363ecc91b6cc430ff83cec1ccc23be622cbac2ae046f43a34bdc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3521
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# instrument-specific-collision-energy-mapping

## Summary

Maps collision energies from one MS instrument family to another (e.g. Orbitrap HCD to nominal QTOF 10/20/40 eV collision energies) using the Thermo Fisher Scientific conversion equation, so that spectral-prediction models trained exclusively on QTOF data (such as CFM-ID 4.0) can be scored against spectra acquired on a different instrument.

## When to use

Use when evaluating a QTOF-trained MS/MS spectral prediction model (e.g. CFM-ID 4.0 trained on Metlin QTOF data) against Orbitrap MS/MS query spectra, such as the CASMI 2016 category 3 dataset of 208 Orbitrap spectra.

## When NOT to use

- The query spectra were already collected on a QTOF instrument, so no cross-instrument energy mapping is needed and predictions can be requested at QTOF 10/20/40 eV directly.
- A model trained on the target instrument's data (e.g. an Orbitrap-trained MSML model) is available; direct prediction at native energies is preferable to energy conversion.
- The spectra are EI-MS/MS rather than ESI-MS/MS; the CFM-ID ESI pipeline and QTOF/HCD energy mapping do not apply.

## Inputs

- Orbitrap MS/MS (HCD) query spectra, e.g. the CASMI 2016 category 3 dataset (208 spectra: 127 [M+H]+, 81 [M-H]-)
- Precursor m/z values and candidate compound structures (retrievable via CFM-ID 3.0's spectral database within a 10 ppm mass tolerance)
- Instrument manufacturer energy-conversion specification (Thermo Fisher Scientific conversion equation) for HCD collision energies

## Outputs

- Per-query mapping from Orbitrap HCD collision energies to nominal QTOF 10/20/40 eV collision energies
- In silico predicted MS/MS spectra for each candidate at the mapped QTOF energies
- Ranked candidate lists with modified Dot Product scores and a correct-identification count out of 208 for comparison against the 147/208 QTOF-trained baseline

## How to apply

First confirm the instrument mismatch: CFM-ID 4.0's MSML neural network is trained on Metlin QTOF data, while the query spectra (e.g. CASMI 2016 category 3: 208 Orbitrap MS/MS spectra, 127 [M+H]+ and 81 [M-H]-) were collected on an Orbitrap. For each Orbitrap HCD spectrum, apply the Thermo Fisher Scientific conversion equation to obtain the equivalent QTOF collision energies at which predictions are requested, i.e. the 10, 20 and 40 eV levels CFM-ID predicts at. Predict in silico spectra for all candidate structures at those mapped energies with the QTOF-trained model, retrieve candidates within a 10 ppm mass tolerance (e.g. from CFM-ID 3.0's spectral database), and score observed vs. predicted spectra with CFM-ID 2.0's modified Dot Product function to rank candidates. The rationale is that instrument-class-specific energy calibration lets a single QTOF-trained model remain usable on Orbitrap data rather than requiring a full retraining on Orbitrap MS/MS data.

## Related tools

- **CFM-ID 4.0** (QTOF-trained MSML prediction model whose 10/20/40 eV output levels receive the Orbitrap-mapped collision energies; run via web server or Docker) — https://hub.docker.com/r/wishartlab/cfmid
- **CFM-ID 3.0** (Source of the spectral database from which candidate structures are retrieved at a 10 ppm mass tolerance for scoring) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Modified Dot Product scoring function used to compare mapped-energy predicted spectra against the Orbitrap query spectra) — https://bitbucket.org/wishartlab/cfm-id-code/

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- A comparison table records both the mapped result and the paper's QTOF-trained baseline of 147/208 correct identifications on CASMI 2016.
- Identification counts fall in a plausible 0-208 range, with no fabricated increase beyond what the empirical evaluation actually produced.
- Every Orbitrap spectrum's collision energies were converted via the Thermo Fisher Scientific equation to the three nominal QTOF levels (10/20/40 eV) before prediction.
- Candidate retrieval for each query used a 10 ppm precursor mass tolerance against CFM-ID 3.0's spectral database, matching the paper's evaluation protocol.
- Candidate ranking used CFM-ID 2.0's modified Dot Product scoring function, not raw cosine or other similarity measures.

## Limitations

- CFM-ID 4.0's learned model was trained on QTOF Metlin data only; the authors explicitly note the inability to handle non-QTOF instruments and state 'we plan to adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments', so energy mapping is an approximation rather than a native fix.
- Deeper fragmentation graphs introduce extra computational complexity, and the authors acknowledge room for further improvement in accuracy and runtime.
- Without metadata assistance, pure in silico approaches could only correctly identify about 25% of testing compounds historically, so absolute identification counts may remain modest even after careful energy mapping.

## Evidence

- [other] use the Thermo Fisher Scientific conversion equation to map Orbitrap HCD spectra to QTOF 10/20/40 eV collision energies: [evidence span withheld — anchor status: not_found]
- [other] CFM-ID 4.0, trained only on QTOF data, correctly identified 147 of 208 CASMI 2016 spectra without metadata: [evidence span withheld — anchor status: not_found]
