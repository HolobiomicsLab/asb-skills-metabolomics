---
name: dhb-adduct-interpretation
description: Use when a set of off-sample matrix cluster ion annotations (e.g., DHB
  clusters from METASPACE datasets) must be interpreted in terms of a combinatorial
  adduct/cluster model, or when reproducing histogram analyses of cluster parameters
  n, p, x, z like Fig. 3 of Ovchinnikova et al. (2020).
when_to_use_negative:
- The ion images are on-sample tissue annotations rather than off-sample matrix clusters
  — this skill only characterizes off-sample DHB matrix clusters.
- The matrix is not DHB (2,5-dihydroxybenzoic acid) — the combinatorial model and
  the 353 predicted formulas are specific to DHB matrix clusters.
- The goal is to recognize/filter off-sample images rather than interpret the chemistry
  of already-recognized off-sample cluster annotations.
tools:
- name: METASPACE
  role: Source of public datasets and the pixel-annotation export from which recognized
    off-sample DHB cluster ion annotations are retrieved
  repo: http://metaspace2020.eu
- name: matplotlib
  role: Plotting the n, p, x, z parameter histograms in the Fig. 3 layout with labeled
    axes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:8562008816a9407fc2b102ac44e106e6cf3c6a6e4b2fc65db6c3cab4f024d163
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - METASPACE
  - matplotlib
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# dhb-adduct-interpretation

## Summary

Interprets recognized off-sample MALDI matrix cluster ions of 2,5-dihydroxybenzoic acid (DHB) by mapping their molecular formulas onto a combinatorial cluster-generating model (Keller and Li, 2000) with parameters n (DHB molecules), p, x (potassium), and z (sodium), then profiling the parameter distributions to characterize cluster chemistry.

## When to use

Use when a set of off-sample matrix cluster ion annotations (e.g., DHB clusters from METASPACE datasets) must be interpreted in terms of a combinatorial adduct/cluster model, or when reproducing histogram analyses of cluster parameters n, p, x, z like Fig. 3 of Ovchinnikova et al. (2020).

## When NOT to use

- The ion images are on-sample tissue annotations rather than off-sample matrix clusters — this skill only characterizes off-sample DHB matrix clusters.
- The matrix is not DHB (2,5-dihydroxybenzoic acid) — the combinatorial model and the 353 predicted formulas are specific to DHB matrix clusters.
- The goal is to recognize/filter off-sample images rather than interpret the chemistry of already-recognized off-sample cluster annotations.

## Inputs

- Recognized off-sample DHB cluster ion annotations from 31 METASPACE datasets (METASPACE pixel-annotation export, pixel-annot-export-v0.10.tar.gz)
- 353 predicted DHB matrix cluster molecular formulas from 'DHB mc mol formulas, predicted.csv' (Keller and Li, 2000 combinatorial model)

## Outputs

- Parameter value tallies (counts of n, p, x, z) across all recognized off-sample DHB cluster ions
- Histogram plots of parameters n, p, x, and z in the layout of Fig. 3

## How to apply

Load the recognized off-sample DHB cluster ion annotations exported from the 31 METASPACE datasets (pixel-annot-export-v0.10.tar.gz) and the 353 predicted DHB matrix cluster molecular formulas generated per Keller and Li (2000) from the CSV 'DHB mc mol formulas, predicted.csv' in the metaspace2020/offsample repository. Match each recognized cluster ion formula to its combinatorial model parameters n (number of DHB molecules), p, x (potassium count), and z (sodium count), then tally occurrence counts of each parameter value across all recognized off-sample DHB cluster ions. Plot histograms of n, p, x, and z (e.g., with matplotlib) in the layout of Fig. 3 with clearly labeled axes per parameter. Validate by checking that clusters with two DHB molecules (n=2) are the most prevalent class and that potassium-containing clusters (x>0) are nearly absent, i.e., x=0 for a prevalent majority of recognized cluster ions.

## Related tools

- **METASPACE** (Source of public datasets and the pixel-annotation export from which recognized off-sample DHB cluster ion annotations are retrieved) — http://metaspace2020.eu
- **matplotlib** (Plotting the n, p, x, z parameter histograms in the Fig. 3 layout with labeled axes)

## Examples

```
import pandas as pd; df = pd.read_csv('DHB matrix clusters/DHB mc mol formulas, predicted.csv'); df['n'].value_counts().sort_index().plot(kind='bar', xlabel='n (number of DHB molecules)')
```

## Evaluation signals

- Every recognized off-sample DHB cluster ion formula is matched to a row in the 353 predicted DHB mc mol formulas CSV (no unmatched formulas for recognized clusters).
- The n histogram shows n=2 (two DHB molecules) as the most prevalent class, matching the paper's reported qualitative pattern.
- The x histogram shows x=0 for a prevalent majority of recognized cluster ions (potassium-containing clusters, x>0, nearly absent).
- Histograms cover all four parameters n, p, x, z with clearly labeled axes in the Fig. 3 layout.
- Tallies are computed across the full set of recognized off-sample DHB cluster ions from the 31 METASPACE datasets.

## Limitations

- The off-sample recognition approach does not solve the segmentation problem of finding pixels which are in the off-sample region, so cluster interpretation is at the annotation (ion image) level, not the pixel level.
- Conclusions rest on the qualitative pattern reported for DHB and the Keller and Li (2000) cluster-generating model; other matrices or cluster models are out of scope.

## Evidence

- [results] Recognized DHB clusters show a slight prevalence of clusters including two DHB molecules, and for a prevalent majority of recognized cluster ions the potassium parameter y is equal to 0.: [evidence span withheld — anchor status: not_found]
- [results] Load the 353 predicted DHB matrix cluster molecular formulas generated per Keller and Li (2000) from the DHB mc mol formulas CSV.: [evidence span withheld — anchor status: not_found]
- [readme] We have generated DHB matrix clusters according to (Keller and Li, 2000). This resulted in 353 molecular formulas available [here](./DHB%20matrix%20clusters/DHB%20mc%20mol%20formulas%2C%20predicted.csv).: [evidence span withheld — anchor status: not_found]
- [intro] we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating"
