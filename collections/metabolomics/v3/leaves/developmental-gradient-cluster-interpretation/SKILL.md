---
name: developmental-gradient-cluster-interpretation
description: Use when DIMPLE cluster outputs and clustered linescans from root DESI-MSI
  need biological interpretation — e.g. identifying meristem-localized Cluster 1 signatures,
  bimodal enrichment, or cortex/vasculature enrichment in B73 and Oaxacan Green.
when_to_use_negative:
- Input is raw DESI-MSI cdf data that has not yet been processed by DIMPLE into cluster
  assignments and linescans — run the DIMPLE pipeline first.
- The spatial axis is not a linear developmental gradient (e.g. isotropic tissue or
  bulk metabolomics), since DIMPLE and cluster interpretation depend on the linear
  root axis.
- The goal is quantifying a known metabolite by targeted MS rather than interpreting
  untargeted spatial enrichment patterns.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: DIMPLE
  role: Generates the clustered mass-signature assignments and linescans along the
    linear developmental gradient that this skill interprets via the GUI
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Source imaging modality producing the hyperspectral root sections whose mass
    signatures are clustered and interpreted
- name: METASPACE
  role: Plug-in via MSiReader v1.03 and standalone cloud platform used to annotate
    unknown Cluster 4 mass signatures
- name: Metlin Classic
  role: Metabolite search engine used to annotate cortex/vasculature-enriched m/z
    values
- name: LOTUS
  role: Open-source natural products tool used to annotate Cluster 4 signature m/z
    values
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:3c52afc339a36edd7af0a7ace50a1e1a2d1fe602ce5a3eba239ad84209a00074
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
  - Metlin Classic
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# developmental-gradient-cluster-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets DIMPLE cluster assignments on root DESI-MSI data by inspecting clustered linescans and the GUI to classify metabolite signatures as meristem-localized, bimodal, or cell-layer-enriched (cortex/vasculature) across developmental gradients and maize varieties.

## When to use

Use when DIMPLE cluster outputs and clustered linescans from root DESI-MSI need biological interpretation — e.g. identifying meristem-localized Cluster 1 signatures, bimodal enrichment, or cortex/vasculature enrichment in B73 and Oaxacan Green.

## When NOT to use

- Input is raw DESI-MSI cdf data that has not yet been processed by DIMPLE into cluster assignments and linescans — run the DIMPLE pipeline first.
- The spatial axis is not a linear developmental gradient (e.g. isotropic tissue or bulk metabolomics), since DIMPLE and cluster interpretation depend on the linear root axis.
- The goal is quantifying a known metabolite by targeted MS rather than interpreting untargeted spatial enrichment patterns.

## Inputs

- DIMPLE cluster assignments for B73 and Oaxacan Green root DESI-MSI hyperspectral channels
- Clustered linescans along the root developmental gradient
- Matlab workspace files from the DIMPLE-code repo (e.g. B73-root11-workspace.mat, OG1-workspace-2.mat)

## Outputs

- Conservation/divergence table (B73 cluster, Oaxacan Green cluster, status per m/z)
- Bimodal enrichment signature table for Clusters 3-4
- Cluster 4 cortex- vs vasculature-enriched annotated signature table mirroring Supplemental Table 7

## How to apply

Load DIMPLE clustered hyperspectral channel outputs and clustered linescans generated for B73 and Oaxacan Green root developmental gradients. For meristem-localized signatures, extract Cluster 1 assignments (e.g. 48 B73 signatures) and cross-reference each m/z against the other variety's clusters to label it conserved or divergent — e.g. m/z 149.0118 root-tip specific in B73 but broader Cluster 5 localization in Oaxacan Green, and m/z 280.2367 root-tip enriched in B73 vs EZ-cortex enriched in Oaxacan Green. For Clusters 3-4, inspect the linescans via the GUI to flag signatures with bimodal meristem/mature-tissue enrichment (≥16 expected) and, for Cluster 4, split signatures into cortex-enriched (>15) and vasculature-enriched (>10) bins. Annotate Cluster 4 m/z values with METASPACE, Metlin Classic, and LOTUS searches, then compile tables mirroring Supplemental Figures 5-7 and Supplemental Table 7.

## Related tools

- **DIMPLE** (Generates the clustered mass-signature assignments and linescans along the linear developmental gradient that this skill interprets via the GUI) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Source imaging modality producing the hyperspectral root sections whose mass signatures are clustered and interpreted)
- **METASPACE** (Plug-in via MSiReader v1.03 and standalone cloud platform used to annotate unknown Cluster 4 mass signatures)
- **Metlin Classic** (Metabolite search engine used to annotate cortex/vasculature-enriched m/z values)
- **LOTUS** (Open-source natural products tool used to annotate Cluster 4 signature m/z values)

## Examples

```
load B73-root11-workspace.mat; % open DIMPLE cluster GUI, inspect Cluster 4 linescans for cortex/vasculature-enriched m/z
```

## Evaluation signals

- Conservation/divergence table classifies 20 of the 48 B73 Cluster 1 signatures as conserved in Oaxacan Green
- Table contains explicit rows for m/z 149.0118 (B73 root-tip specific vs broader Oaxacan Green Cluster 5) and m/z 280.2367 (B73 root-tip enriched vs Oaxacan Green EZ cortex)
- Bimodal table contains at least 16 meristem/mature-enriched signatures
- Cluster 4 table reports >15 cortex-enriched and >10 vasculature-enriched signatures
- Compiled tables mirror Supplemental Figures 5-7 and Supplemental Table 7

## Limitations

- The majority of metabolites remain unannotated, so many interpreted mass signatures lack confirmed identities and must rely on database hits from METASPACE, Metlin Classic, and LOTUS.
- Conservation calls depend on matching m/z across varieties; divergent localization (e.g. retained m/z with changed cluster) is common, so cluster ID alone is insufficient.
- MSiReader is now commercial, and annotation coverage is a major remaining challenge across biological systems.

## Evidence

- [supplementary] Meristem-localized Cluster 1 signatures and divergent m/z classifications: [evidence span withheld — anchor status: not_found]
- [supplementary] Cluster 4 cell-layer counts: [evidence span withheld — anchor status: not_found]
