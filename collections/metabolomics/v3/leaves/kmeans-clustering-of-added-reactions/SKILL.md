---
name: kmeans-clustering-of-added-reactions
description: Use when you have binary added-reaction sets from multiple gap-filling
  runs (e.g., COMMIT community gap filling vs. individual gap filling across draft
  reconstruction types) and want to compare solution composition structure.
when_to_use_negative:
- The gap-filling runs have not yet been executed, so there are no added-reaction
  sets to cluster — run COMMIT and individual gap filling first.
- The goal is to quantify metabolic cross-feeding interactions rather than compare
  gap-filling solution compositions — use the SMETANA-based downstream interaction
  analysis instead.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: COMMIT
  role: Produces the community-aware added-reaction sets that constitute the observations
    being clustered
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: MATLAB environment in which the COMMIT and individual gap-filling runs generating
    the added-reaction sets are executed
  repo: https://github.com/opencobra/cobratoolbox
- name: SMETANA
  role: Complementary downstream analysis of the gap-filled models; interaction analysis
    contrasts with the compositional clustering of added-reaction sets
  repo: https://github.com/cdanielmachado/smetana
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:1acabc77b1068cb28cccdf74760ebca2171e450e880cb2d2a085b8edb9d8127f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox
  - COMMIT
  - Gurobi
  - IBM CPLEX
  - SMETANA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kmeans-clustering-of-added-reactions

## Summary

Cluster the reaction sets added during COMMIT community-dependent gap filling versus individual gap filling using K-means with K=3, to reveal whether added-reaction composition groups by reconstruction type and gap-filling method, and visualize the structure with the R packages pheatmap, igraph, and ape.

## When to use

Use when you have binary added-reaction sets from multiple gap-filling runs (e.g., COMMIT community gap filling vs. individual gap filling across draft reconstruction types) and want to compare solution composition structure.

## When NOT to use

- The gap-filling runs have not yet been executed, so there are no added-reaction sets to cluster — run COMMIT and individual gap filling first.
- The goal is to quantify metabolic cross-feeding interactions rather than compare gap-filling solution compositions — use the SMETANA-based downstream interaction analysis instead.

## Inputs

- Added-reaction sets from COMMIT community-dependent gap filling of the Schlaeppi and Bulgarelli consensus reconstructions
- Added-reaction sets from individual gap filling of the same reconstructions (gap_fill_individual_models.m)
- Metadata labeling each set by reconstruction type (consensus source: KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) and gap-filling method

## Outputs

- K-means cluster assignments (K=3) for each added-reaction set, grouped by reconstruction type and gap-filling method
- Binary presence/absence matrix of added reactions per gap-filling solution
- Heatmap and clustering visualizations produced with the R packages pheatmap, igraph, and ape

## How to apply

First run COMMIT gap filling (100 iterations against the Universal-model-MNXref-balanced database with permeability rules HBD<=5, HBA<=10, MW<=500 Da, TPSA<=140 Angstrom, RB<=10, -0.4<=XLogP3<=5.6) and individual gap filling on the Schlaeppi (24 isolates) and Bulgarelli (20 isolates) consensus reconstructions, recording which reactions were added to each model. Convert each added-reaction set into a binary presence/absence row indexed by reconstruction type (e.g., KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools consensus) and gap-filling method (COMMIT vs. individual). Apply K-means with K=3 to this matrix so that solutions cluster by their added-reaction content rather than by imposed labels, then examine whether clusters align with reconstruction type or gap-filling method. Visualize the clustering with pheatmap (heatmap of the binary matrix), igraph (network of reaction-set overlaps), and ape (tree-like representation) in R. The rationale is to show that community-aware gap filling does not merely shrink solutions but yields qualitatively different added-reaction compositions.

## Related tools

- **COMMIT** (Produces the community-aware added-reaction sets that constitute the observations being clustered) — https://github.com/pwendering/COMMIT
- **COBRA toolbox** (MATLAB environment in which the COMMIT and individual gap-filling runs generating the added-reaction sets are executed) — https://github.com/opencobra/cobratoolbox
- **SMETANA** (Complementary downstream analysis of the gap-filled models; interaction analysis contrasts with the compositional clustering of added-reaction sets) — https://github.com/cdanielmachado/smetana

## Examples

```
km <- kmeans(added_reaction_matrix, centers = 3); pheatmap(added_reaction_matrix, annotation_col = model_metadata)
```

## Evaluation signals

- The number of added reactions in COMMIT solutions is significantly smaller than in individual gap-filling solutions, while genomic support is unchanged — clusters should reflect this contrast.
- K-means is run with K=3 as specified; check that cluster labels can be compared against the reconstruction-type and gap-filling-method metadata.
- The binary added-reaction matrix must have one row per gap-filled model from both soil communities (Schlaeppi, Bulgarelli) and both gap-filling methods.
- Visualizations from pheatmap, igraph, and ape render consistently from the same matrix (no conflicting cluster structures).

## Limitations

- Penalty weights used in gap filling (transport 100, metabolic 50, directionality change 25, permeable transport 50, sequence support 25, uptake of secreted metabolites 1, exchange 10^5) were tuned ad hoc until satisfying results, so cluster structure inherits that tuning bias.
- Comparison of reaction sets is hampered by the lack of reaction cross-references to name spaces other than self-defined identifiers; translations to MNXref were only done whenever possible, which can fragment sets and affect clustering.
- The full COMMIT procedure specification is deferred to supplementary S1 Text, so exact set definitions needed for reproducible clustering may require consulting the supplement.

## Evidence

- [intro] COMMIT decreased number of added reactions vs individual gap filling: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [full_text] R packages for visualization of added-reaction sets: [evidence span withheld — anchor status: not_found]
- [readme] Gap filling performed on consensus reconstructions of two soil communities: "community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions"
- [discussion] Ad hoc tuning of gap-filling penalty weights: "The penalties applied in this study resulted from the comparison of different weights that were altered until satisfying results"
- [intro] SMETANA downstream analysis of gap-filled models: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
