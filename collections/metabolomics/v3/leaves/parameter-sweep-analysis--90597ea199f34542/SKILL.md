---
name: parameter-sweep-analysis
description: Use when you have an aligned post-XCMS feature intensity table and need
  to justify a choice of RAMClustR's sigma_t (retention-time sigma) and sigma_r (correlational
  sigma) values by evaluating cluster quality across a grid of combinations instead
  of accepting defaults blindly.
when_to_use_negative:
- You only need to cluster a single feature set at fixed parameters — use the standard
  RAMClustR ramclustR() call with chosen st/maxt values rather than sweeping a grid.
- The input is not XCMS-aligned feature data (e.g. raw spectra or already-clustered
  spectra) — the sweep is defined over post-XCMS feature tables and RAMClustR's similarity
  scores.
- Your clustering tool does not expose a tunable sigma pair, so there is no meaningful
  two-dimensional parameter space to sweep.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RAMClustR
  role: Computes retention-time and correlational similarity scores and performs the
    hierarchical feature clustering whose quality is scored at each (sigma_t, sigma_r)
    combination
  repo: https://github.com/cbroeckl/RAMClustR
- name: XCMS
  role: Produces the aligned feature table and peak widths that the sweep's sigma_t
    rule (half the median XCMS peak width) is derived from
- name: dynamicTreeCut
  role: Cuts the hierarchical clustering dendrogram at each parameter combination
    so multi-feature cluster counts can be tabulated
- name: R
  role: Implementation environment for the sweep; RAMClustR is released as a freely
    available R package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:3c9f0e301b48162cce2b8a4de832e10c40a61a6f5a69ba6992097dd0c9b3f3e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - R
  - RAMClustR
  - XCMS
  - dynamicTreeCut
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# parameter-sweep-analysis

## Summary

Systematically sweep RAMClustR's two similarity-score parameters — the retention-time sigma (sigma_t) and correlational sigma (sigma_r) — across a grid, quantifying dot-product spectral similarity and the number of multi-feature clusters at each combination to locate an optimal, stable region of the parameter space rather than a sharp peak.

## When to use

Use when you have an aligned post-XCMS feature intensity table and need to justify a choice of RAMClustR's sigma_t (retention-time sigma) and sigma_r (correlational sigma) values by evaluating cluster quality across a grid of combinations instead of accepting defaults blindly.

## When NOT to use

- You only need to cluster a single feature set at fixed parameters — use the standard RAMClustR ramclustR() call with chosen st/maxt values rather than sweeping a grid.
- The input is not XCMS-aligned feature data (e.g. raw spectra or already-clustered spectra) — the sweep is defined over post-XCMS feature tables and RAMClustR's similarity scores.
- Your clustering tool does not expose a tunable sigma pair, so there is no meaningful two-dimensional parameter space to sweep.

## Inputs

- Aligned post-XCMS feature intensity table (features detected on MS and/or idMS/MS data, e.g. faahKO CDF-derived data)
- Parameter grid specification over sigma_t (retention-time sigma) and sigma_r (correlational sigma)
- Optional: median XCMS peak width to anchor sigma_t (optimal sigma_t = half the median peak width)

## Outputs

- Per-combination dot-product spectral similarity of clustered features
- Per-combination count of multi-feature clusters (clusters with at least two features)
- Similarity-score surface over the (sigma_t, sigma_r) grid showing the plateau/stable region
- Optimal (sigma_t, sigma_r) pair and corresponding cluster count (e.g. sigma_t=2, sigma_r=0.5, ~2500 multi-feature clusters)
- Serialized sweep results (e.g. sigma_sweep_results.json)

## How to apply

Load the aligned post-XCMS feature table (e.g. from the faahKO NetCDF/CDF data processed by XCMS) and define a grid of sigma_t and sigma_r combinations — the paper's sweep used 441 combinations (a 21x21 grid), with sigma_t grounded in the rule that the optimal sigma_t equals half the median XCMS peak width. For each (sigma_t, sigma_r) pair, compute the retention-time and correlational similarity scores for every feature pair, form the total similarity score as the product of the two scores, submit the resulting score matrix to hierarchical clustering, and cut the dendrogram with dynamicTreeCut. At each combination, quantify cluster quality as the dot-product spectral similarity of the resulting clusters and record the number of multi-feature clusters (clusters containing at least two features). Inspect the sweep surface for a plateau of high similarity and a broad stable region rather than a sharp optimum; in the faahKO example the optimum landed at sigma_t = 2 and sigma_r = 0.5, yielding approximately 2500 clusters with at least two features. Write the grid extent, the optimal pair, and the approximate cluster count to a results file (e.g. sigma_sweep_results.json) so the sweep is reproducible.

## Related tools

- **RAMClustR** (Computes retention-time and correlational similarity scores and performs the hierarchical feature clustering whose quality is scored at each (sigma_t, sigma_r) combination) — https://github.com/cbroeckl/RAMClustR
- **XCMS** (Produces the aligned feature table and peak widths that the sweep's sigma_t rule (half the median XCMS peak width) is derived from)
- **dynamicTreeCut** (Cuts the hierarchical clustering dendrogram at each parameter combination so multi-feature cluster counts can be tabulated)
- **R** (Implementation environment for the sweep; RAMClustR is released as a freely available R package)

## Examples

```
install_github("cbroeckl/RAMClustR", build_vignettes = TRUE, dependencies = TRUE)
```

## Evaluation signals

- The reproduced optimum falls at sigma_t = 2 and sigma_r = 0.5 with an approximate multi-feature cluster count of ~2500 for the faahKO 441-combination sweep
- The similarity surface shows a plateau/large stable region rather than a sharp peak, matching Figure 2a-c
- At every grid point, cluster quality equals the dot-product similarity between clustered features and the count includes only clusters with >= 2 features
- sigma_t values are consistent with the stated rule that optimal sigma_t equals half the median XCMS peak width
- Grid extent, optimal pair, and approximate cluster count are written to sigma_sweep_results.json

## Limitations

- The optimum is a broad plateau, so nearby parameter values may be equally acceptable; a single 'best' value can be misleading without inspecting the stable region
- Sweep cost scales with the grid size (441 combinations in the paper) since full hierarchical clustering and dendrogram cutting are repeated at every combination
- Cluster quality is measured only by dot-product spectral similarity and multi-feature cluster count; other quality criteria are not assessed in this sweep
- Sensitivity depends on the sigma_t anchoring rule (half the median XCMS peak width); if peak widths are misestimated the grid may not cover the true optimum

## Evidence

- [results] Parameter sweep surface characterization: [evidence span withheld — anchor status: not_found]
- [methods] sigma_t anchoring rule: [evidence span withheld — anchor status: not_found]
