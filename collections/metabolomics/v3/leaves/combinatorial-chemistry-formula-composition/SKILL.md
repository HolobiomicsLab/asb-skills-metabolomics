---
name: combinatorial-chemistry-formula-composition
description: Use when annotating MALDI-imaging data where matrix clusters (e.g., 2,5-dihydroxybenzoic
  acid/DHB) pollute annotations and a candidate list of matrix cluster ion formulas
  must be generated for a custom annotation database.
when_to_use_negative:
- The matrix of interest is not DHB (C7H6O4) — the Keller and Li (2000) combinatorial
  model and the 353-formula set are specific to 2,5-dihydroxybenzoic acid clusters
- The goal is to segment off-sample pixels within an image rather than generate candidate
  matrix-cluster formulas for annotation filtering
- Biologically relevant molecular annotations (not matrix artifacts) are the target
  of enumeration
edam_topics:
- http://edamontology.org/topic_0035
tools:
- name: METASPACE
  role: Knowledge base used to annotate MALDI-imaging datasets with the custom DHB
    matrix-cluster formula database; matrix clusters surfaced as off-sample annotations
    there motivated the formula enumeration
  repo: https://github.com/metaspace2020/metaspace/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:ca8359499e980ee839482329f6c85849a7d54dfff1c5b127e8fe8834ccf2cd44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0035
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# combinatorial-chemistry-formula-composition

## Summary

Enumerates a combinatorial space of MALDI matrix cluster ion formulas — here DHB clusters built as n*M + p*(M-H2O) - x*H + y*K + z*Na with M = C7H6O4 — to generate candidate formulas for a custom METASPACE molecular database used to annotate MALDI-imaging datasets.

## When to use

Use when annotating MALDI-imaging data where matrix clusters (e.g., 2,5-dihydroxybenzoic acid/DHB) pollute annotations and a candidate list of matrix cluster ion formulas must be generated for a custom annotation database.

## When NOT to use

- The matrix of interest is not DHB (C7H6O4) — the Keller and Li (2000) combinatorial model and the 353-formula set are specific to 2,5-dihydroxybenzoic acid clusters
- The goal is to segment off-sample pixels within an image rather than generate candidate matrix-cluster formulas for annotation filtering
- Biologically relevant molecular annotations (not matrix artifacts) are the target of enumeration

## Inputs

- Cluster-generating model specification from Keller and Li (2000) reference PDF
- Base matrix molecular formula M = C7H6O4 (2,5-dihydroxybenzoic acid, DHB)
- Integer ranges/bounds for the combinatorial parameters n (intact M), p (M-H2O), x (proton loss), y (K addition), z (Na addition)

## Outputs

- CSV file 'DHB mc mol formulas, predicted.csv' containing 353 predicted matrix cluster molecular formulas
- Elemental formula strings in normalized chemical notation for each valid (n, p, x, y, z) combination

## How to apply

Load the cluster-generating model specification from Keller and Li (2000) and set the base molecule M = C7H6O4 (2,5-dihydroxybenzoic acid, DHB). Enumerate all valid (n, p, x, y, z) combinations of n intact DHB molecules and p dehydrated DHB units (M-H2O) modified by loss of x protons and addition of y potassium and z sodium. For each combination, sum element counts from the components and normalize the resulting chemical formula notation into a standard elemental formula string. Export the complete predicted formula set (353 formulas in the reference implementation) to a CSV file ('DHB mc mol formulas, predicted.csv') matching the structure deposited in the DHB matrix clusters directory of the metaspace2020/offsample repository. The resulting formula database is then used with METASPACE annotation of MALDI-imaging datasets so matrix-cluster signals can be recognized as off-sample.

## Related tools

- **METASPACE** (Knowledge base used to annotate MALDI-imaging datasets with the custom DHB matrix-cluster formula database; matrix clusters surfaced as off-sample annotations there motivated the formula enumeration) — https://github.com/metaspace2020/metaspace/

## Examples

```
python -c "import pandas as pd; pd.DataFrame(formulas, columns=['formula']).to_csv('DHB mc mol formulas, predicted.csv', index=False)"
```

## Evaluation signals

- Output CSV exists, parses correctly, and contains exactly 353 molecular formula rows consistent with the reported count of 353 formulas
- Every emitted formula decomposes exactly as n*M + p*(M-H2O) - x*H + y*K + z*Na with M = C7H6O4 under integer (n, p, x, y, z)
- Element counts in each formula equal the summed counts of its intact DHB and dehydrated DHB components plus cation modifications
- CSV matches the structure of 'DHB mc mol formulas, predicted.csv' deposited in the DHB matrix clusters directory of github.com/metaspace2020/offsample
- Formula strings are normalized to consistent chemical formula notation

## Limitations

- The combinatorial model is restricted to DHB matrix clusters per Keller and Li (2000); formation of clusters for other MALDI matrices is not covered
- Generating candidate formulas does not solve segmentation of off-sample regions — the authors note the approach 'does not solve the segmentation problem of finding pixels which are in the off-sample region'
- The 353-formula set is a prediction of possible cluster species, not a validated list of observed clusters; validation in the paper relied on expert review of METASPACE ion images

## Evidence

- [readme] This resulted in 353 molecular formulas available [here](./DHB%20matrix%20clusters/DHB%20mc%20mol%20formulas%2C%20predicted.csv).: "This resulted in 353 molecular formulas available [here](./DHB%20matrix%20clusters/DHB%20mc%20mol%20formulas%2C%20predicted.csv)."
- [readme] We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf).: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf)."
- [intro] we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating"
- [discussion] our approach does not solve the segmentation problem of finding pixels which are in the off-sample region.: "our approach does not solve the segmentation problem of finding pixels which are in the off-sample region."
