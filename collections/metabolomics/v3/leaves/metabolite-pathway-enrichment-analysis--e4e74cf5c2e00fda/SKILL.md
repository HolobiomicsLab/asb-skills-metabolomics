---
name: metabolite-pathway-enrichment-analysis
description: Use when a cluster-level marker metabolite list (e.g. from spatialmeta.pp.rank_gene_and_metabolite_groups)
  needs pathway-level interpretation, such as testing whether Imm_4 marker metabolites
  like Phosphatidylglycerol and PA (24:0/24:1(15Z)) indicate upregulated lipid metabolic
  pathways.
when_to_use_negative:
- Input is raw SM mass-spectrometry intensity data that has not been integrated and
  clustered — marker metabolites must come from rank_gene_and_metabolite_groups on
  the joint AnnData first
- Metabolites cannot be matched to HMDB annotations because user-defined adducts and
  ppm tolerance are not specified
- High-resolution single-cell or subcellular data, since SpatialMETA is designed for
  low-resolution ST and SM data only
tools:
- name: SpatialMETA
  role: Provides spatialmeta.pp.calculate_metabolite_enrichment for hypergeometric
    pathway enrichment of marker metabolites over HMDB-referenced annotations, following
    marker calling with spatialmeta.pp.rank_gene_and_metabolite_groups
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:b8855e20efdd07effa6bc937429ecd54ba469337f38f8161ed62e0277402de53
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
  - SpatialMETA
  - calculate_metabolite_enrichment
  - plot_marker_gene_metabolite
  - rank_gene_and_metabolite_groups
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolite-pathway-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functional interpretation of marker metabolites identified from SpatialMETA-integrated ST-SM data by testing their over-representation in HMDB-referenced metabolic pathways with a hypergeometric distribution test, as applied to detect upregulated lipid metabolism in the ccRCC Imm_4 immune sub-cluster.

## When to use

Use when a cluster-level marker metabolite list (e.g. from spatialmeta.pp.rank_gene_and_metabolite_groups) needs pathway-level interpretation, such as testing whether Imm_4 marker metabolites like Phosphatidylglycerol and PA (24:0/24:1(15Z)) indicate upregulated lipid metabolic pathways.

## When NOT to use

- Input is raw SM mass-spectrometry intensity data that has not been integrated and clustered — marker metabolites must come from rank_gene_and_metabolite_groups on the joint AnnData first
- Metabolites cannot be matched to HMDB annotations because user-defined adducts and ppm tolerance are not specified
- High-resolution single-cell or subcellular data, since SpatialMETA is designed for low-resolution ST and SM data only

## Inputs

- Joint ST-SM AnnData from spatialmeta.pp.joint_adata_sm_st with integrated Leiden cluster labels (e.g. Imm_4)
- Marker metabolite table from spatialmeta.pp.rank_gene_and_metabolite_groups (Imm_4 vs. other clusters)
- Human Metabolome Database (HMDB) annotations with user-defined adduct additions and ppm tolerance

## Outputs

- Metabolite enrichment table (CSV) with hypergeometric test results over HMDB pathway groups
- Validated lipid-metabolism enrichment calls for the Imm_4 sub-cluster (cf. Fig. 3i)

## How to apply

First run spatialmeta.pp.rank_gene_and_metabolite_groups (e.g. Imm_4 vs. other clusters in the joint ST-SM AnnData from ConditionalVAESTSM integration) to obtain the marker metabolite table; verify expected markers such as Phosphatidylglycerol and PA (24:0/24:1(15Z)) appear among the top hits. Annotate metabolites by referencing the Human Metabolome Database (HMDB), applying user-defined adduct addition to generate new m/z values and a user-defined ppm tolerance for matching. Run spatialmeta.pp.calculate_metabolite_enrichment, which applies a hypergeometric distribution test over HMDB-referenced annotations, and save the enrichment table (e.g. as CSV). Judge success by confirming positive enrichment for lipid metabolism terms per Fig. 3i, consistent with the Imm_4 lipid-metabolic signature (CYP2J2, ACSM2A at the transcript level).

## Related tools

- **SpatialMETA** (Provides spatialmeta.pp.calculate_metabolite_enrichment for hypergeometric pathway enrichment of marker metabolites over HMDB-referenced annotations, following marker calling with spatialmeta.pp.rank_gene_and_metabolite_groups) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
import spatialmeta as smst; enrichment = smst.pp.calculate_metabolite_enrichment(imm4_marker_metabolites)
```

## Evaluation signals

- Enrichment table shows positive enrichment for lipid metabolism terms for Imm_4 (per Fig. 3i)
- Phosphatidylglycerol and PA (24:0/24:1(15Z)) are present among the Imm_4 marker metabolites entering the enrichment test (per Fig. 3e-h)
- Every tested metabolite maps to an HMDB entry within the user-defined ppm tolerance after adduct addition
- Enrichment p-values derive from the hypergeometric distribution test over HMDB-referenced annotations, not an ad-hoc scoring scheme
- Marker table and enrichment table are both exported (e.g. as CSV) and reproducible from the saved joint AnnData

## Limitations

- Enrichment is limited to HMDB-referenced annotations; metabolites outside HMDB coverage cannot be tested
- SM data are continuous mass-spectrometry intensities rather than sequencing counts, so metabolite annotation quality depends on user-defined adduct addition and ppm tolerance
- Ground-truth biological interpretation (e.g. lipid-metabolism upregulation in Imm_4) relies on manual marker-based annotation after DestVI deconvolution, not an independent gold standard

## Evidence

- [other] Perform lipid-metabolism pathway enrichment on the Imm_4 marker metabolites with spatialmeta.pp.calculate_metabolite_enrichment, which applies a hypergeometric distribution test over HMDB-referenced annotations: [evidence span withheld — anchor status: not_found]
