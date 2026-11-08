---
name: graph-diffusion-prioritization
description: Decision support for ranking matched KEGG compound candidates per LC-MS
  peak when a filtered candidate set exists and network context (KEGG, FELLA or RClass)
  can break ties among chemically plausible matches.
when_to_use_negative:
- Input is a raw LC-MS peak table without prior KEGG matching and cluster-based filtering
  — mWISE requires matchingStage and clusterBased.filter outputs before diffusion
- Annotation targets databases other than KEGG (HMDB, Lipid Maps, T3DB) — mWISE lacks
  support for them, unlike xMSannotator
edam_operation: http://edamontology.org/operation_3907
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0154
tools:
- name: mWISE
  role: R package implementing the diffusion prioritization stage (diffusion.input
    and set.diffusion functions)
  repo: https://github.com/b2slab/mWISE
- name: DiffuStats
  role: R package used internally by set.diffusion to diffuse the label vector over
    the chosen network
- name: FELLA
  role: Provides the biological network over which labels are diffused for one mWISE
    configuration
- name: Genomic Ranges
  role: Used by the upstream matchingStage, not by diffusion itself; included for
    pipeline context
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:a586d97e66058c6f7a4d5148b76fcbef53096c9af084143545de6bdfc87b8a9c
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
  - DiffuStats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  edam_operation: http://edamontology.org/operation_3907
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# graph-diffusion-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prioritize KEGG compound candidates for LC-MS peaks by diffusing matched annotation labels through a biochemical or biological network (FELLA or RClass), as implemented in the mWISE R package via diffusion.input and set.diffusion.

## When to use

Decision support for ranking matched KEGG compound candidates per LC-MS peak when a filtered candidate set exists and network context (KEGG, FELLA or RClass) can break ties among chemically plausible matches.

## When NOT to use

- Input is a raw LC-MS peak table without prior KEGG matching and cluster-based filtering — mWISE requires matchingStage and clusterBased.filter outputs before diffusion
- Annotation targets databases other than KEGG (HMDB, Lipid Maps, T3DB) — mWISE lacks support for them, unlike xMSannotator

## Inputs

- Filtered peak-to-KEGG candidate list (output of mWISE matchingStage and clusterBased.filter) for an untargeted LC-MS feature table
- Biochemical/biological network built from FELLA or RClass (KEGG-derived)
- Benchmark reference data frame of known compounds (optional, for performanceEvaluation)

## Outputs

- Per-peak ranked candidate list, with the top three KEGG compounds (if available) as the final prioritized annotation
- Diffusion scores (probability/z-normalized or binary/raw) per network node
- Performance metrics (sensitivity, specificity, F1) when benchmarked against reference compounds

## How to apply

After matching and cluster-based filtering, first call diffusion.input to compute the initial diffusion labels vector from the filtered peak-to-KEGGC candidates. Then call set.diffusion, which wraps the DiffuStats R package, to diffuse the label vector over a chosen network — the FELLA network or the RClass network — and score nodes. Diffusion scores are computed with the 'probability' input type and the z-normalized score, or alternatively with 'binary' input, raw score and the unique annotation option; ranked lists are then built from these scores. Select the top three candidates per peak, if available, as the final prioritized proposal. This graph diffusion step is the component that most improves annotation performance, and the FELLA-network + raw-scores configuration achieves a mean sensitivity of 0.63 (SD 0.07) across the Trypanosoma datasets versus 0.55 (0.19) for xMSannotator.

## Related tools

- **mWISE** (R package implementing the diffusion prioritization stage (diffusion.input and set.diffusion functions)) — https://github.com/b2slab/mWISE
- **DiffuStats** (R package used internally by set.diffusion to diffuse the label vector over the chosen network)
- **FELLA** (Provides the biological network over which labels are diffused for one mWISE configuration)
- **Genomic Ranges** (Used by the upstream matchingStage, not by diffusion itself; included for pipeline context)

## Examples

```
library(mWISE); res <- set.diffusion(input_labels, network = 'FELLA', inputType = 'probability', score = 'z'); top3 <- finalResults(res)
```

## Evaluation signals

- Benchmark with mWISE performanceEvaluation: F1 of ~0.49 for the FELLA network on Assay 6 (raw score, binary input) is the upper bound reported in the paper
- Sensitivity across the Trypanosoma datasets should be ~0.63 (SD 0.07) for the FELLA + raw-scores configuration
- Specificity after diffusion is near 0.99 on Assays 1–6 (e.g. 0.99 for Assay 6 FELLA configuration)
- Top three candidates per peak (where available) constitute the final output — verify the ranked list length per peak is ≤ 3
- Compare against xMSannotator (expected F1 0.15–0.22) and mummichog (0.08–0.29) as baseline sanity checks

## Limitations

- mWISE lacks support for annotation databases other than KEGG (HMDB, Lipid Maps, T3DB) — an acknowledged limitation relative to xMSannotator
- Diffusion quality depends on network choice (FELLA vs RClass): FELLA reached F1 0.49 while RClass reached 0.39 on Assay 6
- The GitHub repository b2slab/mWISE is no longer active; the maintained fork is hosted on Forgejo at https://dev.b2s.club/b2slab/mWISE

## Evidence

- [methods] The set.diffusion function uses Diffu Stats R package to diffuse the label vector in a given network: [evidence span withheld — anchor status: not_found]
