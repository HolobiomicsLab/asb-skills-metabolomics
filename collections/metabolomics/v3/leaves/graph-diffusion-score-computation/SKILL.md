---
name: graph-diffusion-score-computation
description: Use when after matching LC-MS peaks to KEGG and filtering candidates,
  you need a network-context score that propagates evidence from candidate compounds
  across a graph to prioritize the top three KEGG compounds per peak.
when_to_use_negative:
- No network context available — if you only need compound identity matching against
  KEGG/HMDB/LipidMaps/T3DB without prioritization, xMSannotator-style multi-database
  annotation is more appropriate.
- Peaks have not yet been matched to KEGG or filtered by cluster — diffusion labels
  cannot be computed meaningfully without the candidate set from the matching and
  filtering stages.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: mWISE
  role: 'R package implementing the diffusion scoring stage: diffusion.input builds
    the initial labels vector and set.diffusion runs the diffusion over the chosen
    network'
  repo: https://dev.b2s.club/b2slab/mWISE
- name: Diffu Stats
  role: R package invoked by set.diffusion to actually diffuse the label vector in
    a given network and produce diffusion scores
- name: R
  role: Runtime environment in which mWISE and DiffuStats are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:a91a248f8c63d16d69cef79afec60268bbf9b60d2afc07a6acb8a1e27cde6268
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - R
  - mWISE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# graph-diffusion-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Diffuses label vectors derived from LC-MS peak annotations over a biological or biochemical network (e.g., FELLA or RClass) using DiffuStats to produce a diffusion-score-ranked prioritized list of KEGG compounds per peak, the final scoring stage of mWISE.

## When to use

Use when, after matching LC-MS peaks to KEGG and filtering candidates, you need a network-context score that propagates evidence from candidate compounds across a graph to prioritize the top three KEGG compounds per peak.

## When NOT to use

- No network context available — if you only need compound identity matching against KEGG/HMDB/LipidMaps/T3DB without prioritization, xMSannotator-style multi-database annotation is more appropriate.
- Peaks have not yet been matched to KEGG or filtered by cluster — diffusion labels cannot be computed meaningfully without the candidate set from the matching and filtering stages.

## Inputs

- Filtered LC-MS peak list (features without signal removed, 80% rule applied where intensity equals 0)
- Candidate KEGG compounds per peak after matching Stage and cluster-based filter
- Initial diffusion labels vector (probability or binary input type)
- Graph/biochemical network (FELLA or RClass)

## Outputs

- Diffusion score-ranked list of KEGG compounds per peak
- Final prioritized proposal: top three candidates per peak
- Per-assay sensitivity, specificity, precision, accuracy and F1 metrics table (e.g., rclass_znorm_metrics.csv)

## How to apply

First run mWISE's matching Stage (GenomicRanges-based matching against KEGG with a set of adducts and fragments) and the cluster-based filter to obtain the reduced candidate set per peak. Then compute the initial labels vector with diffusion.input, using either probability input with z-normalized score or binary input with raw score and unique annotation. Call set.diffusion, which invokes the DiffuStats R package to diffuse this label vector over a chosen network — the FELLA biological network or the RClass biochemical network — and rank compounds by the diffusion score. Select the top three candidates per peak (if available) as the final prioritized proposal, since diffusion prioritization is what gives mWISE its edge over xMSannotator in both sensitivity and computation time.

## Related tools

- **mWISE** (R package implementing the diffusion scoring stage: diffusion.input builds the initial labels vector and set.diffusion runs the diffusion over the chosen network) — https://dev.b2s.club/b2slab/mWISE
- **Diffu Stats** (R package invoked by set.diffusion to actually diffuse the label vector in a given network and produce diffusion scores)
- **R** (Runtime environment in which mWISE and DiffuStats are executed)

## Examples

```
Rscript -e "library(mWISE); diff <- set.diffusion(graph=fella_net, input=diffusion.input(peak_list, k_to_kegg, type='probability'), score='z')"
```

## Evaluation signals

- For the FELLA network with raw score and binary input, Assay 6 should yield TP 53, sensitivity 0.72 and F1 0.49.
- For the RClass network with z-normalized score and probability input, Assay 6 sensitivity should be around 0.59–0.62 and F1 around 0.38–0.39.
- The cluster-based filter must reduce false positives before diffusion (e.g., Assay 2 FP from 17598 to 1671 with specificity 0.91); if FP counts are not reduced, the upstream filter stage was skipped or misconfigured.
- Per-peak proposal must contain at most three KEGG candidates; verify against the reference annotations that TP/FP/FN counts reconcile with reported sensitivity and F1 values.
- Cross-check that the chosen diffusion input type (probability + z-normalized score, or binary + raw score with unique annotation) matches the network configuration used for benchmarking.

## Limitations

- mWISE only offers annotation against the KEGG database; it lacks support for other databases such as HMDB, Lipid Maps or T3DB, which xMSannotator covers.
- mWISE matching itself can introduce many false positives (e.g., 11605 FPs for Assay 1 before filtering), so diffusion quality depends heavily on the upstream cluster-based filter.

## Evidence

- [methods] set.diffusion uses Diffu Stats to diffuse label vector: [evidence span withheld — anchor status: not_found]
- [methods] matching Stage uses Genomic Ranges to match peaks to KEGG: [evidence span withheld — anchor status: not_found]
