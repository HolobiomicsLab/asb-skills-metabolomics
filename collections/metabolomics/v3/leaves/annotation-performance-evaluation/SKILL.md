---
name: annotation-performance-evaluation
description: Use when an mWISE (or similar LC-MS annotation) pipeline has produced
  prioritized compound lists for peaks and you need quantitative sensitivity/specificity/F1
  against a ground-truth reference compound set.
when_to_use_negative:
- When no ground-truth annotation benchmark (reference compounds) is available for
  the dataset — metrics cannot be computed
- When the goal is only candidate annotation without ranking quality assessment —
  use mWISE annotation workflow itself, not the evaluation wrapper
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: mWISE
  role: R package providing performanceEvaluation to score annotation outputs against
    benchmark reference compounds
  repo: https://dev.b2s.club/b2slab/mWISE
- name: R
  role: Runtime environment in which mWISE's performanceEvaluation and the annotation
    pipeline are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:c0302b6e7f68e571ee2af3de261740a71338d9645c1f3eb100992e2ef5ac583b
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
  - Genomic Ranges
  - R
  - mWISE
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# annotation-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score mWISE annotation results against a benchmark reference using performanceEvaluation, computing sensitivity, specificity, and F1 for the matching stage, cluster-based filter, and diffusion-based prioritization stages.

## When to use

Use when an mWISE (or similar LC-MS annotation) pipeline has produced prioritized compound lists for peaks and you need quantitative sensitivity/specificity/F1 against a ground-truth reference compound set.

## When NOT to use

- When no ground-truth annotation benchmark (reference compounds) is available for the dataset — metrics cannot be computed
- When the goal is only candidate annotation without ranking quality assessment — use mWISE annotation workflow itself, not the evaluation wrapper

## Inputs

- Prioritized annotation list (peak-to-compound mappings, top three candidates per peak)
- Benchmark reference data frame of peaks with known true compounds (e.g. Trypanosoma dataset references)
- Per-stage intermediate outputs: matching results (TP/FN/FP), cluster-based filter results, diffusion-ranked candidates

## Outputs

- Sensitivity, specificity, and F1 per assay and per pipeline stage
- Confusion counts: TP, FN, FP (and TN) per assay/stage
- Comparative performance table (mWISE configurations vs. xMSannotator vs. mummichog)

## How to apply

Assemble a reference data frame of LC-MS peaks whose true compounds are known (the Trypanosoma sample dataset's benchmark compounds). Run mWISE.annotation end-to-end or stepwise (matchingStage, featuresClustering, clusterBased.filter, diffusion.input, set.diffusion) and feed the prioritized list — top three candidates per peak — into mWISE's performanceEvaluation function. The tool computes true positives, false negatives, and false positives per assay and derives sensitivity (TP / (TP + FN)), specificity (TN / (TN + FP)), and F1 measures. Benchmark each stage separately: matchingStage alone (e.g. 0.92 sensitivity in Assay 1), clusterBased.filter (FP reduced e.g. 17598 to 1671, specificity 0.91 in Assay 2), and diffusion prioritization on FELLA or RClass networks (F1 up to 0.49 and 0.39 in Assay 6 respectively). Compare against xMSannotator and mummichog baselines, which achieved F1 up to 0.22 and 0.29 in Assay 6.

## Related tools

- **mWISE** (R package providing performanceEvaluation to score annotation outputs against benchmark reference compounds) — https://dev.b2s.club/b2slab/mWISE
- **R** (Runtime environment in which mWISE's performanceEvaluation and the annotation pipeline are executed)

## Examples

```
R> res <- mWISE.annotation(peaks = peaks, data = KEGG); R> perf <- mWISE::performanceEvaluation(res, reference = benchmark)
```

## Evaluation signals

- Sensitivity in the 0.6–0.9 range expected for matching/filter stages (e.g. matching sensitivity 0.92 for Assay 1)
- Cluster-based filter should reduce false positives markedly (specificity ≥ ~0.9 on Trypanosoma assays)
- F1 for diffusion-prioritized annotations expected ~0.3–0.5, clearly above xMSannotator (≤ 0.22) and mummichog (≤ 0.29) baselines in Assay 6
- Top three candidates per peak selected as the final proposal before scoring
- Mean sensitivity across datasets should approach 0.63 (SD 0.07) versus xMSannotator's 0.55 (SD 0.19)

## Limitations

- Evaluation requires a benchmark reference data frame; cannot be applied to datasets without known compounds
- mWISE only offers KEGG data for annotation, limiting comparative evaluation against tools that also use HMDB, Lipid Maps, or T3DB
- Metrics reflect KEGG-only candidates; performance estimates are not transferable to other databases

## Evidence

- [methods] performanceEvaluation scoring against reference compounds: [evidence span withheld — anchor status: not_found]
- [results] matching sensitivity evidence: "Assay 1 144 11605 0 13 0 157 0.92 0.00 0.01 0.01 0.02"
- [results] cluster-based filter specificity evidence: "This filter allows to reduce the number of false positives introduced in the diffusion process, thus improving the performance of the final prioritization."
- [results] FELLA diffusion F1 evidence: "Assay 6 53 90 7852 21 1 74 0.72 0.99 0.37 0.99 0.49"
- [abstract] baseline comparison evidence: "all four of them outperform xMSannotator (a state-of-the-art annotator) in terms of both performance and computation time"
