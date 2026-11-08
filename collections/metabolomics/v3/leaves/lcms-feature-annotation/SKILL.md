---
name: lcms-feature-annotation
description: Use when a peak table from untargeted LC-MS metabolomics needs putative
  compound-level annotations from m/z values, especially when candidates must be prioritized
  against KEGG context networks rather than by m/z match alone.
when_to_use_negative:
- Annotation requires databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB), since
  mWISE only offers KEGG data
- The input is already an annotated compound table; the skill targets unannotated
  untargeted LC-MS peak lists
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: mWISE
  role: R package implementing the full annotation pipeline (matching, clustering/filtering,
    diffusion prioritization) and the wrapper mWISE.annotation
  repo: https://dev.b2s.club/b2slab/mWISE
- name: Genomic Ranges
  role: Backing the matchingStage function for rapid matching of LC-MS peaks to KEGG
- name: Diffu Stats
  role: Used by set.diffusion to diffuse the label vector in the chosen network
- name: FELLA
  role: Source of the biological/biochemical network used in the top-performing diffusion
    configuration
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:50531c09c2ae54129bc025863d79a6b063dfb91d27c589389fa0244ea88c509f
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# lcms-feature-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate untargeted LC-MS peaks by matching m/z values to the KEGG database, clustering and filtering candidate adducts/fragments, then prioritizing compounds via label diffusion in a biochemical/biological network, as implemented in the mWISE R package.

## When to use

Use when a peak table from untargeted LC-MS metabolomics needs putative compound-level annotations from m/z values, especially when candidates must be prioritized against KEGG context networks rather than by m/z match alone.

## When NOT to use

- Annotation requires databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB), since mWISE only offers KEGG data
- The input is already an annotated compound table; the skill targets unannotated untargeted LC-MS peak lists

## Inputs

- LC-MS peak table (list of features per acquisition mode, e.g. positive/negative) with m/z, retention time, and sample intensities
- KEGG compound database and set of adducts and fragments to consider
- Benchmark reference data frame of known compound annotations (for performanceEvaluation)
- Choice of biochemical/biological network for diffusion (FELLA or RClass)

## Outputs

- Prioritized list of KEGG compound annotations per LC-MS peak (top three candidates per peak)
- Cluster assignment of features likely to come from the same metabolite
- Performance metrics (sensitivity, specificity, F1) against a benchmark reference

## How to apply

First filter the input peak list by removing LC-MS features without signal and applying the 80% rule where intensity is 0. Run matchingStage (built on the GenomicRanges R package) to match all LC-MS peaks to the KEGG database considering a set of adducts and fragments. Then apply featuresClustering, which uses DBSCAN, spectral clustering, and k-means over principal components, optimizing the epsilon parameter and the number of clusters by maximizing the objective f3 - (f2 + f1) computed from intensity similarity and retention-time similarity. Apply clusterBased.filter to reduce false-positive candidates introduced before diffusion, compute the initial diffusion labels vector with diffusion.input, and use set.diffusion (via the DiffuStats package) to diffuse labels in the chosen FELLA or RClass biochemical/biological network. Select the top three candidates per peak as the final prioritized proposal and score the output with performanceEvaluation (sensitivity, specificity, F1) against a benchmark reference; the paper's Trypanosoma negative-mode dataset demonstrated matching sensitivity up to 0.92 and final F1 up to 0.49.

## Related tools

- **mWISE** (R package implementing the full annotation pipeline (matching, clustering/filtering, diffusion prioritization) and the wrapper mWISE.annotation) — https://dev.b2s.club/b2slab/mWISE
- **Genomic Ranges** (Backing the matchingStage function for rapid matching of LC-MS peaks to KEGG)
- **Diffu Stats** (Used by set.diffusion to diffuse the label vector in the chosen network)
- **FELLA** (Source of the biological/biochemical network used in the top-performing diffusion configuration)

## Examples

```
res <- mWISE.annotation(peaks = trypanosoma_negative, adducts = adducts, network = fella_network)
```

## Evaluation signals

- matchingStage sensitivity against known reference compounds (Trypanosoma Assay 1: 0.92, 144 TP / 13 FN)
- Reduction of false positives after clusterBased.filter (e.g. Assay 2 FP from 17598 to 1671, specificity 0.91)
- End-to-end F1 via performanceEvaluation (FELLA raw-score binary diffusion reached 0.49 on Assay 6)
- Mean sensitivity across datasets ≥ 0.63 with SD ≤ 0.07, versus 0.55 (0.19) for xMSannotator
- Proposed compounds' chemical structures closer to the original compounds than competing annotators

## Limitations

- mWISE only offers the data to annotate in KEGG database, an important limitation with respect to xMSannotator
- Requires choosing diffusion configuration (input type, score, network) — performance varies across datasets
- Repository migrated: the GitHub link is no longer active and points to Forgejo (https://dev.b2s.club/b2slab/mWISE)

## Evidence

- [abstract] Three-stage pipeline: matching, clustering/filtering, diffusion prioritization: [evidence span withheld — anchor status: not_found]
- [methods] matchingStage implementation detail: [evidence span withheld — anchor status: not_found]
- [methods] Clustering objective over intensity and retention time: [evidence span withheld — anchor status: not_found]
- [methods] Diffusion scoring via DiffuStats: [evidence span withheld — anchor status: not_found]
