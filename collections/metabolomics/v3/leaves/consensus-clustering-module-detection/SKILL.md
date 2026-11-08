---
name: consensus-clustering-module-detection
description: Use when you have a microbe-by-metabolite interaction/attribution score
  matrix from MiMeNet and want to group features with shared interaction patterns
  into modules and build a module-level network.
when_to_use_negative:
- You only have per-metabolite prediction correlations (e.g. Spearman correlation
  coefficients from the cross-validated evaluation step) but no attribution/interaction-score
  matrix — this skill operates on the interaction-score matrix, not prediction accuracy
  alone.
- The neural network model is not MiMeNet (e.g. MelonnPan or BiomeNED/NED) — NED models
  metabolites via a latent space so its interaction interpretation is less intuitive
  and does not reveal shared interaction-pattern modules.
- You need mechanistic insight into individual microbe-metabolite interactions rather
  than module-level structure — the paper notes all such methods are limited in providing
  biological plausibility and mechanistic insights.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3173
tools:
- name: MiMeNet
  role: Produces the cross-validated microbe-metabolite attribution score matrix that
    this skill biclusters into modules; consensus clustering and block-mean network
    construction are downstream steps of the MiMeNet pipeline.
  repo: github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:6eaf0cce9a22a5d88f64b7d54e3e8fcae30f99d5f8eadacb2cb9ee1c63d1af1c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Consensus Clustering Module Detection

## Summary

Biclusters the microbe-metabolite interaction-score matrix produced by cross-validated MiMeNet models into microbial and metabolomic modules, then aggregates block-mean attribution scores into a weighted bipartite module-module network.

## When to use

Use when you have a microbe-by-metabolite interaction/attribution score matrix from MiMeNet and want to group features with shared interaction patterns into modules and build a module-level network.

## When NOT to use

- You only have per-metabolite prediction correlations (e.g. Spearman correlation coefficients from the cross-validated evaluation step) but no attribution/interaction-score matrix — this skill operates on the interaction-score matrix, not prediction accuracy alone.
- The neural network model is not MiMeNet (e.g. MelonnPan or BiomeNED/NED) — NED models metabolites via a latent space so its interaction interpretation is less intuitive and does not reveal shared interaction-pattern modules.
- You need mechanistic insight into individual microbe-metabolite interactions rather than module-level structure — the paper notes all such methods are limited in providing biological plausibility and mechanistic insights.

## Inputs

- Microbe-by-metabolite interaction/attribution score matrix (aggregated mean across cross-validated MiMeNet models), e.g. from the IBD paired metagenomic-metabolomic dataset
- Normalizations used upstream (RA or CLR transformations of the microbial and metabolomic feature tables)

## Outputs

- 8 microbe modules and 8 metabolite modules (module membership assignments for each feature)
- Weighted bipartite module-module network file compatible with Cytoscape, with edges scored by mean normalized attribution within each microbe-module / metabolite-module block

## How to apply

Start from the mean normalized interaction-score matrix (microbes as rows, metabolites as columns) aggregated across the cross-validated MiMeNet MLPNN models. Run consensus clustering independently on each axis: compute pairwise distances with Euclidean metric, perform hierarchical clustering with complete linkage, and evaluate candidate cluster numbers via the delta-area criterion, selecting the smallest k whose delta-area falls below the 0.025 threshold (this yields 8 microbial and 8 metabolite modules on the IBD data). Assign each microbe and metabolite to its module, then reshape the normalized interaction-score matrix into a k1-by-k2 block structure. Score each microbe-module / metabolite-module edge by the mean normalized attribution within that block, and export the resulting weighted bipartite network as a Cytoscape-compatible network file. The rationale is that microbes and metabolites with similar interaction patterns may share functions, supporting 'Guilt by Association' inference for uncharacterized metabolites.

## Related tools

- **MiMeNet** (Produces the cross-validated microbe-metabolite attribution score matrix that this skill biclusters into modules; consensus clustering and block-mean network construction are downstream steps of the MiMeNet pipeline.) — github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet.py -micro IBD_micro.csv -metab IBD_metab.csv -micro_norm RA -metab_norm RA -output results/ -threshold 0.025
```

## Evaluation signals

- The selected number of clusters on each axis should satisfy the delta-area < 0.025 threshold (expected 8 microbe modules and 8 metabolite modules for the IBD dataset).
- The exported network file must exist and contain 8 microbial module nodes and 8 metabolite module nodes, with edge weights equal to the mean normalized attribution within each corresponding microbe-module / metabolite-module block.
- Edge weights in the module-module network should be in [0, 1] since they are means of normalized interaction scores.
- Every microbe and metabolite feature from the input matrix must be assigned to exactly one module on its respective axis (complete partition, no orphans).
- The resulting network is bipartite: edges connect only microbe-module nodes to metabolite-module nodes, never nodes of the same feature type.

## Limitations

- The delta-area threshold of 0.025 for cluster number selection was determined via consensus clustering on the IBD (PRISM) dataset; other datasets (cystic fibrosis, soil) may require re-tuning, and the paper notes a higher threshold value was observed for the soil data possibly due to longitudinal observations.
- Module grouping illuminates shared interaction patterns but does not itself provide biological plausibility or mechanistic insight — the paper states 'all these methods including MiMeNet, are still limited in providing biological plausibility and mechanistic insights.'
- Missing or incorrect annotations upstream can lead to poor-performing models, which would propagate degraded interaction scores into the modules.

## Evidence

- [methods] This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns.: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [supplementary] Using consensus clustering analysis for cluster number determination for the IBD (PRISM) dataset.: "Using consensus clustering analysis for cluster number determination for the IBD (PRISM) dataset."
- [abstract] MiMeNet can group microbes and metabolites with similar interaction patterns and functions, potentially shedding light on uncharacterized metabolites via 'Guilt by Association': "MiMeNet can group microbes and metabolites with similar interaction patterns and functions to illuminate the underlying structure of the microbe-metabolite interaction network"
- [intro] The MiMeNet package and the datasets used for evaluation are freely available at https://github.com/YDaiLab/MiMeNet: "The MiMeNet package and the datasets used for evaluation are freely available at https://github.com/YDaiLab/MiMeNet"
- [discussion] all these methods including MiMeNet, are still limited in providing biological plausibility and mechanistic insights.: "Despite the progress, all these methods including MiMeNet, are still limited in providing biological plausibility and mechanistic insights."
