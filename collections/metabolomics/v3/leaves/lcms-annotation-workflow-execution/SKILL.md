---
name: lcms-annotation-workflow-execution
description: Use when you have a filtered peak list from an LC-MS assay with m/z,
  RT and adduct/fragment information, and need a KEGG-based, context-prioritized compound
  annotation per peak rather than a simple mass matching.
when_to_use_negative:
- When annotation targets databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB)
  — mWISE only offers KEGG, a limitation relative to xMSannotator
- When the input is already an annotated compound table or spectral library match
  — this skill starts from raw peak-to-KEGG matching, not from pre-annotated features
- When the peak list has not been pre-filtered (features without signal removed, 80%
  rule applied where intensity equals 0)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3346
tools:
- name: mWISE
  role: Modular R package that orchestrates the full annotation workflow (matching,
    clustering, filtering, diffusion) to produce a prioritized KEGG candidate list
    per LC-MS peak
  repo: https://github.com/b2slab/mWISE
- name: Genomic Ranges
  role: Used inside the matching Stage to rapidly match all LC-MS peaks to KEGG considering
    adducts and fragments
- name: Diffu Stats
  role: Used by set.diffusion to diffuse the label vector in the chosen network (FELLA
    or RClass)
- name: FELLA
  role: Provides the biological network used in the best-performing diffusion configuration
    (FELLA network plus raw scores)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:42e5df45a421ad6590cbc85e24ae22056d7e49f112f5e47244ea87f50f803722
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  repo_url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3346
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# lcms-annotation-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end execution of the mWISE LC-MS annotation workflow: matching LC-MS peaks to KEGG, clustering and filtering candidates, and diffusing labels in a FELLA or RClass network to produce a prioritized per-peak candidate list.

## When to use

Use when you have a filtered peak list from an LC-MS assay with m/z, RT and adduct/fragment information, and need a KEGG-based, context-prioritized compound annotation per peak rather than a simple mass matching.

## When NOT to use

- When annotation targets databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) — mWISE only offers KEGG, a limitation relative to xMSannotator
- When the input is already an annotated compound table or spectral library match — this skill starts from raw peak-to-KEGG matching, not from pre-annotated features
- When the peak list has not been pre-filtered (features without signal removed, 80% rule applied where intensity equals 0)

## Inputs

- Filtered LC-MS peak list (m/z, RT, intensity) per assay, with features without signal removed and the 80% rule applied where intensity equals 0
- Acquisition mode objects organized as lists (e.g. positive and negative mode lists as in the Trypanosoma dataset)
- Adducts and fragments set for matching LC-MS peaks to KEGG
- Chosen network for diffusion (FELLA or RClass)

## Outputs

- Prioritized list of KEGG compounds per LC-MS peak (top three candidates per peak, when available)
- Diffusion scores per compound candidate (e.g. raw score, z-normalized score)
- Filtered candidate set after cluster-based filtering

## How to apply

Run the mWISE pipeline in R: first use the matching Stage with Genomic Ranges to match all LC-MS peaks to KEGG considering a set of adducts and fragments; then apply features Clustering (spectral clustering, DBSCAN, or k-means with varying k) to group features likely to come from the same metabolite, and apply the cluster Based.filter to reduce false positives introduced in the diffusion process; finally compute the diffusion labels vector with diffusion.input and diffuse it via set.diffusion using Diffu Stats in a chosen network (FELLA or RClass) with score options such as raw score and unique annotation. Select the top three candidates per peak as the final prioritized proposal; the FELLA network with raw scores is the configuration evaluated as best performing (e.g. F1 up to 0.49 on Assay 6 with binary input).

## Related tools

- **mWISE** (Modular R package that orchestrates the full annotation workflow (matching, clustering, filtering, diffusion) to produce a prioritized KEGG candidate list per LC-MS peak) — https://github.com/b2slab/mWISE
- **Genomic Ranges** (Used inside the matching Stage to rapidly match all LC-MS peaks to KEGG considering adducts and fragments)
- **Diffu Stats** (Used by set.diffusion to diffuse the label vector in the chosen network (FELLA or RClass))
- **FELLA** (Provides the biological network used in the best-performing diffusion configuration (FELLA network plus raw scores))

## Examples

```
Rscript -e 'library(mWISE); res <- set.diffusion(matching(clusterBased.filter(featuresClustering(peaks, "DBSCAN"), "k-means"), adducts), network="FELLA", scores="raw")'
```

## Evaluation signals

- The top three candidates per peak, if available, are selected as the final prioritized proposal — verify output contains exactly up to three ranked candidates per peak
- Cluster-based filter should visibly reduce false positives before diffusion (e.g. on Assay 2, FP drops from 17598 to 1671, specificity 0.91) — compare candidate counts before and after the filter
- Benchmark sensitivity across the six assays should approximate the reported mWISE value (e.g. sensitivity mean 0.63, SD 0.07, for the FELLA network plus raw scores configuration) rather than the xMSannotator 0.55 (SD 0.19)
- Runtime sanity check: mWISE per-assay wall-clock time should stay on the order of tens of minutes (e.g. FELLA-raw 15.70-53.38 min) rather than xMSannotator's 106.34-318.04 min
- Per-assay F1 from diffusion prioritization should be within the reported ranges (FELLA up to 0.49; RClass up to 0.39) rather than xMSannotator's 0.15-0.22

## Limitations

- mWISE only offers the data to annotate in KEGG database — an important limitation with respect to xMSannotator that should be addressed in future versions of mWISE
- Diffusion prioritization depends on the chosen network and score options; results vary (e.g. FELLA F1 0.49 vs RClass 0.39 on Assay 6) and require configuration choice (probability input with z-normalized score vs binary input with raw score and unique annotation)
- Cluster-based filter and diffusion can introduce false positives if the cluster-based filtering step is not applied before diffusion

## Evidence

- [methods] matching Stage with GenomicRanges: [evidence span withheld — anchor status: not_found]
- [methods] features Clustering: [evidence span withheld — anchor status: not_found]
- [methods] cluster Based.filter: [evidence span withheld — anchor status: not_found]
- [methods] set.diffusion with Diffu Stats: [evidence span withheld — anchor status: not_found]
