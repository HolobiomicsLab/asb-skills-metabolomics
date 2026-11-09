---
name: cytoscape-network-format-export
description: Use when you have an aggregated microbe-module x metabolite-module attribution
  score matrix (e.g. the 8x8 block matrix from MiMeNet consensus biclustering) and
  need a Cytoscape-loadable network file.
when_to_use_negative:
- Input is a raw microbe x metabolite attribution matrix that has not yet been biclustered
  into modules — use the consensus-clustering/biclustering step first.
- Network is a microbe-microbe or metabolite-metabolite (non-bipartite) interaction
  network, which is a different graph structure from the module-module bipartite graph
  this skill exports.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MiMeNet
  role: Produces the microbe-metabolite attribution score matrix and biclusters it
    into modules; the module-block aggregation is then written out as a Cytoscape-compatible
    network file.
  repo: github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:7340a9bbcaa4c55ad326eab291b454dcf9143276982aeac89237e9c197b1c9de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMeNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cytoscape-network-format-export

## Summary

Export the biclustered microbe-metabolite module interaction network as a Cytoscape-compatible file so that weighted bipartite edges (mean attribution per module block) can be visualized and curated in Cytoscape.

## When to use

Use when you have an aggregated microbe-module x metabolite-module attribution score matrix (e.g. the 8x8 block matrix from MiMeNet consensus biclustering) and need a Cytoscape-loadable network file.

## When NOT to use

- Input is a raw microbe x metabolite attribution matrix that has not yet been biclustered into modules — use the consensus-clustering/biclustering step first.
- Network is a microbe-microbe or metabolite-metabolite (non-bipartite) interaction network, which is a different graph structure from the module-module bipartite graph this skill exports.

## Inputs

- Normalized microbe-metabolite interaction-score matrix (microbes x metabolites) from cross-validated MiMeNet models
- Microbial module assignments (8 modules from consensus clustering, delta-area threshold 0.025)
- Metabolite module assignments (8 modules from consensus clustering, delta-area threshold 0.025)

## Outputs

- Cytoscape-compatible network file of weighted bipartite module-module interactions (16 nodes: 8 microbe + 8 metabolite modules; edges weighted by mean block attribution)

## How to apply

After consensus clustering (Euclidean distance, complete linkage, delta-area threshold 0.025) assigns microbes and metabolites into 8 microbe modules and 8 metabolite modules, aggregate the normalized interaction-score matrix into the module-pair blocks and score each bipartite edge by the mean normalized attribution within that block. Write the resulting network — nodes = microbial and metabolomic modules, edges weighted by block-mean attribution — as a Cytoscape-compatible file. Validation: confirm the exported network file exists and contains 8 microbe module nodes and 8 metabolite module nodes with edges weighted by mean normalized block attribution.

## Related tools

- **MiMeNet** (Produces the microbe-metabolite attribution score matrix and biclusters it into modules; the module-block aggregation is then written out as a Cytoscape-compatible network file.) — github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -output out_dir -micro_norm CLR -metab_norm CLR -threshold 0.025 -num_run_cv 10 -num_cv 10
```

## Evaluation signals

- Exported network file exists at the expected output path and is parseable by Cytoscape.
- Network contains exactly 8 microbe module nodes and 8 metabolite module nodes (16 nodes total).
- Every edge in the file carries a weight equal to the mean normalized attribution score within the corresponding module-pair block.
- k1* and k2* cluster counts were selected by the delta-area criterion with threshold 0.025 (both should resolve to 8 on the IBD PRISM data).
- Aggregated block matrix dimensions match k1* x k2* (8x8) and no module-pair block is missing from the export.

## Limitations

- MiMeNet biclustering yields consensus-based module counts (k1* = 8 microbe modules, k2* = 8 metabolite modules on the IBD PRISM dataset), so the exported network's size depends on the data's delta-area curve, not a fixed dimension.
- Edge weights are mean attribution within a block, which can obscure heterogeneity of individual microbe-metabolite interactions inside the module.
- The article notes that procedures to discern the statistical association of microbe-metabolite interactions with host disease do not yet exist, so the exported network is exploratory rather than statistically tested.

## Evidence

- [methods] Consensus biclustering into modules: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [supplementary] Consensus clustering threshold: "Using consensus clustering analysis for cluster number determination for the IBD (PRISM) dataset."
- [methods] Threshold parameter for correlation: "-threshold THRESHOLD              Define significant correlation threshold"
- [methods] Cross-validated interaction score matrix source: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [full_text] Workflow step detail from source card: [evidence span withheld — anchor status: not_found]
