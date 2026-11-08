---
name: differential-gene-expression-analysis
description: Use when you have a joint ST-SM AnnData with integrated Leiden cluster
  labels and need cluster-specific marker genes and metabolites (e.g., Imm_4 markers
  CYP2J2, ACSM2A, Phosphatidylglycerol, PA (24:0/24:1(15Z))).
when_to_use_negative:
- Data lack joint ST-SM integration or cluster labels — differential testing needs
  a pre-integrated joint AnnData with Leiden clusters and modality contributions
- Single-cell or subcellular resolution data — SpatialMETA is designed for low-resolution
  ST and SM spots and does not support single-cell resolution
- More than two modalities need integration — SpatialMETA currently supports only
  ST and SM
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
tools:
- name: SpatialMETA
  role: Framework providing rank_gene_and_metabolite_groups for cluster-vs-rest differential
    testing on joint ST-SM data and calculate_metabolite_enrichment for HMDB hypergeometric
    pathway enrichment
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: ConditionalVAESTSM
  role: Supplies get_latent_embedding and get_modality_contribution used to select
    and verify target cluster spots before differential ranking
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: DestVI
  role: Deconvolution against the ccRCC scRNA-seq reference to assign immune identity
    to spatial clusters prior to marker ranking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:a9db938da734942b9e56aef9135db0a0d15808de3880e154b070617161b8aabb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  repo_url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - ConditionalVAESTSM
  - DestVI
  - SpatialMETA
  - plot_marker_gene_metabolite
  - rank_gene_and_metabolite_groups
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify differentially expressed genes and marker metabolites for a selected spatial cluster (e.g., the Imm_4 immune sub-cluster in ccRCC Y7_T) versus other clusters using SpatialMETA's rank_gene_and_metabolite_groups on the joint ST-SM AnnData, then validate lipid-metabolism biology via HMDB-referenced hypergeometric metabolite enrichment.

## When to use

Use when you have a joint ST-SM AnnData with integrated Leiden cluster labels and need cluster-specific marker genes and metabolites (e.g., Imm_4 markers CYP2J2, ACSM2A, Phosphatidylglycerol, PA (24:0/24:1(15Z))).

## When NOT to use

- Data lack joint ST-SM integration or cluster labels — differential testing needs a pre-integrated joint AnnData with Leiden clusters and modality contributions
- Single-cell or subcellular resolution data — SpatialMETA is designed for low-resolution ST and SM spots and does not support single-cell resolution
- More than two modalities need integration — SpatialMETA currently supports only ST and SM

## Inputs

- Joint ST-SM AnnData (h5ad) from spatialmeta.pp.joint_adata_sm_st with integrated Leiden cluster labels and modality contribution scores
- Shared latent embedding and modality contributions from ConditionalVAESTSM.get_latent_embedding / get_modality_contribution
- Cluster label assignments (Imm_4 immune sub-cluster) from DestVI deconvolution plus manual marker-based annotation
- HMDB-referenced metabolite annotations with user-defined adduct addition and ppm tolerance

## Outputs

- CSV table of differentially expressed genes for the target cluster (expecting CYP2J2 and ACSM2A among top markers)
- CSV table of marker metabolites (expecting Phosphatidylglycerol and PA (24:0/24:1(15Z)))
- Metabolite pathway enrichment table from the hypergeometric test over HMDB annotations
- PNG spatial plots of marker genes and metabolites (plot_marker_gene_metabolite, plot_spot_sm_st)

## How to apply

Load the joint ST-SM AnnData produced by spatialmeta.pp.joint_adata_sm_st with integrated Leiden cluster labels and modality contribution scores. Optionally re-extract the shared latent embedding and modality contributions via ConditionalVAESTSM.get_latent_embedding and ConditionalVAESTSM.get_modality_contribution to confirm the target spots. Select the spots of the cluster of interest (here Imm_4, annotated by manual marker-based annotation after DestVI deconvolution against the ccRCC scRNA-seq reference) and run spatialmeta.pp.rank_gene_and_metabolite_groups comparing that cluster against all others, saving both gene and metabolite marker tables. Follow with spatialmeta.pp.calculate_metabolite_enrichment, which applies a hypergeometric distribution test over HMDB-referenced annotations, to test whether marker metabolites are enriched for lipid metabolism pathways. Visualize spatial colocalization of markers with spatialmeta.pl.plot_marker_gene_metabolite and spatialmeta.pl.plot_spot_sm_st, exporting PNG figures and CSV tables.

## Related tools

- **SpatialMETA** (Framework providing rank_gene_and_metabolite_groups for cluster-vs-rest differential testing on joint ST-SM data and calculate_metabolite_enrichment for HMDB hypergeometric pathway enrichment) — https://github.com/WanluLiuLab/SpatialMETA
- **ConditionalVAESTSM** (Supplies get_latent_embedding and get_modality_contribution used to select and verify target cluster spots before differential ranking) — https://github.com/WanluLiuLab/SpatialMETA
- **DestVI** (Deconvolution against the ccRCC scRNA-seq reference to assign immune identity to spatial clusters prior to marker ranking)

## Examples

```
import spatialmeta as smst; smst.pp.rank_gene_and_metabolite_groups(adata, 'Imm_4', groupby='leiden'); smst.pp.calculate_metabolite_enrichment(marker_metabolites, reference='HMDB')
```

## Evaluation signals

- Imm_4 marker table contains CYP2J2 and ACSM2A among top-ranked genes (per Fig. 3e-f)
- Imm_4 metabolite marker table contains Phosphatidylglycerol and PA (24:0/24:1(15Z)) (per Fig. 3g-h)
- calculate_metabolite_enrichment output shows positive enrichment for lipid metabolism terms (per Fig. 3i)
- Spatial plots from plot_marker_gene_metabolite show the marker genes/metabolites colocalizing with the Imm_4 cluster's spatial footprint
- Marker CSV tables and enrichment CSV are produced with non-empty rows and valid test statistics

## Limitations

- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution
- Precise cell segmentation at cellular/subcellular levels is not implemented, limiting spot-level assignment of markers to individual cells
- Cluster identity relies on manual marker-based annotation after DestVI deconvolution; ground-truth labels were manually annotated from H&E features, not an independent gold standard

## Evidence

- [other] Run rank_gene_and_metabolite_groups on Imm_4 vs. other clusters: [evidence span withheld — anchor status: not_found]
