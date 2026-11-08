---
name: cosine-similarity-metric-interpretation
description: Use when comparing a predicted MS/MS spectrum against a measured spectrum
  (e.g. retrieval-based MS2C ranking with Top-k) and needing to judge whether a cosine
  score reflects genuine spectral agreement rather than artifacts.
when_to_use_negative:
- Spectra come from unmerged acquisitions, different instrument platforms, or precursor
  adducts other than [M+H]+ Orbitrap-HCD — the paper explicitly flags these as unevaluated
  conditions
- The goal is formula annotation for glycans or peptides — the paper excluded these
  and focused on small molecules
- You need absolute mass accuracy guarantees; cosine similarity alone does not certify
  that predicted masses fall within the 10 ppm tolerance of observed peaks
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FraGNNet
  role: Generates the predicted MS/MS spectrum (via recursive bond-breaking fragmentation
    DAG and a Fragment GNN) whose cosine similarity against the measured spectrum
    is then interpreted
  repo: github.com/FraGNNet/fragnnet
- name: Pytorch
  role: Implements the model that produces the spectra being compared; version 2.1
    with CUDA 11.8 per the paper
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: Pytorch Lightning
  role: Trains and evaluates FraGNNet whose output spectra are scored by cosine similarity
  repo: Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:7f0f7677e5179a1737da833420d55c65b45ec61029ffce7182cded79db5825f8
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - NumPy
  - Pandas
  - Python
  - Pytorch
  - Pytorch Lightning
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Cosine similarity metric interpretation

## Summary

Interprets cosine similarity scores between measured and predicted MS/MS spectra (as used in FraGNNet's retrieval-based compound identification) as a match-quality metric, accounting for unexplained peaks, OS/IS support, and adduct/instrument-context caveats.

## When to use

Use when comparing a predicted MS/MS spectrum against a measured spectrum (e.g. retrieval-based MS2C ranking with Top-k) and needing to judge whether a cosine score reflects genuine spectral agreement rather than artifacts.

## When NOT to use

- Spectra come from unmerged acquisitions, different instrument platforms, or precursor adducts other than [M+H]+ Orbitrap-HCD — the paper explicitly flags these as unevaluated conditions
- The goal is formula annotation for glycans or peptides — the paper excluded these and focused on small molecules
- You need absolute mass accuracy guarantees; cosine similarity alone does not certify that predicted masses fall within the 10 ppm tolerance of observed peaks

## Inputs

- Measured MS/MS spectrum (peak list of m/z and intensity) from a merged NIST20 library spectrum
- Predicted MS/MS spectrum from FraGNNet (fragment distribution converted to a spectrum using formula masses)
- Candidate molecule set C_i constructed from the query plus PubChem-sampled molecules ranked by Tanimoto similarity

## Outputs

- Cosine similarity score between measured and predicted spectra
- Top-k retrieval ranking of candidates for MS2C
- Binary IS/OS classification of observed peaks relative to the predicted mass support

## How to apply

In FraGNNet's MS2C retrieval evaluation, predicted spectra are compared against held-out NIST20 MS/MS spectra using cosine similarity over peak intensities, with candidates ranked by Top-k to report retrieval rates. Before interpreting a score, first account for the OS/IS split: masses Outside of the Support (far from any predicted mass) cannot be explained by the model's fragment set, so a high cosine value is meaningful only if most intensity lies Inside the Support. Match predicted formula masses to observed peaks within the paper's 10 ppm tolerance, excluding glycan and peptide spectra and restricting to small molecules. Note that peak-mass accuracy and the probabilistic fragment distribution (not raw cosine) determine whether low scores indicate model failure versus complex chemistry such as cyclizations expanding the fragment search space.

## Related tools

- **FraGNNet** (Generates the predicted MS/MS spectrum (via recursive bond-breaking fragmentation DAG and a Fragment GNN) whose cosine similarity against the measured spectrum is then interpreted) — github.com/FraGNNet/fragnnet
- **Pytorch** (Implements the model that produces the spectra being compared; version 2.1 with CUDA 11.8 per the paper) — https://github.com/Lightning-AI/pytorch-lightning
- **Pytorch Lightning** (Trains and evaluates FraGNNet whose output spectra are scored by cosine similarity) — Lightning-AI/pytorch-lightning

## Examples

```
python -c "import numpy as np; from scipy.spatial.distance import cosine; sim = 1 - cosine(y_true, y_pred)"
```

## Evaluation signals

- Observed peaks classified as OS must lie far from the predicted mass set; if OS intensity is large, a moderate cosine score reflects unexplained peaks (e.g. cyclizations), not metric failure
- For retrieval, cosine ranking should place the true compound within the Top-k cut used in the paper's MS2C evaluation on the NIST20 held-out split
- Formula-to-peak matches used to justify a high score must be within 10 ppm of the associated peak mass
- Glycan and peptide spectra must be excluded before comparing cosine distributions across compound classes

## Limitations

- The fragmentation algorithm fails to account for some fragments, evidenced by unexplained OS peaks; complex chemistry such as cyclizations expands the search space and can depress cosine scores
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts)
- Merged spectra in NIST20 may obscure conditions under which the cosine comparison breaks down

## Evidence

- [methods] Each model was also evaluated in a retrieval-based MS2C task: "Each model was also evaluated in a retrieval-based MS2C task"
- [methods] OS/IS peak definitions: [evidence span withheld — anchor status: not_found]
- [methods] 10 ppm exclusion for formula annotations: "we excluded annotations with formula masses more than 10 ppm away from their associated peak mass"
- [methods] Excluded glycan/peptide spectra: "we excluded glycan and peptide spectra from our formula annotation analysis and focused on small molecules"
- [discussion] Unexplained OS peaks / cyclizations limitation: "These peaks may arise from complex chemistry, such as cyclizations, that dramatically expand the fragment search space and are therefore challenging to model."
- [discussion] Conditions not yet evaluated: "it is important to evaluate performance on unmerged spectra and under a broader range of experimental conditions, including different instrument platforms and precursor adducts."
- [supplementary] Spectral data source: "We exported spectra from the NIST20 MS/MS spectral library"
