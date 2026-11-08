---
name: mass-signature-conservation-assessment
description: Use when two DESI-MSI developmental-gradient datasets (e.
when_to_use_negative:
- When the two varieties' datasets have not yet been processed into enrichment clusters
  by DIMPLE — run the cluster-generation skill first, since this skill consumes cluster
  assignments, not raw cdf/NetCDF pixel data
- When the biological question is about absolute metabolite annotation or database
  ID assignment (Metlin Classic, LOTUS, METASPACE) rather than cross-variety comparison
  of localization patterns
- When the samples do not share a comparable linear developmental axis (e.g., non-root
  tissues or non-aligned gradients), since the classification relies on matching enrichment
  positions along the axis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DIMPLE
  role: Produces the per-variety cluster assignments and clustered linescans that
    this skill consumes; the conservation assessment compares signatures across those
    cluster outputs
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Source imaging modality that generates the spatial mass-spectrometry data
    underlying the clustered linescans being compared across B73 and Oaxacan Green
    roots
- name: batchcdfread
  role: Matlab function (developed by Yifan Meng in the Zare lab) used to load the
    NetCDF/cdf DESI-MSI root files into the workspaces that feed the cluster assignments
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:9a9feca3b72fac64c555e445a95c647a229b7f486a4497a8697dafeffbc8a784
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-signature-conservation-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-variety assessment that takes cluster assignments (e.g., DIMPLE Cluster 1 meristem-localized mass signatures) from one biological variety and classifies whether each signature's developmental enrichment pattern is conserved or divergent in a second variety, using paired clustered linescans from DESI-MSI data.

## When to use

Use when two DESI-MSI developmental-gradient datasets (e.g., maize B73 and Oaxacan Green roots) have already been processed into per-variety enrichment clusters, and you need to determine which mass signatures from one variety's cluster retain their localization in the other versus shift developmental zones.

## When NOT to use

- When the two varieties' datasets have not yet been processed into enrichment clusters by DIMPLE — run the cluster-generation skill first, since this skill consumes cluster assignments, not raw cdf/NetCDF pixel data
- When the biological question is about absolute metabolite annotation or database ID assignment (Metlin Classic, LOTUS, METASPACE) rather than cross-variety comparison of localization patterns
- When the samples do not share a comparable linear developmental axis (e.g., non-root tissues or non-aligned gradients), since the classification relies on matching enrichment positions along the axis

## Inputs

- DIMPLE cluster assignments for the reference variety (e.g., B73 Cluster 1 = 48 meristem-localized mass signatures)
- DIMPLE cluster assignments for the comparison variety (e.g., Oaxacan Green clusters with developmental enrichment patterns)
- Clustered linescans (m/z intensity vs. position along the root developmental axis) for both varieties
- Matlab workspaces and NetCDF/cdf files per root (B73-root11, B73-root2-5, B73-root-3-3; OG1, OG5, OG2-P11-0)

## Outputs

- Conservation/divergence table: one row per reference-cluster mass signature with columns for m/z, reference cluster, comparison cluster, and conservation status (conserved or divergent)
- Count of conserved signatures (e.g., 20 of 48 B73 Cluster 1 signatures conserved in Oaxacan Green)
- List of explicitly flagged divergent metabolites with their variety-specific localizations (e.g., m/z 149.0118, m/z 280.2367)

## How to apply

Load the per-variety DIMPLE cluster assignments and the clustered linescans generated for each root's linear developmental axis (B73-root11, B73-root2-5, B73-root-3-3; OG1, OG5, OG2-P11-0). Select the cluster of interest in the reference variety — here the 48 mass signatures in B73 Cluster 1 (meristem-localized enrichment). For each m/z in that reference cluster, locate the same m/z in the comparison variety's cluster assignments (Oaxacan Green) and classify its enrichment pattern: conserved if the same developmental localization is retained, divergent if it shifts to a different cluster or zone. Explicitly record divergent cases by m/z with both varieties' localizations — e.g., m/z 149.0118 is root-tip specific in B73 but Cluster 5 (broader) in Oaxacan Green, and m/z 280.2367 is root-tip enriched in B73 but EZ-cortex specific in Oaxacan Green. Compile the results into a conservation/divergence table with columns: mass signature (m/z), reference-variety cluster, comparison-variety cluster, and conservation status; report the count of conserved signatures (20 of 48) as a summary statistic.

## Related tools

- **DIMPLE** (Produces the per-variety cluster assignments and clustered linescans that this skill consumes; the conservation assessment compares signatures across those cluster outputs) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Source imaging modality that generates the spatial mass-spectrometry data underlying the clustered linescans being compared across B73 and Oaxacan Green roots)
- **batchcdfread** (Matlab function (developed by Yifan Meng in the Zare lab) used to load the NetCDF/cdf DESI-MSI root files into the workspaces that feed the cluster assignments) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
matlab -batch "W=readtable('cluster_assignments.csv'); b73=W(W.variety=='B73' & W.cluster==1,:); for i=1:height(b73), og=W(strcmp(W.mz,b73.mz{i}) & W.variety=='OG',:); disp([b73.mz{i}, num2str(b73.cluster(i)), num2str(og.cluster(1))]); end"
```

## Evaluation signals

- The output table contains exactly one row per mass signature in the reference cluster (48 rows for B73 Cluster 1)
- The count of signatures marked 'conserved' matches a reproducible recount from the paired cluster assignments (20 of 48 conserved in Oaxacan Green)
- Divergent rows are explicitly listed with m/z values and both varieties' cluster/zone labels — e.g., m/z 149.0118 (root-tip B73 vs Cluster 5 OG) and m/z 280.2367 (root-tip B73 vs EZ-cortex OG)
- Every m/z classified as conserved must appear with the same or equivalent developmental-zone enrichment in both varieties' cluster assignments
- Cluster assignments for both varieties are read from the same workspace/cdf files named in the README (B73-root11, OG1, etc.), not from mismatched roots

## Limitations

- Roughly half of the 20 conserved signatures do not retain meristem localization in Oaxacan Green, so 'conserved' here means the m/z was found in the other variety, not that its localization pattern is identical — both conservation level and localization divergence must be reported separately
- Classification depends on the reference variety's cluster definitions; if B73 Cluster 1 membership changes (e.g., a different root set), the entire table must be recomputed
- The comparison assumes the two varieties' developmental axes have been aligned to comparable zones (meristem, elongation zone, EZ cortex); misalignment inflates divergent classifications
- Mass signatures without database annotation remain functionally unknown, limiting biological interpretation of conserved vs divergent calls

## Evidence

- [results] m/z 149.0118 is root-tip specific in B73 but more broadly localized in Oaxacan Green, and m/z 280.2367 is root-tip enriched in B73 yet shows strong specific localization to the EZ cortex in Oaxacan Green.: [evidence span withheld — anchor status: not_found]
- [results] Of the 48 B73 meristem-localized metabolites, 20 are conserved in Oaxacan Green, but about half of these shared signatures do not retain meristem localization: [evidence span withheld — anchor status: not_found]
