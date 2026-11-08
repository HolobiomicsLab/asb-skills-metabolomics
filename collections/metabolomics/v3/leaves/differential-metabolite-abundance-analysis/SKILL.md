---
name: differential-metabolite-abundance-analysis
description: Use when joint spatial transcriptomics–metabolomics data have been integrated
  and clustered by SpatialMETA and you need cluster-defining genes/metabolites plus
  metabolite pathway enrichment for one sub-cluster vs. the rest.
when_to_use_negative:
- Data are single-cell or subcellular resolution — SpatialMETA is explicitly designed
  for low-resolution ST and SM data only
- SM and ST spots have not yet been aligned and KNN-reassigned (rotation, translation,
  non-linear distortion, spot_align_byknn), so cluster labels cannot be attached to
  metabolite intensities
- More than two modalities are present — SpatialMETA currently supports integration
  of only two modalities
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SpatialMETA
  role: CVAE-based framework whose joint ST-SM embedding, integrated clusters, and
    pp functions (rank_gene_and_metabolite_groups, calculate_metabolite_enrichment)
    perform the differential and enrichment analysis
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: DestVI
  role: Deconvolution of spatial transcriptomics against the ccRCC scRNA-seq reference
    to assign immune sub-cluster identity (Imm_4) used as the test group
- name: Human Metabolome Database (HMDB)
  role: Reference for metabolite annotation (adduct-based m/z matching with ppm tolerance)
    and pathway definitions used by the hypergeometric enrichment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:939ea09a6fdeb3796e5fcff81291dd02dcaa5435523a18e77df813cb25fe4477
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
  - DestVI
  - Human Metabolome Database (HMDB)
  - SpatialMETA
  - calculate_metabolite_enrichment
  - plot_marker_gene_metabolite
  - rank_gene_and_metabolite_groups
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# differential-metabolite-abundance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies marker genes and marker metabolites that define a spatial sub-cluster (e.g. the Imm_4 immune sub-cluster in ccRCC Y7_T) in a joint ST-SM AnnData, then tests marker metabolites for pathway enrichment via a hypergeometric test over HMDB-referenced annotations to establish metabolite-level (e.g. lipid metabolism) signatures.

## When to use

Use when joint spatial transcriptomics–metabolomics data have been integrated and clustered by SpatialMETA and you need cluster-defining genes/metabolites plus metabolite pathway enrichment for one sub-cluster vs. the rest.

## When NOT to use

- Data are single-cell or subcellular resolution — SpatialMETA is explicitly designed for low-resolution ST and SM data only
- SM and ST spots have not yet been aligned and KNN-reassigned (rotation, translation, non-linear distortion, spot_align_byknn), so cluster labels cannot be attached to metabolite intensities
- More than two modalities are present — SpatialMETA currently supports integration of only two modalities

## Inputs

- Joint ST-SM AnnData from spatialmeta.pp.joint_adata_sm_st with integrated Leiden cluster labels and modality contribution scores
- Cluster labels for the spatial sub-cluster of interest (e.g. Imm_4) assigned via DestVI deconvolution + manual marker annotation
- Metabolite features annotated against the Human Metabolome Database (HMDB) with user-defined adduct additions and ppm tolerance
- HMDB-referenced metabolite pathway annotation sets for the hypergeometric enrichment

## Outputs

- Differentially expressed gene (marker gene) table for the target cluster vs. rest (CSV)
- Marker metabolite table for the target cluster vs. rest (CSV)
- Metabolite pathway enrichment table from the hypergeometric test (CSV)
- Spatial PNG plots of marker gene and metabolite distributions (plot_marker_gene_metabolite, plot_spot_sm_st)

## How to apply

Load the joint ST-SM AnnData produced by spatialmeta.pp.joint_adata_sm_st, along with integrated Leiden cluster labels and modality contribution scores; optionally re-extract the shared latent embedding and modality contributions with ConditionalVAESTSM.get_latent_embedding and get_modality_contribution, and derive cluster identity from DestVI deconvolution against a ccRCC scRNA-seq reference with manual marker-based annotation. Before testing, remove batch-biased SVGs/SVMs — features with fold change greater than min_logfc (default 3), expressed in at least min_frac (default 0.8) of spots, shared across at least min_samples (default 2) samples — so batch effects do not masquerade as cluster markers. Run spatialmeta.pp.rank_gene_and_metabolite_groups on the target cluster (e.g. Imm_4) vs. all other clusters to produce paired differentially expressed gene and marker metabolite tables. Ensure metabolites carry HMDB annotations generated by user-defined adduct addition with a user-defined ppm tolerance, then run spatialmeta.pp.calculate_metabolite_enrichment, which applies a hypergeometric distribution test to assign pathway-level significance. Export the marker and enrichment tables as CSVs and visualize spatial distributions of markers with spatialmeta.pl.plot_marker_gene_metabolite and spatialmeta.pl.plot_spot_sm_st as PNG figures.

## Related tools

- **SpatialMETA** (CVAE-based framework whose joint ST-SM embedding, integrated clusters, and pp functions (rank_gene_and_metabolite_groups, calculate_metabolite_enrichment) perform the differential and enrichment analysis) — https://github.com/WanluLiuLab/SpatialMETA
- **DestVI** (Deconvolution of spatial transcriptomics against the ccRCC scRNA-seq reference to assign immune sub-cluster identity (Imm_4) used as the test group)
- **Human Metabolome Database (HMDB)** (Reference for metabolite annotation (adduct-based m/z matching with ppm tolerance) and pathway definitions used by the hypergeometric enrichment)

## Examples

```
import spatialmeta as smst; genes, mets = smst.pp.rank_gene_and_metabolite_groups(adata_sm_st, groups=['Imm_4'], reference='rest'); smst.pp.calculate_metabolite_enrichment(mets)
```

## Evaluation signals

- Top Imm_4 marker genes include CYP2J2 and ACSM2A and top marker metabolites include Phosphatidylglycerol and PA (24:0/24:1(15Z)), matching Fig. 3e-h
- Enrichment table shows positive enrichment for lipid metabolism terms in the Imm_4 markers, consistent with Fig. 3i
- Marker/enrichment tables contain valid hypergeometric test statistics for every reported metabolite group
- Batch-biased SVGs/SVMs (fold change > min_logfc default 3, present in ≥ min_frac 0.8 of spots across ≥ 2 samples) were removed before ranking
- Spatial PNG plots show co-localization of the reported metabolites with the Imm_4 spot mask in the Y7_T section

## Limitations

- Batch-biased SVGs/SVMs must be removed first; otherwise features with high cross-sample fold change (default min_logfc 3) can be mistaken for biological cluster markers
- Only two modalities are supported, and horizontal integration was validated on a limited number of samples from a single study
- Cluster identities depend on manual marker-based annotation after DestVI deconvolution; ground-truth labels were manually annotated from H&E features, not an independent gold standard

## Evidence

- [discussion] Removal of batch-biased SVGs/SVMs before differential ranking: [evidence span withheld — anchor status: not_found]
