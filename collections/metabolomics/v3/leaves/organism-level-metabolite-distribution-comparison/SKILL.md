---
name: organism-level-metabolite-distribution-comparison
description: Use when you have SMILES-tagged natural-product structures grouped by
  organism origin and want to identify which chemical pathway or superclass distributions
  differ between those groups.
when_to_use_negative:
- When the input structures are not natural products or lack organism-origin labels
  — classification alone will not give a group comparison.
- When you need fine-grained Class-level rather than Pathway/Superclass-level comparison
  without prior aggregation, since the article's comparison was done at pathway/superclass
  level.
- When the server is not running or models are not downloaded (HDF5 TF2 models require
  tensorflow 2.3.0) — the /classify endpoint will not respond.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: NPClassifier
  role: Predicts Pathway, Superclass, and Class for each SMILES via /classify?smiles=
    endpoint, feeding the per-organism distribution comparison
  repo: github.com/mwang87/NP-Classifier
- name: TensorFlow
  role: Backs the three feed-forward DNN networks that generate the classification
    labels compared between organism groups
  repo: github.com/mwang87/NP-Classifier
- name: Python
  role: Loads the organism-tagged structure list, queries the classification API,
    and aggregates per-pathway/per-superclass counts
- name: docker / docker-compose
  role: Deploys the local classification server whose /classify endpoint the comparison
    pipeline queries
  repo: github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:012d876b3be12210c534b030755b9965ef598463c5ada531108907f0c3d1f4de
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  tools:
  - NPClassifier
  - Python
  - TensorFlow
  - docker
  - docker / docker-compose
  - docker-compose
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# organism-level-metabolite-distribution-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares the structural-classification distribution of natural products across organism groups (e.g. bacterial vs fungal metabolites) by classifying each structure with NPClassifier and aggregating per-pathway/superclass counts per organism. Reveals biosynthetic-level patterns such as the terpenoid dominance in fungi.

## When to use

Use when you have SMILES-tagged natural-product structures grouped by organism origin and want to identify which chemical pathway or superclass distributions differ between those groups.

## When NOT to use

- When the input structures are not natural products or lack organism-origin labels — classification alone will not give a group comparison.
- When you need fine-grained Class-level rather than Pathway/Superclass-level comparison without prior aggregation, since the article's comparison was done at pathway/superclass level.
- When the server is not running or models are not downloaded (HDF5 TF2 models require tensorflow 2.3.0) — the /classify endpoint will not respond.

## Inputs

- List of natural-product SMILES strings tagged with organism origin (e.g. bacterial vs fungal) from the Natural Products Atlas
- Running NPClassifier dockerized classification server (docker + docker-compose, models downloaded via get_models.sh)

## Outputs

- Per-organism aggregate count table of NPClassifier Pathway and Superclass labels (e.g. npatlas_pathway_class_counts.csv)
- Cross-organism comparison of pathway-level distributions highlighting differentially abundant superclasses (e.g. PKS subtypes: macrolides/polycyclic aromatic polyketides vs cyclic polyketides/chromones/naphthalene derivatives)

## How to apply

Load a natural-product structure list (SMILES strings tagged with organism origin, e.g. from the Natural Products Atlas) with Python and query the dockerized NPClassifier server at /classify?smiles=<smiles>; the server converts each SMILES to a counted Morgan fingerprint and runs three feed-forward DNN networks (TensorFlow 2.3.0) that predict Pathway, Superclass, and Class simultaneously. Split the returned classifications by organism group, then aggregate counts per pathway and per superclass for each organism into a single table (e.g. npatlas_pathway_class_counts.csv). Decide group membership from the organism-origin tag, not from structure alone, and interpret differences at the biosynthetic level — e.g. fungal metabolites yielded over 3000 terpenoids versus around 300 from bacteria, mirroring BGC patterns in MIBiG.

## Related tools

- **NPClassifier** (Predicts Pathway, Superclass, and Class for each SMILES via /classify?smiles= endpoint, feeding the per-organism distribution comparison) — github.com/mwang87/NP-Classifier
- **TensorFlow** (Backs the three feed-forward DNN networks that generate the classification labels compared between organism groups) — github.com/mwang87/NP-Classifier
- **Python** (Loads the organism-tagged structure list, queries the classification API, and aggregates per-pathway/per-superclass counts)
- **docker / docker-compose** (Deploys the local classification server whose /classify endpoint the comparison pipeline queries) — github.com/mwang87/NP-Classifier

## Examples

```
curl 'http://localhost:8501/classify?smiles=CC(=O)OC1=CC=CC=C1C(=O)OH'
```

## Evaluation signals

- Verify the aggregate table reproduces the known terpenoid split: over 3000 fungal terpenoids versus around 300 bacterial terpenoids.
- Check that bacterial PKS superclass distribution is dominated by macrolides and polycyclic aromatic polyketides while fungal PKS classes are cyclic polyketides, chromones, aromatic polyketides, and naphthalene derivatives.
- Confirm every input SMILES received a three-level classification (Pathway, Superclass, Class) before aggregation; missing labels indicate a failed /classify call.
- Cross-check that row counts of the aggregated table sum to the number of unique organism-tagged structures queried.

## Limitations

- Comparison quality depends entirely on the completeness and curation of the organism-origin tags in the source structure list (e.g. NP Atlas entries).
- Requires the pretrained NPClassifier models to be downloaded and converted with tensorflow 2.3.0 before the dockerized server can serve /classify; otherwise the pipeline stalls at startup.
- Differences reflect structural-classification distribution, not direct BGC annotation — interpretation as biosynthetic differences is inferred (e.g. the terpenoid ratio 'similar to BGCs in MIBiG').

## Evidence

- [discussion] over 3000 terpenoids were reported from fungi versus only around 300 from bacteria: [evidence span withheld — anchor status: not_found]
