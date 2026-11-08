---
name: maldi-matrix-cluster-modeling
description: Use when annotating MALDI-imaging datasets where matrix cluster ions
  (e.g., from 2,5-dihydroxybenzoic acid) may pollute annotations, and a predicted
  list of matrix cluster formulas is needed to match recognized off-sample signals.
when_to_use_negative:
- The MALDI matrix is not DHB and no analogous cluster-generating model is available
  for it — the Keller and Li (2000) parameterization is specific to DHB-derived clusters
- The goal is detecting analyte metabolites rather than matrix/off-sample cluster
  ions — use the standard METASPACE annotation database instead
- Segmenting individual off-sample pixels is required — this skill models cluster
  formulas only and 'does not solve the segmentation problem of finding pixels which
  are in the off-sample region'
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE
  role: Source of public MALDI-imaging datasets and pixel-level annotations against
    which the predicted matrix cluster formulas are matched to identify recognized
    off-sample DHB clusters
  repo: https://github.com/metaspace2020/metaspace/
- name: TagOff
  role: Web app used by expert taggers to label ion images as off-sample/on-sample/unknown,
    providing the recognized off-sample DHB cluster annotations to compare with predicted
    formulas
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:3f422ac1b552943855d0c095f7426dc4a41d7475358a81fbbaa6ca2587f29a31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - TagOff
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# maldi-matrix-cluster-modeling

## Summary

Enumerate candidate MALDI matrix cluster ion molecular formulas using a combinatorial cluster-generating model (Keller and Li, 2000), here applied to the DHB matrix, to build a custom molecular database for annotating matrix/off-sample signals in METASPACE MALDI-imaging data.

## When to use

Use when annotating MALDI-imaging datasets where matrix cluster ions (e.g., from 2,5-dihydroxybenzoic acid) may pollute annotations, and a predicted list of matrix cluster formulas is needed to match recognized off-sample signals.

## When NOT to use

- The MALDI matrix is not DHB and no analogous cluster-generating model is available for it — the Keller and Li (2000) parameterization is specific to DHB-derived clusters
- The goal is detecting analyte metabolites rather than matrix/off-sample cluster ions — use the standard METASPACE annotation database instead
- Segmenting individual off-sample pixels is required — this skill models cluster formulas only and 'does not solve the segmentation problem of finding pixels which are in the off-sample region'

## Inputs

- Cluster-generating model specification from Keller and Li (2000) (./DHB matrix clusters/Keller and Li, 2000.pdf)
- Base matrix molecular formula (DHB: C7H6O4)
- METASPACE pixel-annotation export for matching recognized clusters (pixel-annot-export-v0.10.tar.gz, 31 datasets)

## Outputs

- CSV of predicted matrix cluster molecular formulas ('DHB mc mol formulas, predicted.csv'; 353 formulas for DHB)
- Parameter tallies/histograms (n, p, x, z) for recognized matrix cluster ions (cf. Fig. 3)

## How to apply

Set the base molecule M to the matrix formula, e.g., M = C7H6O4 for 2,5-dihydroxybenzoic acid (DHB). Enumerate all valid combinations of n intact matrix molecules and p water-loss units (M-H2O) modified by proton losses and alkali additions: n*M + p*(M-H2O) - x*H + y*K + z*Na, taking the parameter ranges from Keller and Li (2000). Compose each elemental formula string by summing element counts and normalizing chemical formula notation. Export the resulting set (353 predicted DHB matrix cluster formulas in this project) to CSV (e.g., 'DHB mc mol formulas, predicted.csv' in the offsample repository's DHB matrix clusters directory). These predicted formulas are then matched against METASPACE annotations recognized as off-sample to derive cluster properties such as the distributions of n, p, x (potassium), and z (sodium).

## Related tools

- **METASPACE** (Source of public MALDI-imaging datasets and pixel-level annotations against which the predicted matrix cluster formulas are matched to identify recognized off-sample DHB clusters) — https://github.com/metaspace2020/metaspace/
- **TagOff** (Web app used by expert taggers to label ion images as off-sample/on-sample/unknown, providing the recognized off-sample DHB cluster annotations to compare with predicted formulas) — https://github.com/metaspace2020/offsample

## Examples

```
pandas.read_csv('./DHB matrix clusters/DHB mc mol formulas, predicted.csv')  # verify 353 predicted DHB cluster formulas
```

## Evaluation signals

- Output CSV exists, parses correctly, and contains exactly 353 molecular formula rows, consistent with the reported count
- Each enumerated (n, p, x, y, z) combination maps to a normalized elemental formula string with correct summed element counts from n*C7H6O4 + p*(C7H6O4-H2O)
- Histograms of parameters across recognized off-sample DHB clusters reproduce the reported qualitative pattern: n=2 (two DHB molecules) is the most prevalent class and potassium-containing clusters (y>0) are nearly absent
- Matched formulas correspond to annotations from the 31 METASPACE datasets in the pixel-annotation export

## Limitations

- The model is parameterized for the DHB matrix (M = C7H6O4); it does not generalize to other MALDI matrices without a new cluster-generating model
- Predicted formulas are candidates only; validation relies on which formulas are actually recognized as off-sample in expert-tagged METASPACE data
- The off-sample recognition approach underlying the matching is image-level classification, not pixel-level segmentation

## Evidence

- [other] candidate DHB matrix cluster ion formulas are enumerated as n*M + p*(M-H2O) - x*H + y*K + z*Na, where M is the DHB matrix molecular formula C7H6O4: [evidence span withheld — anchor status: not_found]
- [readme] We have generated DHB matrix clusters according to (Keller and Li, 2000). This resulted in 353 molecular formulas: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas"
- [results] Recognized DHB clusters show a slight prevalence of clusters including two DHB molecules, and for a prevalent majority of recognized cluster ions the potassium parameter y is equal to 0.: [evidence span withheld — anchor status: not_found]
- [intro] we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and derived properties of matrix clusters using a cluster-generating: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating"
- [discussion] our approach does not solve the segmentation problem of finding pixels which are in the off-sample region: "our approach does not solve the segmentation problem of finding pixels which are in the off-sample region."
