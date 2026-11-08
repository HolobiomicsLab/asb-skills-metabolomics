---
name: decalin-motif-group-comparison
description: Use when a set of classified natural-product SMILES (e.g. spirotetronate
  macrolides from NP Atlas) must be split by presence/absence of a structural motif
  such as decalin and their predicted-potency scores compared between subgroups.
when_to_use_negative:
- Input compounds are not a defined NP-structure family (e.g. unclassified metabolomics
  features with no SMILES) — MAIP requires structures and NPClassifier annotation
  is a prerequisite.
- The goal is only pathway/class assignment rather than potency-score comparison between
  structural subgroups — NPClassifier classification alone suffices.
- The subgroup of interest is not representable as a structural annotation of the
  SMILES (e.g. an experimental condition rather than a moiety).
edam_operation: http://edamontology.org/operation_2492
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: NPClassifier
  role: Confirms pathway/class assignments for the MAIP-predicted actives via its
    /classify?smiles=<> endpoint before subgroup comparison.
  repo: https://github.com/mwang87/NP-Classifier
- name: MAIP
  role: Public EMBL-EBI web service that assigns biosynthetic-likelihood (antimalarial-potency)
    scores to each macrolide SMILES in batch.
- name: Python
  role: Used with the classified NP Atlas table for filtering, moiety annotation,
    subgroup splitting, and CSV output.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:273d4074a6bf898ffb17a5d337f34332240b0c0724827c92eac417f901652ce7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - MAIP
  - NPClassifier
  - Python
  edam_operation: http://edamontology.org/operation_2492
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# decalin-motif-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Group-comparison analysis of predicted antimalarial MAIP scores for spirotetronate macrolides from the NP Atlas, stratifying compounds by the presence of a decalin moiety and reporting subgroup means and size/range statistics to a CSV.

## When to use

Use when a set of classified natural-product SMILES (e.g. spirotetronate macrolides from NP Atlas) must be split by presence/absence of a structural motif such as decalin and their predicted-potency scores compared between subgroups.

## When NOT to use

- Input compounds are not a defined NP-structure family (e.g. unclassified metabolomics features with no SMILES) — MAIP requires structures and NPClassifier annotation is a prerequisite.
- The goal is only pathway/class assignment rather than potency-score comparison between structural subgroups — NPClassifier classification alone suffices.
- The subgroup of interest is not representable as a structural annotation of the SMILES (e.g. an experimental condition rather than a moiety).

## Inputs

- NPClassifier-classified table of NP Atlas natural products (with SMILES per compound)
- Batch SMILES strings of spirotetronate macrolide entries
- MAIP biosynthetic-likelihood scores returned by the EMBL-EBI MAIP web service

## Outputs

- maip_spirotetronate_analysis.csv with per-compound MAIP scores and subgroup summary statistics (group sizes, mean MAIP per subgroup, observed score range)

## How to apply

Filter the upstream NPClassifier-classified NP Atlas table down to spirotetronate macrolide entries, then submit their SMILES in batch to the public EMBL-EBI MAIP web service and collect the returned MAIP biosynthetic-likelihood scores. Re-confirm pathway/class assignments for the predicted actives by calling NPClassifier's /classify?smiles=<> API endpoint. Annotate each macrolide for decalin presence and split the set into decalin-containing versus non-decalin subgroups. Compute the mean MAIP score per subgroup, record group sizes and the observed score range (reported span 14.2–138.8), and write both per-compound scores and subgroup summary statistics to maip_spirotetronate_analysis.csv. The comparison is the rationale: if a structural motif (decalin) is hypothesized to drive predicted potency, subgroup means quantify whether the split is meaningful rather than a single pooled statistic.

## Related tools

- **NPClassifier** (Confirms pathway/class assignments for the MAIP-predicted actives via its /classify?smiles=<> endpoint before subgroup comparison.) — https://github.com/mwang87/NP-Classifier
- **MAIP** (Public EMBL-EBI web service that assigns biosynthetic-likelihood (antimalarial-potency) scores to each macrolide SMILES in batch.)
- **Python** (Used with the classified NP Atlas table for filtering, moiety annotation, subgroup splitting, and CSV output.)

## Examples

```
GET /classify?smiles=<macrolide_smiles>  # then, in Python: df = df.groupby('has_decalin')['maip_score'].agg(['mean','count']); df.to_csv('maip_spirotetronate_analysis.csv')
```

## Evaluation signals

- Decalin-containing subgroup size equals 55 and non-decalin subgroup is the remaining macrolides.
- Mean MAIP score of decalin-containing macrolides equals 97.2, versus 25.1 for macrolides lacking decalin.
- All individual MAIP scores fall within the reported 14.2–138.8 range.
- maip_spirotetronate_analysis.csv exists and contains both per-compound scores and subgroup summary statistics.
- Every analyzed macrolide has a confirmed NPClassifier pathway/class assignment before subgroup splitting.

## Limitations

- MAIP scores are predicted (biosynthetic-likelihood) values, not measured antimalarial activities; the subgroup comparison reflects prediction only.
- The reported statistics are specific to the NP Atlas spirotetronate macrolide set; applying this skill to other compound families would need new reference means, sizes, and score ranges.
- NPClassifier server deployment depends on TensorFlow 2.3.0 model conversion and docker/docker-compose availability (see README).

## Evidence

- [methods] Batch MAIP scoring of macrolides: [evidence span withheld — anchor status: not_found]
- [methods] NPClassifier classification confirmation: [evidence span withheld — anchor status: not_found]
- [methods] Subgroup definition and comparison: [evidence span withheld — anchor status: not_found]
- [results] Subgroup means: [evidence span withheld — anchor status: not_found]
- [readme] NPClassifier classify endpoint: "Classify programmatically ```/classify?smiles=<>```"
