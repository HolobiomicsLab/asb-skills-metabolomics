---
name: hierarchical-linkage-clustering
description: Use when you have a scored feature-pair matrix (e.g. normalized microbe-metabolite
  attribution scores) and need to group each feature axis into modules with shared
  interaction patterns before aggregating scores at module level.
when_to_use_negative:
- You only need single-feature (microbe-to-single-metabolite) interactions rather
  than groups of features sharing interaction patterns - use per-feature attribution
  or a method like NED whose latent-space interpretation does not reveal shared interaction-pattern
  modules.
- The interaction matrix has not yet been produced by the cross-validated MiMeNet
  models - clustering raw abundance tables instead of the attribution-score matrix
  will not yield interpretable modules.
- Metabolite abundances are being predicted directly rather than analyzed for network
  structure - use the MiMeNet prediction pipeline, not module biclustering.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3452
tools:
- name: MiMeNet
  role: Produces the cross-validated microbe-metabolite interaction-score matrix and
    provides the biclustering/consensus-clustering step that groups microbes and metabolites
    into modules before constructing the module interaction network.
  repo: github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:e66e2331da5ff0e70e7f38c3b8a9b577cce6f49dd30ebc0caa67a3e4b39a284a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMeNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# hierarchical-linkage-clustering

## Summary

Applies consensus clustering with Euclidean distance and complete linkage to split the MiMeNet microbe-metabolite interaction-score matrix into microbe and metabolite modules, selecting the cluster number via a delta-area threshold of 0.025.

## When to use

Use when you have a scored feature-pair matrix (e.g. normalized microbe-metabolite attribution scores) and need to group each feature axis into modules with shared interaction patterns before aggregating scores at module level.

## When NOT to use

- You only need single-feature (microbe-to-single-metabolite) interactions rather than groups of features sharing interaction patterns - use per-feature attribution or a method like NED whose latent-space interpretation does not reveal shared interaction-pattern modules.
- The interaction matrix has not yet been produced by the cross-validated MiMeNet models - clustering raw abundance tables instead of the attribution-score matrix will not yield interpretable modules.
- Metabolite abundances are being predicted directly rather than analyzed for network structure - use the MiMeNet prediction pipeline, not module biclustering.

## Inputs

- Normalized microbe-metabolite interaction-score matrix (samples/aggregated attribution scores, microbes x metabolites)
- Candidate cluster-number range for delta-area evaluation

## Outputs

- Microbe module assignments (8 modules at k1* on IBD PRISM)
- Metabolite module assignments (8 modules at k2*)
- Mean-normalized 8x8 block attribution matrix
- Weighted bipartite microbe-module / metabolite-module network in Cytoscape-compatible format

## How to apply

Load the normalized microbe x metabolite interaction-score matrix produced by cross-validated MiMeNet models. Run consensus clustering on the microbe axis: compute Euclidean distances, apply complete-linkage hierarchical clustering, evaluate the delta-area criterion across candidate cluster numbers, and select k1* where the delta-area falls below 0.025 (giving 8 microbe modules on the IBD PRISM data). Repeat the identical procedure on the metabolite axis to select k2* (8 metabolite modules). Assign every feature to its module, then aggregate the interaction matrix into a k1* x k2* block structure and score each module-pair edge by the mean normalized attribution within that block. The rationale is that biclustering groups microbes and metabolites with similar interaction patterns and functions, enabling 'Guilt by Association' interpretation of uncharacterized metabolites and construction of a weighted bipartite module-module network.

## Related tools

- **MiMeNet** (Produces the cross-validated microbe-metabolite interaction-score matrix and provides the biclustering/consensus-clustering step that groups microbes and metabolites into modules before constructing the module interaction network.) — github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro microbes_RA.csv -metab metabolites_RA.csv -output out_dir -micro_norm RA -metab_norm CLR -threshold 0.025 -num_run_cv 10
```

## Evaluation signals

- Selected microbe and metabolite module counts (k1* and k2*) match the consensus-clustering delta-area threshold of 0.025 applied on each feature axis.
- The exported network file contains exactly 8 microbial module nodes and 8 metabolomic module nodes on the IBD PRISM dataset.
- Every edge weight in the module-module network equals the mean normalized attribution score computed within the corresponding microbe-module / metabolite-module block of the original matrix.
- The interaction-score matrix is fully partitioned: each microbe and each metabolite appears in exactly one module, and no feature is dropped during biclustering.
- The module network is written as a Cytoscape-compatible file whose nodes separate into two disjoint sets (microbe vs metabolite modules) with no intra-set edges.

## Limitations

- Consensus clustering with a delta-area cutoff of 0.025 is a heuristic; a poorly calibrated threshold can under- or over-partition features into modules.
- MiMeNet (and by extension its module analysis) is limited in providing biological plausibility and mechanistic insights, so module groupings are patterns of co-prediction rather than validated causal pathways.
- The clustering is performed on attribution scores, not direct biochemical evidence; uncharacterized metabolites grouped with known ones rely on the 'Guilt by Association' assumption and require downstream validation.
- Metadata fields such as publication details (TBA) and explicit hyperparameter tables are not yet documented in the repository, so exact reproducibility depends on the article's reported thresholds.

## Evidence

- [methods] This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns.: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [abstract] MiMeNet can group microbes and metabolites with similar interaction patterns and functions to illuminate the underlying structure of the microbe-metabolite interaction network: "MiMeNet can group microbes and metabolites with similar interaction patterns and functions to illuminate the underlying structure of the microbe-metabolite interaction network"
- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [supplementary] Using consensus clustering analysis for cluster number determination for the IBD (PRISM) dataset.: "Using consensus clustering analysis for cluster number determination for the IBD (PRISM) dataset."
