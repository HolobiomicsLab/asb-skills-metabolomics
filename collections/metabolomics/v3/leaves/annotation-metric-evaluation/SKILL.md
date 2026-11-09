---
name: annotation-metric-evaluation
description: Use when you have mWISE (or xMSannotator) candidate annotations for LC-MS
  peaks and a reference annotation table, and you need per-assay sensitivity/specificity/precision/F1
  to benchmark a prioritization configuration.
when_to_use_negative:
- The peaks have not yet been passed through mWISE's matching and cluster-based filter
  stages, so no candidate KEGG annotations exist to score.
- The goal is annotation discovery for a new dataset rather than benchmarking an existing
  prioritization against known reference annotations.
- The analysis requires non-KEGG reference databases such as HMDB, Lipid Maps or T3DB,
  which mWISE does not offer data to annotate against.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: mWISE
  role: Generates the candidate KEGG annotations (matching, cluster-based filter,
    diffusion prioritization) whose metrics are computed.
  repo: https://dev.b2s.club/b2slab/mWISE
- name: Diffu Stats
  role: Performs the diffusion used in set.diffusion to produce the prioritized candidate
    list that is then scored against reference annotations.
- name: R
  role: Environment in which the metric computation (TP/FP/FN/TN counts and derived
    metrics) is carried out.
- name: xMSannotator
  role: Competing annotator benchmarked with the same metric framework (F1 0.15-0.22
    vs mWISE).
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:23ac8e173da1a1eb8c413fd138929ae26b81b1c4d77147c98523e2429ec66289
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
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# annotation-metric-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-assay annotation quality metrics (TP, FP, FN, TN, sensitivity, specificity, precision, accuracy, F1) for mWISE's diffusion-based prioritization of KEGG compounds, enabling direct comparison of configurations (e.g. RClass + z-normalized score) and competing annotators such as xMSannotator.

## When to use

Use when you have mWISE (or xMSannotator) candidate annotations for LC-MS peaks and a reference annotation table, and you need per-assay sensitivity/specificity/precision/F1 to benchmark a prioritization configuration.

## When NOT to use

- The peaks have not yet been passed through mWISE's matching and cluster-based filter stages, so no candidate KEGG annotations exist to score.
- The goal is annotation discovery for a new dataset rather than benchmarking an existing prioritization against known reference annotations.
- The analysis requires non-KEGG reference databases such as HMDB, Lipid Maps or T3DB, which mWISE does not offer data to annotate against.

## Inputs

- Filtered peak lists for each benchmark assay (LC-MS features after 80% rule / zero-signal removal)
- Reference annotation tables for each assay
- Candidate KEGG compounds and reduced feature set per assay from mWISE matching and cluster-based filter
- mWISE diffusion-based prioritized proposal (top three candidates per peak)

## Outputs

- Per-assay confusion matrix counts (TP, FP, FN, TN)
- Per-assay metric table: sensitivity, specificity, precision, accuracy, F1 (e.g. rclass_znorm_metrics.csv)

## How to apply

Run mWISE's three stages — matching (GenomicRanges-based matching Stage), cluster-based filter, and diffusion with DiffuStats (diffuse() on a FELLA or RClass network with the probability input type and z-normalized score, or binary input with raw score) — selecting the top three candidates per peak as the final prioritized proposal. Compare these proposals against the reference annotations to count TP, FP, FN and TN per assay. Compute sensitivity, specificity, precision, accuracy and F1 per assay and export the metric table (e.g. rclass_znorm_metrics.csv). The matching stage alone can be scored separately (e.g. Assay 1: 144 TP, 13 FN, sensitivity 0.92), and the cluster-based filter can be scored before and after (Assay 2 FP drops from 17598 to 1671) to quantify its contribution to the final prioritization.

## Related tools

- **mWISE** (Generates the candidate KEGG annotations (matching, cluster-based filter, diffusion prioritization) whose metrics are computed.) — https://dev.b2s.club/b2slab/mWISE
- **Diffu Stats** (Performs the diffusion used in set.diffusion to produce the prioritized candidate list that is then scored against reference annotations.)
- **R** (Environment in which the metric computation (TP/FP/FN/TN counts and derived metrics) is carried out.)
- **xMSannotator** (Competing annotator benchmarked with the same metric framework (F1 0.15-0.22 vs mWISE).)

## Examples

```
Rscript -e 'mWISE::matchingStage(peaks, kegg, adducts); mWISE::featuresClustering(); mWISE::clusterBased.filter(); mWISE::diffusion.input(); mWISE::set.diffusion(network="rclass", input="probability", score="znorm")'
```

## Evaluation signals

- Assay 1 matching-stage sensitivity should equal 0.92 (144 TP, 13 FN, 157 references).
- Cluster-based filter on Assay 2 should reduce false positives from 17598 to 1671, giving specificity 0.91.
- For the RClass network with z-normalized score and probability input, Assay 6 sensitivity should be ~0.62 and F1 ~0.39 (TP 46, precision 0.29).
- The FELLA network with raw score and binary input should reach F1 0.49 for Assay 6 (53 TP, 21 FN).
- Metric values must fall within the reported ranges across all six assays (mWISE F1 0.15-0.49; xMSannotator F1 0.15-0.22).

## Limitations

- mWISE only annotates against KEGG, an important limitation relative to xMSannotator, which supports KEGG, HMDB, Lipid Maps and T3DB.
- Evaluation depends on the availability of complete reference annotation tables; assays with high false-positive counts (e.g. 17598 before filtering) make specificity sensitive to filter quality.
- Only the top three candidates per peak are selected as the final proposal, so peaks with fewer candidates or metric sensitivity to ranking depth are inherently constrained.

## Evidence

- [results] mWISE matching sensitivity for Assay 1 was 0.92 (144 TP, 13 FN, 157 references): "Assay 1 144 11605 0 13 0 157 0.92 0.00 0.01 0.01 0.02"
- [results] The top three candidates for each peak are selected as the final prioritized proposal: "The top three candidates for each peak, if available, are selected as the final prioritized proposal."
- [results] Cluster-based filter reduces false positives before diffusion: "This filter allows to reduce the number of false positives introduced in the diffusion process, thus improving the performance of the final prioritization."
- [results] mWISE is limited to KEGG annotation: "mWISE only offers the data to annotate in KEGG database"
- [results] Diffusion-based scores are computed with probability input type and z-normalized score: "The diffusion-based scores are computed using the probability input type and the z normal- ized score and a ranked list is built."
