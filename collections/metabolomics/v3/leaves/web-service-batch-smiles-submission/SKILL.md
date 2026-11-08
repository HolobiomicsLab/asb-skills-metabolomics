---
name: web-service-batch-smiles-submission
description: Use when a filtered set of natural-product SMILES strings (e.g., NP Atlas
  macrolide entries) must be scored or classified by batch submission to web services
  such as EMBL-EBI MAIP or the NPClassifier /classify?smiles=<> endpoint.
when_to_use_negative:
- Input is raw LC-MS/MS spectra or a feature table rather than SMILES structures —
  MAIP and NPClassifier both take SMILES as query input.
- Compounds already carry pathway/class labels from a prior NPClassifier run and no
  new scoring is needed — reuse cached results instead of resubmitting every SMILES.
- The analysis requires measured antimalarial activity — MAIP returns predicted biosynthetic-likelihood
  scores, not an assay readout.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NPClassifier
  role: Structural classifier queried programmatically via /classify?smiles=<> to
    assign pathway/class to each submitted SMILES; deployable locally via docker/docker-compose
  repo: github.com/mwang87/NP-Classifier
- name: MAIP
  role: EMBL-EBI public web service that receives the batch SMILES of the selected
    macrolides and returns per-compound biosynthetic-likelihood scores
- name: docker-compose
  role: Brings up the local NPClassifier server (nginx-net network, make server-compose)
    that serves the /classify endpoint
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:8ffb03343cf9ed1fac80a04cc979a9daa8afe3e9bdf7d34903bb2f730d1a8870
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  tools:
  - MAIP
  - NPClassifier
  - Python
  - docker-compose
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# web-service-batch-smiles-submission

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch-submits natural-product SMILES strings (e.g., spirotetronate macrolides filtered from the NP Atlas) to web services — the EMBL-EBI MAIP service for per-compound biosynthetic-likelihood scores and NPClassifier's /classify?smiles=<> API for pathway/class structural classification — so subgroup statistics such as decalin-containing vs non-decalin mean MAIP scores can be computed from the returned values.

## When to use

Use when a filtered set of natural-product SMILES strings (e.g., NP Atlas macrolide entries) must be scored or classified by batch submission to web services such as EMBL-EBI MAIP or the NPClassifier /classify?smiles=<> endpoint.

## When NOT to use

- Input is raw LC-MS/MS spectra or a feature table rather than SMILES structures — MAIP and NPClassifier both take SMILES as query input.
- Compounds already carry pathway/class labels from a prior NPClassifier run and no new scoring is needed — reuse cached results instead of resubmitting every SMILES.
- The analysis requires measured antimalarial activity — MAIP returns predicted biosynthetic-likelihood scores, not an assay readout.

## Inputs

- SMILES strings of the target natural products (e.g., spirotetronate macrolide entries filtered from a classified NP Atlas table)
- Access to the EMBL-EBI MAIP web service for biosynthetic-likelihood scoring
- A locally deployed NPClassifier server (dockerized) or reachable /classify endpoint, with models fetched via get_models.sh

## Outputs

- Per-compound MAIP biosynthetic-likelihood scores (observed range 14.2-138.8) keyed to each submitted SMILES
- NPClassifier pathway/class assignments returned by /classify?smiles=<>
- Per-compound score table with subgroup summaries, e.g., maip_spirotetronate_analysis.csv (n=55 decalin-containing vs non-decalin mean comparison)

## How to apply

Extract the SMILES strings of the target compounds (e.g., spirotetronate macrolides filtered from the classified NP Atlas table) and submit them in batch: the EMBL-EBI MAIP web service returns a per-compound biosynthetic-likelihood score (scores in the reproduced analysis fell in the 14.2-138.8 range), while NPClassifier returns structural classification at its three hierarchy levels. For classification, deploy NPClassifier locally with docker and docker-compose — create the nginx-net network, fetch the models with get_models.sh (TensorFlow 2.3.0 is required to convert the Keras models into HDF5 TF2 models), then run make server-compose — or query /classify?smiles=<> directly. Add the cached flag parameter for repeat queries since it returns a cached version that is faster. Use MAIP when the endpoint of interest is the potency-relevant biosynthetic-likelihood score and NPClassifier when confirming pathway/class assignments of MAIP-predicted actives. Persist each returned per-compound score (e.g., to maip_spirotetronate_analysis.csv) so subgroup means (97.2 for the n=55 decalin-containing macrolides vs 25.1 without decalin) and score ranges can be recomputed from the raw outputs.

## Related tools

- **NPClassifier** (Structural classifier queried programmatically via /classify?smiles=<> to assign pathway/class to each submitted SMILES; deployable locally via docker/docker-compose) — github.com/mwang87/NP-Classifier
- **MAIP** (EMBL-EBI public web service that receives the batch SMILES of the selected macrolides and returns per-compound biosynthetic-likelihood scores)
- **docker-compose** (Brings up the local NPClassifier server (nginx-net network, make server-compose) that serves the /classify endpoint)

## Examples

```
curl "http://localhost/classify?smiles=<>&cached=true"
```

## Evaluation signals

- One returned score/classification per submitted SMILES; maip_spirotetronate_analysis.csv has a row for every filtered macrolide entry.
- MAIP scores for the spirotetronate macrolide set fall within the reported 14.2-138.8 range.
- Reproduced subgroup statistics match the validation target: decalin-containing macrolides (n=55) mean MAIP = 97.2 vs 25.1 for macrolides lacking the decalin motif.
- NPClassifier responses populate the three classification hierarchy levels consistently across repeat queries.
- A query submitted with the cached flag returns the same classification as the uncached call.

## Limitations

- Local NPClassifier deployment requires docker, docker-compose, and TensorFlow 2.3.0 (get_models.sh converts the Keras models into HDF5 TF2 models) before queries can be served.
- Model layer names are fixed (inputs 'input_2048' and 'input_4096', output 'output'); if model input names change, the code must be updated.
- The deployed service keeps logs of which structures were classified (though not which users queried them) — consider this when submitting confidential structures.
- MAIP outputs are predicted biosynthetic-likelihood scores, so the decalin-vs-non-decalin difference (97.2 vs 25.1) is a predictive correlation, not measured antimalarial potency.

## Evidence
