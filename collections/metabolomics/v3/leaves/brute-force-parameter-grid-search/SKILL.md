---
name: brute-force-parameter-grid-search
description: Use when a task requires choosing optimal numeric weights/thresholds
  by exhaustive enumeration against a ground-truth reference, e.g. calibrating BiG-SCAPE
  JI/DSS/GK/AI weights per BGC class by maximizing correlation with MIBiG compound
  structural distances.
when_to_use_negative:
- When no independent reference (e.g. MIBiG compound annotations) is available to
  correlate candidate parameter settings against — there is no objective function
  to maximize.
- When the parameter space is high-dimensional or continuous beyond a feasible dense
  grid (e.g. more than four weights at 0.01 increments); brute force becomes computationally
  prohibitive.
- When running BiG-SCAPE for standard GCF network construction with defaults — recalibration
  is only needed to produce class-specific weights.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: BiG-SCAPE
  role: Source of the pairwise distance components (Jaccard Index, Domain Sequence
    Similarity, Adjacency Index) whose per-class weights are being optimized via grid
    search
  repo: https://github.com/medema-group/BiG-SCAPE
- name: RDKit
  role: Computes Morgan fingerprint (radius=4) Tanimoto structural distances between
    MIBiG compounds, serving as the reference matrix the weighted BGC distances are
    correlated against
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:63b3c1def57f81176e0c741f9089654822cd4adaa7a6a510abd650d98c0ea3ee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/medema-group/BiG-SCAPE
  tools:
  - BiG-SCAPE
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# brute-force-parameter-grid-search

## Summary

Systematically enumerates a dense grid of numeric parameter combinations and selects the setting that maximizes agreement with an independent reference signal. In this paper it calibrates per-class BiG-SCAPE distance-index weights (JI, DSS, GK, AI) against MIBiG compound Tanimoto distances.

## When to use

Use when a task requires choosing optimal numeric weights/thresholds by exhaustive enumeration against a ground-truth reference, e.g. calibrating BiG-SCAPE JI/DSS/GK/AI weights per BGC class by maximizing correlation with MIBiG compound structural distances.

## When NOT to use

- When no independent reference (e.g. MIBiG compound annotations) is available to correlate candidate parameter settings against — there is no objective function to maximize.
- When the parameter space is high-dimensional or continuous beyond a feasible dense grid (e.g. more than four weights at 0.01 increments); brute force becomes computationally prohibitive.
- When running BiG-SCAPE for standard GCF network construction with defaults — recalibration is only needed to produce class-specific weights.

## Inputs

- MIBiG v1.3 compound SMILES set (reference structural ground truth)
- BiG-SCAPE pairwise distance components per BGC class: Jaccard Index, Domain Sequence Similarity, Goodman-Kruskal, Adjacency Index
- BiG-SCAPE class assignments (the eight BiG-SCAPE classes)

## Outputs

- optimized_class_weights.tsv with one optimized weight row per BiG-SCAPE class
- Per-class correlation values of the weighted BGC distance vs the MIBiG compound Tanimoto distance matrix
- Per-class comparison showing Adjacency Index outperforming Goodman-Kruskal under optimized weights

## How to apply

Parse the MIBiG v1.3 compound SMILES set and compute an all-pairs structural distance matrix with RDKit using Morgan fingerprints (radius=4) scored by Tanimoto coefficient — this is the fixed reference signal. Load the BiG-SCAPE pairwise distance components per BGC class: Jaccard Index of shared domain types, Domain Sequence Similarity, Goodman-Kruskal, and Adjacency Index. Brute-force enumerate weight combinations per BiG-SCAPE class, stepping JI/DSS/GK/AI weights in increments of 0.01 and the anchorboost weights 1–4 in steps of 0.5. For each candidate, form the weighted BGC distance and correlate it with the compound Tanimoto distance matrix; retain the weight set that maximizes agreement, done separately per class because BiG-SCAPE 'accounts for differences in modes of evolution between BGC classes'. Finally verify that under the optimized weights the Adjacency Index correlates better with compound Tanimoto distances than the Goodman-Kruskal metric for each class, and write the results as a per-class weight table.

## Related tools

- **BiG-SCAPE** (Source of the pairwise distance components (Jaccard Index, Domain Sequence Similarity, Adjacency Index) whose per-class weights are being optimized via grid search) — https://github.com/medema-group/BiG-SCAPE
- **RDKit** (Computes Morgan fingerprint (radius=4) Tanimoto structural distances between MIBiG compounds, serving as the reference matrix the weighted BGC distances are correlated against)

## Examples

```
best = max(itertools.product(np.arange(0,1.01,0.01), repeat=4), key=lambda w: pearsonr(w @ D_bgc, D_tanimoto)[0])
```

## Evaluation signals

- optimized_class_weights.tsv contains exactly one optimized weight row for each of the eight BiG-SCAPE classes
- For every class, the correlation of the Adjacency Index weighting with compound Tanimoto distances exceeds that of the Goodman-Kruskal metric
- Enumerated JI/DSS/GK/AI weight values fall on the 0.01 grid and anchorboost weights 1–4 fall on the 0.5 step grid
- The selected per-class weight combination is the argmax of the correlation against the MIBiG Tanimoto matrix (Morgan fingerprints, radius=4)
- Weighted BGC distances are recomputed consistently with the same per-class component definitions BiG-SCAPE uses for pairwise distance calculation

## Limitations

- Brute-force enumeration over dense grids (0.01 increments across four weights) scales combinatorially; the search is only tractable because it is split per BiG-SCAPE class.
- Optimized weights depend on the reference set (MIBiG v1.3) and fingerprint choice (Morgan radius=4, Tanimoto); they may not transfer to other compound spaces.
- Grid granularity caps achievable precision — the true optimum can lie between grid points, similar to how the paper's network cutoff of 0.75 was itself an empirical compromise maximizing node count while minimizing impact on network structural integrity.

## Evidence

- [readme] BiG-SCAPE computes the distances being weighted: [evidence span withheld — anchor status: external_doc]
