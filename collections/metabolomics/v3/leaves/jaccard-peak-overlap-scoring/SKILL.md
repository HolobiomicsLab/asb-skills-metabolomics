---
name: jaccard-peak-overlap-scoring
description: Use when ranking candidate compounds for an unknown GC/EI-MS analyte
  where cosine spectrum similarity alone leaves the true compound below rank 1, and
  DeepEI-predicted fingerprints (msp-merged) are available for the analyte and enumerated
  candidates.
when_to_use_negative:
- When the analyte's mass spectrum already matches an established reference spectra
  in NIST/Wiley/SWGDRUG libraries, since conventional library screening, not integrated
  scoring, applies
- When the candidate structure or fingerprint is unknown/unpredictable — existing
  in silico prediction methods (e.g., NEIMS) are limited to cases where the chemical
  structure of the NPS is already known
- When fingerprints cannot be produced or merged into the msp entries, since the Jaccard
  term has no input
edam_operation: http://edamontology.org/operation_2945
tools:
- name: PS2MS
  role: Orchestrates the integrated similarity scoring and drug-detection step that
    compares analyte spectrum and chemical fingerprint against the synthetic database
    and ranks the hundred most similar compounds
  repo: jhhung/PS2MS
- name: DeepEI
  role: Converts the analyte's mass spectrum into its corresponding fingerprint and
    computes fingerprints of the synthetic-database compounds so Jaccard similarity
    can be calculated
- name: python
  role: Runtime for the deep learning tools and for computing top-1 proportions and
    assembling the comparison table
- name: rdkit
  role: Python package dependency for fingerprint handling in the PS2MS pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:6dc205a5967f60db1bfdc1490af5744e82cab9c19027510abb65ed7b9360c112
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# jaccard-peak-overlap-scoring

## Summary

Integrates Jaccard similarity of molecular fingerprints with cosine/spectral (SMSF) similarity to re-rank candidate matches, raising the proportion of correct top-1 predictions from below 40% to more than 50% on SWGDRUG test substances in the PS2MS workflow.

## When to use

Use when ranking candidate compounds for an unknown GC/EI-MS analyte where cosine spectrum similarity alone leaves the true compound below rank 1, and DeepEI-predicted fingerprints (msp-merged) are available for the analyte and enumerated candidates.

## When NOT to use

- When the analyte's mass spectrum already matches an established reference spectra in NIST/Wiley/SWGDRUG libraries, since conventional library screening, not integrated scoring, applies
- When the candidate structure or fingerprint is unknown/unpredictable — existing in silico prediction methods (e.g., NEIMS) are limited to cases where the chemical structure of the NPS is already known
- When fingerprints cannot be produced or merged into the msp entries, since the Jaccard term has no input

## Inputs

- Ranked-match output files for SWGDRUG test substances from the cosine-only and cosine+Jaccard metrics
- DeepEI-predicted molecular fingerprint of the unknown analyte
- Fingerprints of enumerated candidate compounds merged into the msp file
- SWGDRUG ground-truth identities for test substances

## Outputs

- Ranked list of candidate NPS identities ordered by integrated cosine+Jaccard similarity score (top ~100 most similar compounds)
- top1_metric_comparison.csv — side-by-side table with one row per metric (cosine-only vs cosine+Jaccard) and columns for metric name and top-1 proportion

## How to apply

Use DeepEI to convert the unknown analyte's mass spectrum into its corresponding fingerprint, and merge the fingerprint of each enumerated candidate into its msp entry so both representations are available per match. Compute Jaccard similarity of the binary fingerprint vectors (intersection over union) for analyte–candidate pairs and combine it with the cosine/SMSF spectral similarity to form the integrated similarity metric, then summarize and rank candidates by this combined score. Re-evaluate the ranking against ground truth by flagging a correct prediction when the known SWGDRUG compound appears at rank 1, computing the top-1 proportion separately for cosine-only and cosine+Jaccard rankings. Validate the integration by confirming the cosine-only top-1 proportion falls below 0.40 and the Jaccard-integrated proportion exceeds 0.50, and retain the ranking step that yields the hundred most similar compounds ordered by similarity score.

## Related tools

- **PS2MS** (Orchestrates the integrated similarity scoring and drug-detection step that compares analyte spectrum and chemical fingerprint against the synthetic database and ranks the hundred most similar compounds) — jhhung/PS2MS
- **DeepEI** (Converts the analyte's mass spectrum into its corresponding fingerprint and computes fingerprints of the synthetic-database compounds so Jaccard similarity can be calculated)
- **python** (Runtime for the deep learning tools and for computing top-1 proportions and assembling the comparison table)
- **rdkit** (Python package dependency for fingerprint handling in the PS2MS pipeline)

## Examples

```
from scipy.spatial.distance import jaccard; s_jaccard = 1 - jaccard(fp_analyte, fp_candidate)
```

## Evaluation signals

- Cosine-only top-1 proportion on SWGDRUG test substances falls below 0.40
- Jaccard-integrated top-1 proportion exceeds 0.50
- Output table top1_metric_comparison.csv contains one row per metric with columns metric name and top-1 proportion
- With the SMSF metric, over 85% of SWGDRUG test substances (and over 90% on NIST) rank within the top 20 predictions
- Drug detection output yields a list of the hundred most similar compounds ranked by similarity score

## Limitations

- The approach presumes a core structure for enumeration; it addresses NPS whose spectra do not match any established reference and does not handle fully unknown scaffolds
- The fingerprint term depends on DeepEI prediction quality from the analyte spectrum; no version numbers for DeepEI or other tools are stated
- Reported improvement is stated as a range (below 40% to more than 50%) without exact per-metric values or significance testing

## Evidence

- [methods] Jaccard integration lifts top-1 accuracy: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] DeepEI supplies the analyte fingerprint: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [intro] Integrated metric definition: "The integrated similarity metric between the analyte and enumerated compounds in the NPS database based on the mass spectrum and fingerprint are calculated, summarized, and ranked."
- [readme] README: fingerprint prediction and msp merging: "PS2MS employs DeepEI to predict the fingerprint of the unknown analyte."
- [readme] README: ranked candidate list output: "The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score."
