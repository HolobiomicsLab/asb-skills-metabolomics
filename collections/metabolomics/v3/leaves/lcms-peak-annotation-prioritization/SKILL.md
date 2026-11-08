---
name: lcms-peak-annotation-prioritization
description: Use when you have a filtered LC-MS peak table (signal-bearing features,
  80% rule applied) and need a biologically prioritized KEGG compound annotation list
  that outperforms mass-only matching, e.g. reproducing mWISE vs xMSannotator sensitivity
  benchmarks.
when_to_use_negative:
- You need annotation against non-KEGG databases (e.g. HMDB, Lipid Maps, T3DB) since
  mWISE only offers KEGG data, unlike xMSannotator.
- The input is raw instrument spectra or unfiltered peak lists; features without signal
  and the 80% rule must be applied first.
- Computation time or exact standalone database coverage matters more than a small
  set of biologically prioritized KEGG candidates.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mWISE
  role: R package orchestrating the matching, clustering, filtering and diffusion
    prioritization pipeline
  repo: https://dev.b2s.club/b2slab/mWISE
- name: R
  role: Runtime environment in which the mWISE pipeline and evaluation scripts are
    executed
- name: xMSannotator
  role: State-of-the-art annotator used as the benchmark baseline against mWISE sensitivity
- name: FELLA
  role: Source of the biological network used in the diffusion configuration achieving
    mean sensitivity 0.63 (0.07)
- name: Genomic Ranges
  role: Used by matchingStage to rapidly match all LC-MS peaks to KEGG under the adduct/fragment
    set
- name: Diffu Stats
  role: Diffuses the label vector in a given network inside set.diffusion to produce
    diffusion scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:5d921b8473378effb199d12dd890b55dd8ed3e9c349abf7f0a38fd8cb7f51ab8
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lcms-peak-annotation-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate LC-MS peaks to KEGG compounds via mWISE by matching m/z against a default adduct/fragment set, clustering candidate features with spectral clustering (DBSCAN-optimized) plus a cluster-based filter, and prioritizing a final top-three list per peak using diffusion scores in the FELLA or RClass networks.

## When to use

Use when you have a filtered LC-MS peak table (signal-bearing features, 80% rule applied) and need a biologically prioritized KEGG compound annotation list that outperforms mass-only matching, e.g. reproducing mWISE vs xMSannotator sensitivity benchmarks.

## When NOT to use

- You need annotation against non-KEGG databases (e.g. HMDB, Lipid Maps, T3DB) since mWISE only offers KEGG data, unlike xMSannotator.
- The input is raw instrument spectra or unfiltered peak lists; features without signal and the 80% rule must be applied first.
- Computation time or exact standalone database coverage matters more than a small set of biologically prioritized KEGG candidates.

## Inputs

- Filtered LC-MS peak table (features without signal removed, 80% rule applied for zero intensities)
- KEGG database compounds
- Set of adducts and fragments for m/z matching
- FELLA or RClass graph for diffusion
- Known compound reference list (for sensitivity evaluation)

## Outputs

- Prioritized list of KEGG compound candidates per LC-MS peak (top three per peak)
- Per-dataset sensitivity/specificity/F1 metrics for the annotation
- Comparison table of mean (SD) sensitivity across mWISE configurations and xMSannotator

## How to apply

First filter the input peak list in R by removing LC-MS features without signal and applying the 80% rule (drop peaks with zero intensity in more than 20% of samples). Run the matchingStage command with GenomicRanges to match all peaks to KEGG under the default set of adducts and fragments, then apply featuresClustering (spectral clustering, optimizing the number of principal components and the epsilon parameter of DBSCAN) followed by clusterBased.filter to cut false positives before diffusion. Next call diffusion.input to build the label vector and set.diffusion with DiffuStats to diffuse labels over either the FELLA or RClass graph, yielding four mWISE configurations: FELLA and RClass with probability input type and z-normalized scores, plus binary input with raw scores and the unique annotation option. Finally select the top three candidates per peak as the prioritized annotation proposal and benchmark sensitivity against known compound references, expecting roughly 0.63 mean sensitivity for the FELLA raw-score configuration versus 0.55 for xMSannotator.

## Related tools

- **mWISE** (R package orchestrating the matching, clustering, filtering and diffusion prioritization pipeline) — https://dev.b2s.club/b2slab/mWISE
- **R** (Runtime environment in which the mWISE pipeline and evaluation scripts are executed)
- **xMSannotator** (State-of-the-art annotator used as the benchmark baseline against mWISE sensitivity)
- **FELLA** (Source of the biological network used in the diffusion configuration achieving mean sensitivity 0.63 (0.07))
- **Genomic Ranges** (Used by matchingStage to rapidly match all LC-MS peaks to KEGG under the adduct/fragment set)
- **Diffu Stats** (Diffuses the label vector in a given network inside set.diffusion to produce diffusion scores)

## Examples

```
Rscript -e 'mWISE::matchingStage(peaks, kegg, adducts=TRUE); mWISE::featuresClustering(...); mWISE::clusterBased.filter(...); mWISE::diffusion.input(...); mWISE::set.diffusion(...)'
```

## Evaluation signals

- Mean (SD) sensitivity across datasets for the FELLA/raw-scores configuration should approximate 0.63 (0.07) and 0.55 (0.19) for xMSannotator.
- Per-assay matching sensitivity values should match reported figures, e.g. Assay 1: 144 TP, 13 FN, 157 references, sensitivity 0.92.
- Cluster-based filter must reduce false positives before diffusion, e.g. Assay 2: FP from 17598 to 1671 with specificity 0.91.
- FELLA and RClass diffusion F1 should not exceed the reported maxima (FELLA up to 0.49, RClass up to 0.39) on the corresponding assays.
- Exactly three candidate KEGG compounds per peak (where available) appear in the final prioritized proposal.

## Limitations

- mWISE only offers annotation data from the KEGG database, an important limitation with respect to xMSannotator which supports HMDB, Lipid Maps and T3DB.
- Diffusion-based prioritization depends on the FELLA or RClass graph choice and on input type (probability with z-normalized scores vs binary with raw scores) plus the unique annotation option.
- Performance is reported for the top three candidates per peak; prioritization depth beyond that is not characterized in this article.

## Evidence

- [methods] Matching stage with GenomicRanges to KEGG under default adduct/fragment set: [evidence span withheld — anchor status: not_found]
- [methods] Spectral clustering with DBSCAN optimization groups co-eluting features: [evidence span withheld — anchor status: not_found]
- [methods] DiffuStats performs label diffusion in set.diffusion: [evidence span withheld — anchor status: not_found]
