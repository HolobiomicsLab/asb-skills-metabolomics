---
name: fingerprint-similarity-scoring
description: Use when ranked candidate lists from cosine-only spectral matching leave
  the true identity below rank 1 (top-1 < 40%) and fingerprint information is available
  to integrate for improved ranking.
when_to_use_negative:
- The analyte's mass spectrum already matches an established reference spectra in
  NIST/Wiley/SWGDRUG libraries — conventional library screening suffices and the fingerprint-integrated
  ranking pipeline is unnecessary
- Input is a raw spectrum with no way to derive a molecular fingerprint (e.g., no
  DeepEI-compatible GC/EI-MS spectrum), so Jaccard similarity cannot be computed
- The task concerns spectral generation for known structures (NEIMS in silico prediction),
  not similarity-based ranking of unknown analytes
tools:
- name: DeepEI
  role: Converts the unknown analyte's mass spectrum into its corresponding fingerprint
    and computes fingerprints of synthetic database compounds (merged into the msp
    file)
- name: rdkit
  role: Python package used for molecular fingerprint handling and similarity computation
    in the detection pipeline
  repo: https://www.rdkit.org/docs/Install.html
- name: NEIMS
  role: Predicts the mass spectra of enumerated NPS database compounds, supplying
    the spectral component that the fingerprint similarity is integrated with
- name: PS2MS
  role: Hosts the drug-detection step that compares spectrum and chemical fingerprint
    and ranks compounds by similarity score
  repo: https://github.com/jhhung/PS2MS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:f1a9570aca2dec509f9d8437f9a0d5d5ab0f6c83af2f9a6abf791c5d8d981c20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - python
  - rdkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fingerprint-similarity-scoring

## Summary

Computes a Jaccard (Tanimoto) similarity between the DeepEI-predicted molecular fingerprint of an unknown GC/EI-MS analyte and the fingerprints of enumerated NPS candidates, and integrates it with the cosine spectral similarity (SMSF) to re-rank candidate compounds.

## When to use

Use when ranked candidate lists from cosine-only spectral matching leave the true identity below rank 1 (top-1 < 40%) and fingerprint information is available to integrate for improved ranking.

## When NOT to use

- The analyte's mass spectrum already matches an established reference spectra in NIST/Wiley/SWGDRUG libraries — conventional library screening suffices and the fingerprint-integrated ranking pipeline is unnecessary
- Input is a raw spectrum with no way to derive a molecular fingerprint (e.g., no DeepEI-compatible GC/EI-MS spectrum), so Jaccard similarity cannot be computed
- The task concerns spectral generation for known structures (NEIMS in silico prediction), not similarity-based ranking of unknown analytes

## Inputs

- Ranked-match output files per test substance: one ranking from cosine-similarity-only (SMSF) and one from the integrated cosine+Jaccard metric
- SWGDRUG ground-truth identities for each test substance (known compound for each analyte)
- DeepEI-predicted fingerprint of the unknown analyte (GC/EI-MS spectrum as input)
- Fingerprints of enumerated synthetic NPS database candidates, merged into the msp file

## Outputs

- Per-substance top-1 correctness flags for cosine-only and cosine+Jaccard rankings
- top1_metric_comparison.csv: side-by-side table with one row per metric and columns for metric name and top-1 proportion
- Re-ranked list of the hundred most similar compounds by integrated similarity score

## How to apply

Use the DeepEI deep learning model to convert the unknown analyte's mass spectrum into its corresponding fingerprint, and obtain fingerprints for the enumerated synthetic NPS database compounds (DeepEI also computes these, and they are merged into the msp file). Compute the Jaccard similarity between analyte and candidate fingerprints and combine it with the cosine-based mass spectral similarity (SMSF) into the integrated similarity metric. Rank the hundred most similar compounds by the integrated score and record which metric (cosine-only vs cosine+Jaccard) places the known ground-truth compound at rank 1. Rationale: the paper reports the proportion of correct predictions at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity on SWGDRUG test substances.

## Related tools

- **DeepEI** (Converts the unknown analyte's mass spectrum into its corresponding fingerprint and computes fingerprints of synthetic database compounds (merged into the msp file))
- **rdkit** (Python package used for molecular fingerprint handling and similarity computation in the detection pipeline) — https://www.rdkit.org/docs/Install.html
- **NEIMS** (Predicts the mass spectra of enumerated NPS database compounds, supplying the spectral component that the fingerprint similarity is integrated with)
- **PS2MS** (Hosts the drug-detection step that compares spectrum and chemical fingerprint and ranks compounds by similarity score) — https://github.com/jhhung/PS2MS

## Examples

```
from rdkit import DataStructs; jac = DataStructs.TanimotoSimilarity(fp_analyte, fp_candidate)  # Jaccard term integrated with cosine SMSF for re-ranking
```

## Evaluation signals

- Cosine-only top-1 proportion of correct predictions falls below 0.40 on the SWGDRUG test substances
- Jaccard-integrated top-1 proportion exceeds 0.50 on the SWGDRUG test substances
- top1_metric_comparison.csv contains exactly one row per metric with metric-name and top-1-proportion columns
- For SMSF ranking, over 85% of SWGDRUG test substances (and over 90% of NIST test substances) rank within the top 20 predictions
- Detection output contains a ranked list of the hundred most similar compounds, ordered by similarity score

## Limitations

- Improvement was demonstrated on SWGDRUG test substances; the metric is supplementary and candidate identity still requires confirmation against reference standards by retention time and mass spectra analysis
- Fingerprint prediction quality depends on the DeepEI model; NEIMS-based in silico prediction is limited to cases where the chemical structure of the NPS is already known
- The fingerprint component only helps when the analyte spectrum fails to match any known reference; it does not replace conventional library screening

## Evidence

- [methods] Top-1 improvement from Jaccard integration: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] DeepEI converts spectrum to fingerprint: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [intro] Integrated similarity ranking of candidates: "The integrated similarity metric between the analyte and enumerated compounds in the NPS database based on the mass spectrum and fingerprint are calculated, summarized, and ranked."
- [readme] Detection step output definition: "The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score."
- [methods] Top-20 recall using SMSF: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
