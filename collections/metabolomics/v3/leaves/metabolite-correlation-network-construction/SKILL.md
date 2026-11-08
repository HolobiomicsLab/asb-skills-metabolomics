---
name: metabolite-correlation-network-construction
description: Use when raw metabolite abundance profiles (e.g., tomato pericarp 2001/2003/2004
  seasons with missing values) must be converted into a weighted/thresholded correlation
  graph to feed hyperlink prediction or network-based pathway analysis.
when_to_use_negative:
- Input is already a validated network or adjacency structure — re-thresholding would
  duplicate the filtering step
- Profiles contain very few replicates per metabolite, making pairwise Pearson correlations
  and p-values statistically unreliable
- The downstream task requires directed reaction graphs (e.g., the directional pathway
  pool of 126 tomato pericarp pathways used in augmentation), not undirected correlation
  networks
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: pcaMethods
  role: Imputes missing metabolite abundance values via probabilistic PCA before correlation
    computation
- name: Rstudio
  role: Environment in which the pcaMethods-based imputation and correlation network
    construction is executed
- name: NetAurHPD
  role: Consumer of the constructed Metabolite CNs; the CNs are auralized and used
    to predict metabolic pathways
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  - task_005
  source_content_hash: sha256:856f79ca3f1ee2819dd8526987f7bb5eed026ae6e89931539306f278dddc1504
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - Adam optimizer
  - CHESHIRE
  - M5
  - NetAurHPD
  - Rstudio
  - pcaMethods
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-correlation-network-construction

## Summary

Constructs Metabolite correlation networks (Metabolite CNs) by imputing missing values in metabolomics profiles with probabilistic PCA (pcaMethods in Rstudio) and linking metabolite pairs whose Pearson Correlation Coefficient passes the |r| ≥ 0.3 and p ≤ 0.01 threshold. The resulting nodes-and-edges graphs are the input substrate for NetAurHPD's auralization and downstream pathway prediction.

## When to use

Use when raw metabolite abundance profiles (e.g., tomato pericarp 2001/2003/2004 seasons with missing values) must be converted into a weighted/thresholded correlation graph to feed hyperlink prediction or network-based pathway analysis.

## When NOT to use

- Input is already a validated network or adjacency structure — re-thresholding would duplicate the filtering step
- Profiles contain very few replicates per metabolite, making pairwise Pearson correlations and p-values statistically unreliable
- The downstream task requires directed reaction graphs (e.g., the directional pathway pool of 126 tomato pericarp pathways used in augmentation), not undirected correlation networks

## Inputs

- Normalized metabolite abundance profiles per harvesting season (tomato pericarp 2001, 2003, 2004), with missing values requiring imputation
- Biological replicate structure used to compute pairwise Pearson correlations

## Outputs

- Per-season Metabolite CN as a nodes-and-edges table (e.g., metabolite_corr_network.csv)
- Network statistics: node and edge counts per season (2001: 75 nodes/473 edges; 2003: 75 nodes/869 edges; 2004: 78 nodes/338 edges)

## How to apply

First normalize the metabolic profiles at both the sample and metabolite levels, then impute missing values using probabilistic PCA from the pcaMethods package in Rstudio. For each harvesting season (2001, 2003, 2004), compute the Pearson Correlation Coefficient between every pair of metabolites across biological replicates. Retain links only when the correlation passes the selected threshold |r| ≥ 0.3 and p ≤ 0.01. Export each year's network as a nodes-and-edges table (e.g., metabolite_corr_network.csv) recording node and edge counts per season. The threshold choice is a decision point: 0.3/0.01 is the setting validated in the NetAurHPD experiments, and it also governs the Augmentation CNs generated during data augmentation.

## Related tools

- **pcaMethods** (Imputes missing metabolite abundance values via probabilistic PCA before correlation computation)
- **Rstudio** (Environment in which the pcaMethods-based imputation and correlation network construction is executed)
- **NetAurHPD** (Consumer of the constructed Metabolite CNs; the CNs are auralized and used to predict metabolic pathways) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Evaluation signals

- Network sizes match reported reference values: 2001 = 75 metabolites/473 links, 2003 = 75 nodes/869 edges, 2004 = 78 nodes/338 edges
- Every retained edge satisfies |r| ≥ 0.3 and p ≤ 0.01; no edge below threshold survives in the exported table
- Missing values are imputed via probabilistic PCA before correlation; no NaN correlations in the output matrix
- Output CSV is a valid nodes-and-edges table with per-season node/edge counts recorded

## Limitations

- The |r| ≥ 0.3, p ≤ 0.01 threshold was calibrated for the tomato pericarp datasets; other organisms or seasons may need re-tuning
- Correlation networks are undirected and do not encode causal/directional reactions, unlike the directional pathway pool used for augmentation
- Imputation quality depends on the probabilistic PCA fit; poorly estimated metabolite profiles propagate into spurious or dropped correlations

## Evidence

- [methods] Correlation threshold filtering via Pearson coefficient: [evidence span withheld — anchor status: not_found]
- [methods] Pairwise correlation computation defines Metabolite CN links: [evidence span withheld — anchor status: not_found]
