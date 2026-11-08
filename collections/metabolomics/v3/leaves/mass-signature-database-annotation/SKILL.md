---
name: mass-signature-database-annotation
description: Use when untargeted mass spectrometry imaging (e.g. DESI-MSI) yields
  unannotated m/z peaks—such as DIMPLE cluster signatures—and you need putative metabolite
  identities via database searching.
when_to_use_negative:
- Your m/z signatures are already annotated with confident metabolite IDs and you
  only need downstream biological interpretation.
- You are working with annotated, targeted metabolomics feature tables rather than
  untargeted MSI peaks.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: METASPACE
  role: Primary open-source cloud platform for spatial data interpretation used to
    annotate unknown mass signatures from MSI, accessible via the MSiReader v1.03
    plug-in.
- name: Metlin Classic
  role: Metabolite search engine used to obtain putative hits for m/z values that
    METASPACE does not resolve.
- name: LOTUS
  role: Open-source natural products identification tool used as a complementary annotation
    resource for m/z signatures.
- name: MSiReader v1.03
  role: MSI analysis software whose plug-in connects the imaging dataset to METASPACE
    database searching for peak annotation.
- name: DESI-MSI
  role: Imaging modality producing the spatially resolved m/z signatures that are
    annotated through these databases.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:891e1459da0212ded2e224211005d44ef43330885707e575205e10ea1c53468b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - LOTUS
  - METASPACE
  - MSiReader v1.03
  - Metlin Classic
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# mass-signature-database-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate unknown m/z signatures from DESI-MSI data by searching METASPACE, Metlin Classic, and LOTUS, as done for DIMPLE-derived maize-root mass signatures such as Cluster 4 cortex- and vasculature-enriched peaks.

## When to use

Use when untargeted mass spectrometry imaging (e.g. DESI-MSI) yields unannotated m/z peaks—such as DIMPLE cluster signatures—and you need putative metabolite identities via database searching.

## When NOT to use

- Your m/z signatures are already annotated with confident metabolite IDs and you only need downstream biological interpretation.
- You are working with annotated, targeted metabolomics feature tables rather than untargeted MSI peaks.

## Inputs

- List of unannotated m/z mass signatures from DESI-MSI (e.g. DIMPLE cluster assignments for B73 root Clusters 3–4)
- MSiReader v1.03 MSI dataset with METASPACE plug-in access

## Outputs

- Annotated table of m/z values with putative metabolite identifications (e.g. Cluster 4 cortex- and vasculature-enriched signatures)
- Database search hits from METASPACE, Metlin Classic, and LOTUS tied to each m/z

## How to apply

Take the list of unannotated mass signatures (m/z values) from your MSI analysis, e.g. DIMPLE cluster outputs from maize root DESI-MSI data, and query METASPACE, which is an open-source cloud platform for spatial data interpretation with a plug-in available in MSiReader v1.03. For peaks not resolved by METASPACE, cross-search Metlin Classic, a metabolite search engine that returns hits for m/z values, and LOTUS, an open-source natural products identification tool. Apply this primarily to spatially enriched signatures—such as Cluster 4 cortex- or vasculature-enriched peaks—so annotations can be tied to biological context (cell layer or developmental zone). Compile the annotated m/z values into tables alongside their enrichment evidence, mirroring the paper's Supplemental Figures 6–7 and Supplemental Table 7.

## Related tools

- **METASPACE** (Primary open-source cloud platform for spatial data interpretation used to annotate unknown mass signatures from MSI, accessible via the MSiReader v1.03 plug-in.)
- **Metlin Classic** (Metabolite search engine used to obtain putative hits for m/z values that METASPACE does not resolve.)
- **LOTUS** (Open-source natural products identification tool used as a complementary annotation resource for m/z signatures.)
- **MSiReader v1.03** (MSI analysis software whose plug-in connects the imaging dataset to METASPACE database searching for peak annotation.)
- **DESI-MSI** (Imaging modality producing the spatially resolved m/z signatures that are annotated through these databases.)

## Evaluation signals

- Annotated signature counts reproduce the reported quantities: at least 16 bimodal signatures, more than 15 cortex-enriched, and more than 10 vasculature-enriched signatures.
- Compiled tables mirror the content of Supplemental Figures 6–7 and Supplemental Table 7.
- Each m/z entry has hits reported from METASPACE, Metlin Classic, and/or LOTUS rather than remaining unannotated.
- Annotations are tied to spatial enrichment evidence (e.g. Cluster 4 cortex vs. vasculature channel localization) rather than provided as bare m/z lists.

## Limitations

- A major remaining challenge is that the majority of metabolites in both plants and animals remain unannotated, so many m/z values will not return confident hits.
- Untargeted MSI datasets are large (each pixel contains a full mass spectrum with hundreds to thousands of peaks), which complicates annotation workflows.

## Evidence
