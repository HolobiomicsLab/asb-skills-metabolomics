---
name: metabolite-divergence-tabulation
description: Use when two DESI-MSI developmental-gradient datasets have already been
  clustered (e.g., via DIMPLE) and the goal is to classify which clustered mass signatures
  keep the same enrichment pattern across varieties and which have shifted localization.
when_to_use_negative:
- The MSI datasets have not yet been processed by DIMPLE — without cluster assignments
  and clustered linescans there is no enrichment pattern to compare across varieties.
- The analysis goal is metabolite identification/annotation of unknown m/z values
  (e.g., via METASPACE, Metlin Classic, or LOTUS) rather than cross-variety localization
  comparison.
- The samples are not comparable developmental gradients — this skill assumes a linear
  root developmental axis so clusters map between varieties.
tools:
- name: DIMPLE
  role: Provides the developmental enrichment clusters (e.g., B73 Cluster 1 meristem-localized
    signatures) and clustered linescans that are cross-referenced between B73 and
    Oaxacan Green.
  repo: https://github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: 'Source imaging modality: maps metabolites along the developing maize root
    gradient at ~50-100 µm spatial resolution, producing the mass spectra from which
    cluster-consistent mass signatures are derived.'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:20c0e696aba8abe0b8ba86f1b5142ee64894a195726bdde6c6f461b958d52e90
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  repo_url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolite-divergence-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-variety comparison workflow that maps DIMPLE cluster assignments of mass signatures from one maize variety (e.g., B73) onto another (e.g., Oaxacan Green) and tabulates each signature as conserved or divergent in developmental localization, exemplified by the B73 meristem-localized Cluster 1 set.

## When to use

Use when two DESI-MSI developmental-gradient datasets have already been clustered (e.g., via DIMPLE) and the goal is to classify which clustered mass signatures keep the same enrichment pattern across varieties and which have shifted localization.

## When NOT to use

- The MSI datasets have not yet been processed by DIMPLE — without cluster assignments and clustered linescans there is no enrichment pattern to compare across varieties.
- The analysis goal is metabolite identification/annotation of unknown m/z values (e.g., via METASPACE, Metlin Classic, or LOTUS) rather than cross-variety localization comparison.
- The samples are not comparable developmental gradients — this skill assumes a linear root developmental axis so clusters map between varieties.

## Inputs

- DIMPLE cluster assignments for B73 root DESI-MSI data (Cluster 1 meristem-localized mass signatures)
- DIMPLE cluster assignments for Oaxacan Green root DESI-MSI data
- Clustered linescans along the linear root developmental gradient for both varieties
- cdf files and Matlab workspaces shipped with DIMPLE-code (e.g., B73-root11-workspace.mat, OG1-workspace-2.mat)

## Outputs

- Conservation/divergence table with columns: signature m/z, B73 cluster, Oaxacan Green cluster, conservation status
- Explicitly annotated divergent metabolite records (e.g., m/z 149.0118, m/z 280.2367 with variety-specific localization patterns)

## How to apply

Load the DIMPLE cluster assignments and clustered linescans previously generated for both root developmental gradients (B73 and Oaxacan Green cdf-derived Matlab workspaces from the DIMPLE-code repository). Extract the 48 mass signatures assigned to Cluster 1 (meristem-localized enrichment) in B73, then for each signature locate its corresponding m/z in the Oaxacan Green clusters and classify its enrichment pattern as conserved or divergent. Explicitly record divergent metabolites with their variety-specific localizations, e.g., m/z 149.0118 (root-tip specific in B73 vs broader Cluster 5 localization in Oaxacan Green) and m/z 280.2367 (root-tip enriched in B73 vs strong specific localization to the EZ cortex in Oaxacan Green). Compile the per-signature classifications into a conservation/divergence table listing each signature, its B73 cluster, its Oaxacan Green cluster, and its conservation status.

## Related tools

- **DIMPLE** (Provides the developmental enrichment clusters (e.g., B73 Cluster 1 meristem-localized signatures) and clustered linescans that are cross-referenced between B73 and Oaxacan Green.) — https://github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Source imaging modality: maps metabolites along the developing maize root gradient at ~50-100 µm spatial resolution, producing the mass spectra from which cluster-consistent mass signatures are derived.)

## Examples

```
load('B73-root11-workspace.mat'); load('OG1-workspace-2.mat')  % MATLAB: load DIMPLE cluster workspaces to cross-map B73 Cluster 1 m/z into Oaxacan Green clusters
```

## Evaluation signals

- The conservation table reports 20 of the 48 B73 Cluster 1 signatures as conserved in Oaxacan Green.
- The table contains rows for both divergent m/z values (149.0118 and 280.2367) with their variety-specific localization patterns.
- Every row has non-empty entries for B73 cluster, Oaxacan Green cluster, and conservation status — no signature left unclassified.
- Divergent annotations correctly pair m/z 149.0118 with root-tip specific (B73) vs broader Cluster 5 (Oaxacan Green), and m/z 280.2367 with root-tip enriched (B73) vs EZ cortex enriched (Oaxacan Green).

## Limitations

- Only about half of conserved signatures retain meristem localization — conservation of a mass signature does not imply conservation of its developmental localization pattern.
- Most metabolites remain unannotated, so divergent signatures can typically only be reported as m/z values rather than named metabolites.
- Results depend on the DIMPLE clustering of each variety; the skill assumes both datasets were processed on the same linear developmental axis with comparable cluster definitions.

## Evidence
