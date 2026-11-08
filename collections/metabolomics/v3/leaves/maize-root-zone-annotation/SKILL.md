---
name: maize-root-zone-annotation
description: Use when DESI-MSI mass signatures from linear maize-root developmental
  gradients need zone-level localization labels (e.g., meristem-localized Cluster
  1) or when comparing signature localization between varieties such as B73 and Oaxacan
  Green.
when_to_use_negative:
- The tissue lacks a linear developmental axis (branched organs, arbitrary 2D sections)
  — DIMPLE's zone clustering is designed to leverage the linear root tip-to-base gradient.
- The goal is compound-identity annotation of m/z values — use database searches (METASPACE,
  Metlin Classic, LOTUS) instead of developmental-zone clustering.
- Input is a bulk metabolomics feature table without spatial coordinates — zone localization
  cannot be inferred, and bulk tissue-scale data masks rare cell-type signatures.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0091
tools:
- name: DIMPLE
  role: Clusters mass-signature intensity linescans along the linear developmental
    axis into zone-enrichment clusters (e.g., Cluster 1 meristem-localized) whose
    assignments form the annotation and cross-variety comparison backbone.
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Acquires per-pixel mass spectra of longitudinal maize root sections at ~50–100
    µm spatial resolution, providing the spatial metabolite maps whose signatures
    are zone-annotated.
- name: batchcdfread
  role: Matlab function (bundled with the DIMPLE repository) that reads the cdf root-imaging
    files into Matlab workspaces for DIMPLE processing.
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:dbd93d64c065a9ced160eda12fbcca7b96b71bd9924f59c5c7b25b60393dfdc8
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
  - batchcdfread
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# maize-root-zone-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns developmental-zone localization classes (e.g., meristem-localized Cluster 1, root-tip, EZ cortex) to DESI-MSI mass signatures from maize root longitudinal sections using DIMPLE clusters along the linear tip-to-base axis, enabling cross-variety conservation/divergence calls between B73 and Oaxacan Green.

## When to use

Use when DESI-MSI mass signatures from linear maize-root developmental gradients need zone-level localization labels (e.g., meristem-localized Cluster 1) or when comparing signature localization between varieties such as B73 and Oaxacan Green.

## When NOT to use

- The tissue lacks a linear developmental axis (branched organs, arbitrary 2D sections) — DIMPLE's zone clustering is designed to leverage the linear root tip-to-base gradient.
- The goal is compound-identity annotation of m/z values — use database searches (METASPACE, Metlin Classic, LOTUS) instead of developmental-zone clustering.
- Input is a bulk metabolomics feature table without spatial coordinates — zone localization cannot be inferred, and bulk tissue-scale data masks rare cell-type signatures.

## Inputs

- DESI-MSI imaging data of longitudinal maize root sections as cdf files (e.g., B73-root11, B73-root2-5, B73-root-3-3, OG1, OG5, OG2-P11-0)
- Matlab workspaces with DIMPLE-processed linescans (e.g., B73-root11-workspace.mat, OG1-workspace-2.mat)
- DIMPLE cluster assignment tables mapping m/z to enrichment clusters (e.g., B73 Cluster 1 meristem-localized)

## Outputs

- Cluster-to-zone annotation per mass signature (m/z -> cluster -> developmental zone label such as meristem, root tip, EZ cortex)
- Cross-variety conservation/divergence table with columns: signature m/z, B73 cluster, Oaxacan Green cluster, conservation status
- Divergent-signature records (e.g., m/z 149.0118, m/z 280.2367) with variety-specific localization patterns

## How to apply

Acquire DESI-MSI images of longitudinal maize root sections (~50–100 µm spatial resolution) and read the cdf files into Matlab with batchcdfread (e.g., B73-root11, B73-root2-5, OG1, OG5), ordering pixels along the linear developmental axis. Run DIMPLE (Developmental Imaging Mass Spectrometry Pipeline for Linear Evaluation), which processes each mass signature's intensity linescan along this axis and bins signatures into clusters with distinct developmental enrichment patterns (e.g., Cluster 1 = meristem-localized). Label each cluster with its anatomical zone by inspecting the clustered linescans, then extract cluster membership lists such as the 48 B73 Cluster 1 meristem-localized signatures. For cross-variety annotation, locate each m/z in the second variety's cluster table and classify its enrichment as conserved or divergent, explicitly recording exemplars like m/z 149.0118 (root-tip specific in B73 vs broader Cluster 5 in Oaxacan Green) and m/z 280.2367 (root-tip enriched in B73 vs strongly EZ-cortex localized in Oaxacan Green). Compile a conservation/divergence table listing each signature, its B73 cluster, its Oaxacan Green cluster, and its conservation status, then validate counts (20 of the 48 Cluster 1 signatures called conserved) and confirm both divergent-m/z rows are present.

## Related tools

- **DIMPLE** (Clusters mass-signature intensity linescans along the linear developmental axis into zone-enrichment clusters (e.g., Cluster 1 meristem-localized) whose assignments form the annotation and cross-variety comparison backbone.) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Acquires per-pixel mass spectra of longitudinal maize root sections at ~50–100 µm spatial resolution, providing the spatial metabolite maps whose signatures are zone-annotated.)
- **batchcdfread** (Matlab function (bundled with the DIMPLE repository) that reads the cdf root-imaging files into Matlab workspaces for DIMPLE processing.) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
matlab -batch "data = batchcdfread('B73-root11.cdf'); save('B73-root11-workspace.mat','data')"
```

## Evaluation signals

- Zone-annotation table reports the expected cluster membership count (48 mass signatures in B73 Cluster 1, meristem-localized).
- Conservation table classifies exactly 20 of the 48 B73 Cluster 1 signatures as conserved in Oaxacan Green.
- Table contains rows for m/z 149.0118 and m/z 280.2367 with the correct variety-specific patterns (B73 root-tip vs broader Cluster 5 / EZ-cortex localization in Oaxacan Green).
- Every signature row carries both a B73 cluster assignment and an Oaxacan Green cluster assignment plus a conserved/divergent status label.
- Cluster labels agree with the clustered linescans' intensity maxima positions along the tip-to-base axis (root tip, meristem, elongation zone).

## Limitations

- Zone assignment is not compound identification — the article notes the majority of metabolites in plants and animals remain unannotated, so clusters localize mass signatures without naming metabolites.
- DESI-MSI spatial resolution (~50–100 µm) restricts calls to developmental-zone level; rare cell types such as stem cells can be masked at tissue scale.
- Localization is variety-specific and not transferable: only 20 of 48 B73 meristem-localized signatures were conserved in Oaxacan Green, and about half of those shared signatures no longer retain meristem localization.

## Evidence
