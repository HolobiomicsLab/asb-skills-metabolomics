---
name: tanimoto-structural-similarity-scoring
description: Use when evaluating entity-level annotation quality of LC-MS feature
  prioritization results and false positives must be graded by chemical proximity
  rather than counted as uniform errors.
when_to_use_negative:
- When evaluating the pure matching stage rather than the diffusion-based prioritization
  stage, since the Tanimoto ratio grades near-miss annotations of prioritized candidates,
  not mass-to-charge matching outcomes.
- When no chemical structure information is available for the proposed candidates,
  since the ratio requires structure-based similarity computation rather than only
  hit counts.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: mWISE
  role: Produces the prioritized per-peak candidate lists whose annotation quality
    is graded via Tanimoto structural similarity against reference compounds
  repo: https://github.com/b2slab/mWISE
- name: R
  role: Language environment in which mWISE is implemented and the Tanimoto structural
    similarity scoring is performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:2164b04bbcf74dbc31dcc1d708335bc871448768eccc536f56925cca6e74dc5a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  repo_url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - R
  - mWISE
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tanimoto-structural-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the Tanimoto ratio between proposed and original compound structures to quantify how chemically close mWISE's prioritized annotation candidates are to true LC-MS metabolite identities, beyond simple hit/miss counts.

## When to use

Use when evaluating entity-level annotation quality of LC-MS feature prioritization results and false positives must be graded by chemical proximity rather than counted as uniform errors.

## When NOT to use

- When evaluating the pure matching stage rather than the diffusion-based prioritization stage, since the Tanimoto ratio grades near-miss annotations of prioritized candidates, not mass-to-charge matching outcomes.
- When no chemical structure information is available for the proposed candidates, since the ratio requires structure-based similarity computation rather than only hit counts.

## Inputs

- Per-assay mWISE prioritized candidate lists (top three candidates per peak) from the FELLA + raw-score diffusion configuration
- Reference compound sets per assay defining true positives, false positives, and false negatives
- Chemical structure representations (e.g. structural fingerprints or molecular structures) for proposed and original compounds

## Outputs

- Per-assay Tanimoto ratio summarizing chemical closeness of proposed versus original compounds
- Entity-level metrics table rows (TP, TN, FP, FN, ref FP, references, sensitivity, specificity, precision, NPV, F1, Tanimoto ratio) per assay

## How to apply

Take the top three candidates per peak selected by the mWISE pipeline (FELLA network with raw-score diffusion, binary input, unique annotation option) and load the reference compound set for each assay to define true positives, false positives, and false negatives. For every proposed candidate that is a false positive, compute the Tanimoto similarity between its chemical structure and the original reference compound using the same structure-based scheme reported for mWISE. Aggregate these similarities into a per-assay Tanimoto ratio summarizing the chemical closeness of proposed versus original compounds. Report the ratio alongside the entity-level TP/FP/FN table (e.g. Assay 6: 53 TP, 90 FP, 7852 TN, 21 FN, F1 = 0.49) so that a high FP count accompanied by a high Tanimoto ratio is interpreted as structurally near-miss annotations rather than noise. Validate the ratio against the article's reported values, e.g. Tanimoto ratio 0.69 for Assay 1 and 0.61 for Assay 3 under the FELLA + raw score configuration.

## Related tools

- **mWISE** (Produces the prioritized per-peak candidate lists whose annotation quality is graded via Tanimoto structural similarity against reference compounds) — https://github.com/b2slab/mWISE
- **R** (Language environment in which mWISE is implemented and the Tanimoto structural similarity scoring is performed)

## Evaluation signals

- Recomputed per-assay F1 and sensitivity match the article's reported values for the FELLA + raw score configuration (e.g. Assay 6 F1 = 0.49, sensitivity = 0.72).
- Recomputed Tanimoto ratios match reported values (Assay 1: 0.69; Assay 3: 0.61) under the FELLA + raw score configuration.
- Per-assay TP/FP/FN counts reproduce the article's table values (e.g. Assay 6: 53 TP, 90 FP, 7852 TN, 21 FN).
- High false-positive counts are accompanied by correspondingly high Tanimoto ratios, confirming that many false positives are structurally close to the true compounds.
- Every candidate row in the aggregated entity table carries a Tanimoto ratio column consistent with the article's results table schema.

## Limitations

- The Tanimoto ratio is only meaningful when chemical structure data exist for both proposed and reference compounds; entities without structural annotations cannot be scored this way.
- mWISE only offers data to annotate in the KEGG database, so structural similarity comparisons are bounded by KEGG-compound structures and cannot be extended to HMDB, Lipid Maps, or T3DB entries as xMSannotator can.
- A high Tanimoto ratio does not recover a hit — near-miss false positives still count against precision and F1 in the entity-level table.

## Evidence

- [results] Tanimoto ratios confirm near-miss FPs: [evidence span withheld — anchor status: not_found]
