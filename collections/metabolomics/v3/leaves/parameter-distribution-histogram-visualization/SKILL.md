---
name: parameter-distribution-histogram-visualization
description: Use when you have a set of recognized (e.g., off-sample-flagged) matrix
  cluster ion formulas and a predicted formula-to-parameter lookup table, and need
  to visualize which combinatorial parameter values dominate the clusters.
when_to_use_negative:
- You are analyzing on-sample (tissue-relevant) metabolite annotations rather than
  off-sample matrix cluster ions — the parameter interpretation only applies to matrix
  clusters.
- You lack a formula-to-parameter mapping (e.g., the Keller and Li 2000 combinatorial
  model predictions) for the matrix of interest, so formulas cannot be decomposed
  into n, p, x, z.
- Your goal is pixel-level segmentation of off-sample regions, which the paper explicitly
  states this approach does not solve.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: METASPACE
  role: Source of public imaging MS datasets and the pixel-annotation export containing
    recognized off-sample DHB cluster ion formulas
  repo: https://github.com/metaspace2020/metaspace/
- name: matplotlib
  role: Plotting the four histograms of combinatorial parameters n, p, x, z in the
    Fig. 3 layout (named in the task workflow)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:30de1ee33fdcd6ea6386235ac6a29b41efaed652f80ffdbfb6927d4cef4af70b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# parameter-distribution-histogram-visualization

## Summary

Characterize recognized DHB matrix cluster ions by mapping each annotated molecular formula to its combinatorial cluster-generating model parameters (n = DHB molecule count, p, x = potassium count, z = sodium count), then tally parameter occurrences and plot per-parameter histograms in the layout of Fig. 3 of Ovchinnikova et al. (2020).

## When to use

Use when you have a set of recognized (e.g., off-sample-flagged) matrix cluster ion formulas and a predicted formula-to-parameter lookup table, and need to visualize which combinatorial parameter values dominate the clusters.

## When NOT to use

- You are analyzing on-sample (tissue-relevant) metabolite annotations rather than off-sample matrix cluster ions — the parameter interpretation only applies to matrix clusters.
- You lack a formula-to-parameter mapping (e.g., the Keller and Li 2000 combinatorial model predictions) for the matrix of interest, so formulas cannot be decomposed into n, p, x, z.
- Your goal is pixel-level segmentation of off-sample regions, which the paper explicitly states this approach does not solve.

## Inputs

- Recognized off-sample DHB cluster ion annotations from 31 METASPACE datasets (METASPACE pixel-annotation export, pixel-annot-export-v0.10.tar.gz)
- 353 predicted DHB matrix cluster molecular formulas with combinatorial model parameters n, p, x, z (DHB mc mol formulas, predicted.csv, from github.com/metaspace2020/offsample, per Keller and Li, 2000)

## Outputs

- Tally table of occurrence counts for each value of the parameters n, p, x, and z across all recognized DHB cluster ions
- Four-panel histogram figure (one panel per parameter n, p, x, z) reproducing the layout of Fig. 3

## How to apply

Load the recognized off-sample DHB cluster ion annotations from the 31 METASPACE datasets via the pixel-annot-export-v0.10.tar.gz export of the METASPACE knowledge base. Load the 353 predicted DHB matrix cluster molecular formulas generated per Keller and Li (2000) from the 'DHB mc mol formulas, predicted.csv' file in the offsample repository. Match each recognized cluster ion formula to its combinatorial model parameters n (number of DHB molecules), p, x (potassium count), and z (sodium count), then tally occurrence counts of each parameter value across all recognized clusters. Plot histograms of n, p, x, and z (e.g., with matplotlib) in a four-panel Fig. 3 layout with clearly labeled axes per parameter.

## Related tools

- **METASPACE** (Source of public imaging MS datasets and the pixel-annotation export containing recognized off-sample DHB cluster ion formulas) — https://github.com/metaspace2020/metaspace/
- **matplotlib** (Plotting the four histograms of combinatorial parameters n, p, x, z in the Fig. 3 layout (named in the task workflow))

## Examples

```
wget -qO - https://s3-eu-west-1.amazonaws.com/sm-off-sample/pixel-annot-export-v0.10.tar.gz | tar -xvz; python -c "import matplotlib.pyplot as plt; df=pd.read_csv('DHB mc mol formulas, predicted.csv'); [plt.hist(df[p]) for p in ['n','p','x','z']]"
```

## Evaluation signals

- Every recognized off-sample DHB cluster ion formula must find exactly one matching entry among the 353 predicted formulas; unmatched formulas should be inspected or reported.
- Histogram of n should show n=2 (clusters with two DHB molecules) as the most prevalent class, reproducing the paper's qualitative pattern.
- Histogram of x should show x=0 for a prevalent majority of recognized cluster ions, i.e., potassium-containing clusters (x>0) are nearly absent.
- All four panels (n, p, x, z) are present with clearly labeled axes and occurrence counts consistent with the tally table.
- Counts aggregated across the 31 METASPACE datasets, not per dataset, matching the paper's pooled analysis.

## Limitations

- The analysis is specific to the DHB matrix and the Keller and Li (2000) cluster-generating model; other MALDI matrices would require their own predicted formula sets.
- Findings are qualitative pattern-level claims (slight prevalence of two-DHB clusters, near-absence of potassium); they depend on which datasets were annotated and which annotations were recognized as off-sample.
- The approach characterizes cluster formulas at the annotation level and does not solve segmentation of off-sample pixels within images.

## Evidence

- [results] Recognized DHB clusters show a slight prevalence of clusters including two DHB molecules, and for a prevalent majority of recognized cluster ions the potassium parameter y is equal to 0.: [evidence span withheld — anchor status: not_found]
- [readme] DHB clusters generated per Keller and Li 2000: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas"
- [methods] METASPACE pixel annotation export: "https://s3-eu-west-1.amazonaws.com/sm-off-sample/pixel-annot-export-v0.10.tar.gz"
- [abstract] off-sample DHB investigation in abstract: "we investigated off-sample images corresponding to the most common MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and characterized properties of matrix clusters"
