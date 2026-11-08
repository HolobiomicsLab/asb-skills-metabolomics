---
name: graph-diffusion-scoring
description: Use when annotating LC-MS peak lists that have already been mass-matched
  to KEGG (with the resulting false positives reduced by cluster-based filtering),
  and a network-context prioritized candidate ranking is needed rather than a plain
  mass-delta list.
when_to_use_negative:
- Input is a raw LC-MS peak table that has not yet been mass-matched to KEGG or cluster-filtered
  — the diffusion step presumes this prior filtering.
- Annotation against databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) is required
  — mWISE diffusion only offers KEGG data.
- A simple mass-to-charge candidate list is sufficient and network-context prioritization
  is unnecessary overhead.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0153
tools:
- name: mWISE
  role: R package providing diffusion.input and set.diffusion to propagate labels
    through networks and produce the prioritized candidate list
  repo: https://dev.b2s.club/b2slab/mWISE
- name: Diffu Stats
  role: Underlying diffusion engine invoked by set.diffusion to diffuse the label
    vector over a given network
- name: FELLA
  role: Provides the biological network used as one of the two diffusion graph options
- name: R
  role: Runtime environment in which mWISE, DiffuStats, and FELLA are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:1eaade6cf0337fc1c84d43cc97a6e125e775af4e8d23a42474be8243e7151ce0
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
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# graph-diffusion-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A graph-diffusion prioritization step from the mWISE LC-MS annotation pipeline: initial per-feature labels are propagated through KEGG-derived biological (FELLA) or biochemical (RClass) networks with DiffuStats to convert sparse mass-match candidates into a scored, network-context-aware compound ranking.

## When to use

Use when annotating LC-MS peak lists that have already been mass-matched to KEGG (with the resulting false positives reduced by cluster-based filtering), and a network-context prioritized candidate ranking is needed rather than a plain mass-delta list.

## When NOT to use

- Input is a raw LC-MS peak table that has not yet been mass-matched to KEGG or cluster-filtered — the diffusion step presumes this prior filtering.
- Annotation against databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) is required — mWISE diffusion only offers KEGG data.
- A simple mass-to-charge candidate list is sufficient and network-context prioritization is unnecessary overhead.

## Inputs

- Filtered LC-MS peak list (peaks with signal, 80%-rule-filtered) already mass-matched to KEGG and cluster-based filtered
- KEGG-derived graph: FELLA network or RClass biochemical network
- Per-feature label vector from diffusion.input

## Outputs

- Diffusion score for each candidate compound per feature
- Ranked list of KEGG candidates per LC-MS feature
- Top-3 prioritized mWISE annotation proposal per peak

## How to apply

After matchingStage has matched peaks to KEGG and clusterBased.filter has trimmed the candidate set, call diffusion.input to build the initial label vector for each feature, then set.diffusion to diffuse it over a chosen network with the DiffuStats package. Choose between the FELLA network (biological) or RClass graph (biochemical); scores can be input as binary labels with raw scores (optionally keeping only the unique closest annotation) or as probability-type input with z-normalized scores. Then merge the four resulting configurations (FELLA and RClass, each in binary/raw-score and probability/z-score variants) with the matched and filtered candidates, and select the top three candidates per peak as the final prioritized proposal.

## Related tools

- **mWISE** (R package providing diffusion.input and set.diffusion to propagate labels through networks and produce the prioritized candidate list) — https://dev.b2s.club/b2slab/mWISE
- **Diffu Stats** (Underlying diffusion engine invoked by set.diffusion to diffuse the label vector over a given network)
- **FELLA** (Provides the biological network used as one of the two diffusion graph options)
- **R** (Runtime environment in which mWISE, DiffuStats, and FELLA are executed)

## Examples

```
R> mWISE::set.diffusion(labels, network = "FELLA", input = "binary", scores = "raw")
```

## Evaluation signals

- Per-assay benchmark on reference-annotated assays: mWISE diffusion prioritization reached F1 up to 0.49 (Assay 6, raw score, binary input) with FELLA, and 0.38 with RClass — expect the same order of magnitude on similar assays.
- Cluster-based filter upstream of diffusion should substantially cut false positives (e.g. Assay 2: FP from 17598 to 1671, specificity 0.91) before scores are propagated.
- Aggregated benchmark against xMSannotator on the six public LC-MS assays should show higher mean sensitivity (0.63 SD 0.07 vs 0.55 SD 0.19) for mWISE diffusion-scoring configurations.
- Each peak should end with at most three prioritized candidates (top-3 proposal), consistent with the article's stated final selection.
- Computation time should be comparable to or better than xMSannotator on the same peak lists, as reported in the article.

## Limitations

- Diffusion is limited to KEGG — unlike xMSannotator, mWISE cannot prioritize against HMDB, Lipid Maps, or T3DB.
- Prioritization is only as good as the upstream mass-matching and cluster-based filtering; diffusion amplifies errors present in those steps if not filtered.
- The choice of network (FELLA vs RClass) and score type (binary/raw vs probability/z-normalized) materially changes results and must be chosen per assay.

## Evidence

- [methods] the function set.diffusion uses Diffu Stats R package to diffuse the label vector in a given network: [evidence span withheld — anchor status: not_found]
