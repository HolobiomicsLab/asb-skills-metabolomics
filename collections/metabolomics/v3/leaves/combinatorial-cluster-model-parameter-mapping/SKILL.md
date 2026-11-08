---
name: combinatorial-cluster-model-parameter-mapping
description: Use when you have off-sample matrix cluster ion annotations (e.g. from
  METASPACE pixel-annot-export) and a predicted list of matrix cluster molecular formulas
  with combinatorial model parameters, and need to characterize parameter distributions
  of recognized clusters.
when_to_use_negative:
- Input annotations are on-sample or unclassified ion images rather than recognized
  off-sample matrix cluster ions — the parameter mapping is only meaningful for recognized
  off-sample DHB clusters
- The matrix is not DHB (2,5-dihydroxybenzoic acid); the 353 predicted formulas and
  the n/p/x/z parameterization come from the Keller and Li (2000) DHB-specific cluster
  model
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: METASPACE
  role: Source of the open-access pixel-annotation export containing recognized off-sample
    DHB cluster ion annotations used to map against predicted cluster formulas
  repo: https://github.com/metaspace2020/metaspace/
- name: metaspace2020/offsample
  role: Hosts the predicted DHB matrix cluster molecular formulas CSV ('DHB mc mol
    formulas, predicted.csv') generated per Keller and Li (2000)
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:dbd36e1b8ba20fc641115721cc7b9357a65fe6677f6f31709e0121c29f9cc955
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - metaspace2020/offsample
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# combinatorial-cluster-model-parameter-mapping

## Summary

Maps recognized off-sample DHB matrix cluster ion formulas from METASPACE to the parameters (n, p, x, z) of the Keller and Li (2000) combinatorial cluster-generating model, then tallies and histograms each parameter to characterize DHB matrix cluster composition.

## When to use

Use when you have off-sample matrix cluster ion annotations (e.g. from METASPACE pixel-annot-export) and a predicted list of matrix cluster molecular formulas with combinatorial model parameters, and need to characterize parameter distributions of recognized clusters.

## When NOT to use

- Input annotations are on-sample or unclassified ion images rather than recognized off-sample matrix cluster ions — the parameter mapping is only meaningful for recognized off-sample DHB clusters
- The matrix is not DHB (2,5-dihydroxybenzoic acid); the 353 predicted formulas and the n/p/x/z parameterization come from the Keller and Li (2000) DHB-specific cluster model

## Inputs

- Recognized off-sample DHB cluster ion annotations from 31 METASPACE datasets (pixel-annot-export-v0.10.tar.gz export)
- 353 predicted DHB matrix cluster molecular formulas per Keller and Li (2000) ('DHB mc mol formulas, predicted.csv') with combinatorial parameters n, p, x (potassium count), z (sodium count)

## Outputs

- Occurrence count tallies for each combinatorial parameter value (n, p, x, z) across all recognized DHB cluster ions
- Histogram plots of n, p, x, and z parameter distributions reproducing the layout of Fig. 3

## How to apply

Load the recognized off-sample DHB cluster ion annotations from the 31 METASPACE datasets via the pixel-annot-export-v0.10.tar.gz archive, and load the 353 predicted DHB matrix cluster molecular formulas generated per Keller and Li (2000) from the 'DHB mc mol formulas, predicted.csv' file. For each recognized cluster ion formula, match it to the combinatorial model parameters: n (number of DHB molecules), p, x (potassium count), and z (sodium count). Tally occurrence counts of each parameter value across all recognized off-sample cluster ions and plot histograms of n, p, x, and z (e.g. matplotlib) with clearly labeled axes, reproducing the layout of Fig. 3. Rationale: the model parameters explain which DHB cluster compositions the mass spectrometer actually detected; the expected qualitative pattern is that n=2 (two-DHB-molecule) clusters are most prevalent and potassium-containing clusters (x>0) are nearly absent.

## Related tools

- **METASPACE** (Source of the open-access pixel-annotation export containing recognized off-sample DHB cluster ion annotations used to map against predicted cluster formulas) — https://github.com/metaspace2020/metaspace/
- **metaspace2020/offsample** (Hosts the predicted DHB matrix cluster molecular formulas CSV ('DHB mc mol formulas, predicted.csv') generated per Keller and Li (2000)) — https://github.com/metaspace2020/offsample

## Examples

```
import pandas as pd; df = pd.read_csv('DHB mc mol formulas, predicted.csv'); print(df.groupby(['n','p','x','z']).size())
```

## Evaluation signals

- Every recognized off-sample DHB cluster ion formula resolves to a unique row in the 353-formula predicted list with defined n, p, x, z values (no unmatched formulas)
- Histogram of n shows n=2 (two-DHB-molecule clusters) as the most prevalent class, matching the paper's reported pattern
- Histogram of x shows a prevalent majority of recognized cluster ions with x=0 (near-absence of potassium-containing clusters)
- Counting is restricted to annotations from the 31 METASPACE datasets with DHB off-sample cluster ions, not the full 87-dataset gold standard

## Limitations

- The analysis characterizes only off-sample DHB matrix clusters and does not provide ground truth about the biological molecular content of samples
- Results depend on the quality of off-sample recognition (deep learning F1-score 0.97) and on the Keller and Li (2000) cluster-generating model, which enumerates 353 formulas for DHB specifically
- Conclusions are qualitative (slight prevalence of n=2, potassium parameter mostly 0) rather than quantitative rate estimates

## Evidence

- [intro] we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating"
- [readme] We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas available [here]: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas"
- [results] Recognized DHB clusters show a slight prevalence of clusters including two DHB molecules, and for a prevalent majority of recognized cluster ions the potassium parameter y is equal to 0.: [evidence span withheld — anchor status: not_found]
