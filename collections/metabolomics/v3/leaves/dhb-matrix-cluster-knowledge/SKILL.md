---
name: dhb-matrix-cluster-knowledge
description: Use when building a custom METASPACE molecular database for MALDI-imaging
  datasets acquired with the DHB matrix, or when matrix-cluster ion signals (e.g.
  DHB adducts and dehydrates) must be predicted and filtered from annotations.
when_to_use_negative:
- The MALDI matrix is not DHB (2,5-dihydroxybenzoic acid) — the Keller and Li (2000)
  combinatorial model and the C7H6O4 base molecule are DHB-specific
- The goal is annotating genuine biological/metabolite ions rather than recognizing
  or filtering off-sample matrix-cluster signals
- The task requires pixel-level segmentation of off-sample regions, which the article
  explicitly states this approach does not solve
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE
  role: Cloud platform into which the predicted DHB cluster formulas feed a custom
    molecular database for annotating MALDI-imaging datasets, and where the off-sample
    filter hides matrix-cluster annotations
  repo: https://github.com/metaspace2020/metaspace/
- name: manual expert review
  role: Expert inspection of DHB off-sample ion images used to validate the characterized
    matrix-cluster properties
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:f5479170af17179daab527076840857c9652e9d95b39c1ee3d516c3eba559526
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - manual expert review
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# dhb-matrix-cluster-knowledge

## Summary

Enumerates candidate molecular formulas for DHB (2,5-dihydroxybenzoic acid) matrix clusters using the Keller and Li (2000) combinatorial cluster-generating model, producing a custom reference list for annotating MALDI-imaging data in METASPACE.

## When to use

Use when building a custom METASPACE molecular database for MALDI-imaging datasets acquired with the DHB matrix, or when matrix-cluster ion signals (e.g. DHB adducts and dehydrates) must be predicted and filtered from annotations.

## When NOT to use

- The MALDI matrix is not DHB (2,5-dihydroxybenzoic acid) — the Keller and Li (2000) combinatorial model and the C7H6O4 base molecule are DHB-specific
- The goal is annotating genuine biological/metabolite ions rather than recognizing or filtering off-sample matrix-cluster signals
- The task requires pixel-level segmentation of off-sample regions, which the article explicitly states this approach does not solve

## Inputs

- Cluster-generating model specification from Keller and Li, 2000 (./DHB matrix clusters/Keller and Li, 2000.pdf)
- Base matrix molecular formula M = C7H6O4 (2,5-dihydroxybenzoic acid)
- Target MALDI-imaging datasets acquired with the DHB matrix

## Outputs

- 'DHB mc mol formulas, predicted.csv' with exactly 353 predicted DHB matrix-cluster molecular formulas
- Custom METASPACE molecular database for annotating DHB MALDI-imaging datasets

## How to apply

Set the base molecule M = C7H6O4 (2,5-dihydroxybenzoic acid) from the Keller and Li (2000) cluster-generating model. Enumerate the combinatorial space of candidate cluster ion formulas as n*M + p*(M-H2O) - x*H + y*K + z*Na, i.e. combinations of intact DHB molecules and dehydrated DHB units with variable proton losses and potassium/sodium additions. For each valid (n, p, x, y, z) combination, sum element counts and normalize the chemical formula notation. Export the full set to 'DHB mc mol formulas, predicted.csv', which in the offsample repository contains exactly 353 predicted molecular formulas. The list is used as a custom database so DHB matrix-cluster off-sample signals can be recognized (e.g. by the off-sample filter in METASPACE) and excluded from biological interpretation.

## Related tools

- **METASPACE** (Cloud platform into which the predicted DHB cluster formulas feed a custom molecular database for annotating MALDI-imaging datasets, and where the off-sample filter hides matrix-cluster annotations) — https://github.com/metaspace2020/metaspace/
- **manual expert review** (Expert inspection of DHB off-sample ion images used to validate the characterized matrix-cluster properties)

## Evaluation signals

- The output CSV 'DHB mc mol formulas, predicted.csv' exists and parses correctly
- The CSV contains exactly 353 molecular formula rows, matching the reported count of 353 predicted formulas
- Each formula is expressible as n*M + p*(M-H2O) - x*H + y*K + z*Na with M = C7H6O4 (element counts consistent with integer n, p, x, y, z)
- Formulas are in normalized chemical notation with summed element counts
- Predicted clusters correspond to the off-sample DHB ion images identified in METASPACE public datasets

## Limitations

- The model is specific to the DHB matrix; matrix-cluster formation for other MALDI matrices is not covered
- The approach does not solve the segmentation problem of finding which pixels are off-sample
- Ground-truth information about the molecular content of analyzed samples is lacking, so validation relies on expert tagging of off-sample images

## Evidence

- [other] candidate DHB matrix cluster ion formulas are enumerated as n*M + p*(M-H2O) - x*H + y*K + z*Na, where M is the DHB matrix molecular formula C7H6O4: [evidence span withheld — anchor status: not_found]
- [readme] We have generated DHB matrix clusters according to (Keller and Li, 2000). This resulted in 353 molecular formulas: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas available"
- [intro] we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and derived properties of matrix clusters using a cluster-generating: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating"
- [abstract] we investigated off-sample images corresponding to the most common MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and characterized properties of matrix clusters: "we investigated off-sample images corresponding to the most common MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and characterized properties of matrix clusters"
