---
name: msml-model-training-on-qtof-spectra
description: Use when training CFM-ID 4.0's learned (MSML) predictor on QTOF ESI-MS/MS
  spectra at 10/20/40 eV, or when benchmarking in silico compound identification (MS2C)
  on CASMI 2016 category-3 Orbitrap/QTOF data.
when_to_use_negative:
- Input spectra were collected on instruments other than QTOF instruments — the authors
  note they plan to adapt CFM-ID to handle other instruments, so Orbitrap/QQQ data
  must first be converted or handled differently
- The task is EI-MS/MS (electron ionization) spectral prediction — the MSML/QTOF training
  pipeline described here covers ESI-MS/MS, not EI
- Only a handful of training spectra are available for the compound classes of interest
  — the learned predictor needs large Metlin-scale training sets; hand-written rule-based
  prediction may be the better route
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CFM-ID 4.0
  role: Hosts the MSML machine-learning predictor trained on QTOF Metlin spectra;
    used to predict ESI-MS/MS spectra and score/rank candidate compounds
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Distribution mechanism for running CFM-ID 4.0's training and prediction pipeline
    reproducibly
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: Metlin
  role: Source of the experimental QTOF ESI-MS/MS training data for the [M+H]+ and
    [M-H]- MSML models
- name: CFM-ID 2.0
  role: Baseline whose modified Dot Product scoring function is reused for candidate
    ranking, and whose identification accuracy (CASMI 2016) is the reference for the
    reported 22.5% improvement
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:1a0f555eba381e3a6ca3ada15ec60940751ee6683631946cc66a649fb6f88582
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# msml-model-training-on-qtof-spectra

## Summary

Train CFM-ID 4.0's MSML machine-learning spectral predictor on QTOF ESI-MS/MS data (Metlin 2019 [M+H]+ and [M-H]- sets) using an adjacency/connectivity-matrix molecular representation, then use the trained models to predict spectra and rank candidate compounds for identification benchmarks such as CASMI 2016.

## When to use

Use when training CFM-ID 4.0's learned (MSML) predictor on QTOF ESI-MS/MS spectra at 10/20/40 eV, or when benchmarking in silico compound identification (MS2C) on CASMI 2016 category-3 Orbitrap/QTOF data.

## When NOT to use

- Input spectra were collected on instruments other than QTOF instruments — the authors note they plan to adapt CFM-ID to handle other instruments, so Orbitrap/QQQ data must first be converted or handled differently
- The task is EI-MS/MS (electron ionization) spectral prediction — the MSML/QTOF training pipeline described here covers ESI-MS/MS, not EI
- Only a handful of training spectra are available for the compound classes of interest — the learned predictor needs large Metlin-scale training sets; hand-written rule-based prediction may be the better route

## Inputs

- QTOF ESI-MS/MS training spectra at 10, 20, and 40 eV (Metlin 2019 [M+H]+: 12,165 spectra / 4,055 metabolites; Metlin 2019 [M-H]-: 6,120 spectra / 2,040 metabolites)
- Molecular structures of training compounds represented as adjacency/connectivity matrices (features capped at 10 atoms)
- Experimental test spectra to identify (e.g. 208 CASMI 2016 category-3 spectra: 127 positive [M+H]+, 81 negative [M-H]-)
- Candidate structures retrieved from the CFM-ID 3.0 spectral database within a 10 ppm mass tolerance

## Outputs

- Trained MSML neural-network models, one per adduct ([M+H]+ and [M-H]-)
- Predicted ESI-MS/MS spectra (10/20/40 eV) for each candidate structure, post-processed to 80% cumulative intensity or 30 peaks
- Ranked candidate list per input spectrum with top-hit compound identification and accuracy metrics (e.g. 147/208 without metadata, 162/208 with metadata on CASMI 2016)

## How to apply

Convert experimental spectra (e.g. Orbitrap HCD) to the three QTOF collision energy levels (10, 20, 40 eV) using the Thermo Fisher Scientific conversion equation, splitting each compound into per-energy spectra for the [M+H]+ or [M-H]- adduct. Train the MSML model separately for each adduct on the Metlin 2019 sets — [M+H]+: 12,165 spectra / 4,055 metabolites; [M-H]-: 6,120 spectra / 2,040 metabolites — using a fully connected neural network with two hidden layers of 128 nodes and connectivity-matrix features capped at 10 atoms. The adjacency-matrix representation encodes chemical bond topology so bond-dissociation parameters are learned from QTOF data rather than hand-coded rules. At prediction time, generate the fragmentation graph for each candidate structure (retrieved from the CFM-ID 3.0 spectral database within a 10 ppm mass tolerance), predict spectra for each energy level, and post-process iteratively selecting peaks until 80% of total input intensity is reached or 30 peaks are output. Score candidates against the experimental spectrum with the modified Dot Product function from CFM-ID 2.0, rank compounds, and optionally repeat with the CFM-ID 3.0 metadata scoring function (e.g. citation frequency) to quantify the metadata-free gain.

## Related tools

- **CFM-ID 4.0** (Hosts the MSML machine-learning predictor trained on QTOF Metlin spectra; used to predict ESI-MS/MS spectra and score/rank candidate compounds) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Distribution mechanism for running CFM-ID 4.0's training and prediction pipeline reproducibly) — https://hub.docker.com/r/wishartlab/cfmid
- **Metlin** (Source of the experimental QTOF ESI-MS/MS training data for the [M+H]+ and [M-H]- MSML models)
- **CFM-ID 2.0** (Baseline whose modified Dot Product scoring function is reused for candidate ranking, and whose identification accuracy (CASMI 2016) is the reference for the reported 22.5% improvement)

## Examples

```
docker pull wishartlab/cfmid:latest
```

## Evaluation signals

- Trained models reproduce the CASMI 2016 category-3 benchmark counts: 147/208 correct identifications without metadata and 162/208 with metadata
- Identification accuracy shows the reported 22.5% improvement over CFM-ID 2.0 and outperforms MS-Finder and SIRIUS 4 on the same 208-spectrum set
- Training-set statistics match the paper: 12,165 [M+H]+ spectra / 4,055 metabolites and 6,120 [M-H]- spectra / 2,040 metabolites from Metlin 2019
- Predicted spectra obey the post-processing invariant: peak selection stops when 80% of total input intensity is reached or 30 peaks are output
- Network architecture check: fully connected model with exactly two hidden layers of 128 nodes and connectivity-matrix features capped at 10 atoms

## Limitations

- CFM-ID 4.0's learned QTOF models do not yet handle MS spectra collected on instruments other than QTOF instruments
- Runtime and accuracy still have room for further improvement, and deeper fragmentation graphs introduce extra computational complexity
- Pure in silico approaches without metadata help correctly identify only a limited fraction of testing compounds (~25% reported historically), so metadata (e.g. citation frequency) materially affects identification counts

## Evidence
