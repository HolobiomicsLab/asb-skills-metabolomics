---
name: spirotetronate-macrolide-subset-selection
description: Use when an NPClassifier-labeled NP Atlas table must be narrowed to spirotetronate
  macrolide entries and split by decalin-moiety presence for subgroup-level comparison
  of MAIP-predicted antimalarial scores.
when_to_use_negative:
- The compound set has not been structurally classified yet — run NPClassifier first
  (/classify?smiles=<>); this skill assumes upstream pathway/class labels already
  exist.
- The analysis targets scaffolds outside the spirotetronate macrolide chemical space
  in the NP Atlas — the filter and decalin-annotation criteria are motif-specific.
- Measured antimalarial activity is required rather than MAIP-predicted biosynthetic-likelihood
  scores.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3316
tools:
- name: NPClassifier
  role: Provides the pathway/class labels used to filter NP Atlas entries to spirotetronate
    macrolides and to re-confirm MAIP-predicted actives via /classify?smiles=<>, optionally
    with the cached flag for faster repeat queries
  repo: https://github.com/mwang87/NP-Classifier
- name: MAIP
  role: EMBL-EBI web service supplying batch-predicted antimalarial biosynthetic-likelihood
    scores that are aggregated per decalin subgroup
- name: Python
  role: Scripting the table filtering, decalin annotation, subgroup split, summary
    statistics, and CSV export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:38a511ad8c3dd1bf4c2c9ef86399a42076991493255d175739361d72a4335e91
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# spirotetronate-macrolide-subset-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects spirotetronate macrolide entries from an NPClassifier-annotated NP Atlas table, scores their SMILES via the EMBL-EBI MAIP web service, and splits the set by decalin-moiety presence to compare predicted antimalarial potency (mean MAIP 97.2 vs 25.1).

## When to use

Use when an NPClassifier-labeled NP Atlas table must be narrowed to spirotetronate macrolide entries and split by decalin-moiety presence for subgroup-level comparison of MAIP-predicted antimalarial scores.

## When NOT to use

- The compound set has not been structurally classified yet — run NPClassifier first (/classify?smiles=<>); this skill assumes upstream pathway/class labels already exist.
- The analysis targets scaffolds outside the spirotetronate macrolide chemical space in the NP Atlas — the filter and decalin-annotation criteria are motif-specific.
- Measured antimalarial activity is required rather than MAIP-predicted biosynthetic-likelihood scores.

## Inputs

- NPClassifier-labeled NP Atlas table (per-compound SMILES plus Pathway/Class assignments from the upstream classification task)
- SMILES strings of candidate spirotetronate macrolide structures

## Outputs

- maip_spirotetronate_analysis.csv containing per-compound MAIP scores with decalin subgroup assignments
- Subgroup summary statistics: group sizes (decalin-containing n=55), mean MAIP scores (97.2 vs 25.1), observed score range (14.2-138.8)
- Confirmed NPClassifier pathway/class labels for MAIP-predicted actives

## How to apply

Load the classified NP Atlas table produced by the upstream NPClassifier task (structures classified from counted Morgan fingerprints) and filter it to spirotetronate macrolide entries. Submit the selected SMILES strings in batch to the public EMBL-EBI MAIP web service and collect the returned MAIP biosynthetic-likelihood scores; re-confirm pathway/class assignments of MAIP-predicted actives by calling NPClassifier's /classify?smiles=<> endpoint. Annotate each macrolide binary for the decalin moiety and split the set into decalin-containing versus non-decalin subgroups. Compute the mean MAIP score per subgroup together with group sizes and the observed score range, and write per-compound scores plus subgroup summary statistics to maip_spirotetronate_analysis.csv. The reference outcome is a decalin-containing subgroup of n=55 with mean MAIP 97.2 versus 25.1 without decalin, with individual scores spanning 14.2-138.8.

## Related tools

- **NPClassifier** (Provides the pathway/class labels used to filter NP Atlas entries to spirotetronate macrolides and to re-confirm MAIP-predicted actives via /classify?smiles=<>, optionally with the cached flag for faster repeat queries) — https://github.com/mwang87/NP-Classifier
- **MAIP** (EMBL-EBI web service supplying batch-predicted antimalarial biosynthetic-likelihood scores that are aggregated per decalin subgroup)
- **Python** (Scripting the table filtering, decalin annotation, subgroup split, summary statistics, and CSV export)

## Examples

```
requests.get(f"{NPCLASSIFIER_URL}/classify", params={"smiles": smi, "cached": "true"}).json()
```

## Evaluation signals

- Decalin-containing subgroup contains n=55 compounds with mean MAIP score 97.2, versus 25.1 for macrolides lacking the decalin motif.
- All individual MAIP scores fall within the reported 14.2-138.8 range.
- Every retained row carries spirotetronate macrolide NPClassifier pathway/class labels, re-confirmed for actives via /classify?smiles=<>, and local NPClassifier serving exposes input layers 'input_2048'/'input_4096' and output layer 'output'.
- maip_spirotetronate_analysis.csv contains both per-compound scores and subgroup summary statistics (group sizes, means, score range).

## Limitations

- MAIP outputs are predicted biosynthetic-likelihood/antimalarial scores, so the 97.2-vs-25.1 decalin contrast reflects predicted potency, not measured activity.
- Subset fidelity depends on upstream NPClassifier labels — misassigned Pathway/Class entries are wrongly included or excluded from the macrolide subset.
- Local NPClassifier deployment requires TensorFlow 2.3.0 to convert the Keras models into HDF5 TF2 models plus docker and docker-compose; repeated queries are slower without the cached parameter.

## Evidence

- [other] Load the classified NP Atlas table produced by the upstream NPClassifier task and filter it to spirotetronate macrolide entries.: [evidence span withheld — anchor status: not_found]
- [other] the decalin-containing macrolide subgroup (n=55) yields a mean MAIP score of 97.2 versus 25.1 for macrolides without decalin, with individual scores falling in the reported 14.2-138.8 range.: [evidence span withheld — anchor status: not_found]
