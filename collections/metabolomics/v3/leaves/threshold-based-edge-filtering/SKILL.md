---
name: threshold-based-edge-filtering
description: Use when you have a processed samples-by-metabolites intensity table
  from Viime and want a network diagram revealing correlated metabolite behavior,
  including cross-platform (MS/NMR fused) relationships.
when_to_use_negative:
- Your goal is a heatmap or volcano plot view of significant metabolites rather than
  pairwise metabolite-metabolite edge structure — use Viime's Heatmaps or volcano
  plot pages instead.
- 'The input table is still raw/unnormalized: Viime expects the workflow to pick up
  after preprocessing, imputation (MCAR/MNAR), normalization, transformation, and
  scaling, before correlation-based network construction.'
- You need pairwise correlations between samples rather than between metabolites;
  the skill computes metabolite x metabolite correlations only.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3172
tools:
- name: Viime
  role: web application that hosts the correlation network diagram page, exposes the
    minimum correlation threshold and node filter/color controls, and renders the
    signed, width-encoded edges
  repo: https://github.com/girder/viime
- name: Pandas
  role: backend library used by Viime for raw file parsing and for computing the pairwise
    metabolite correlation matrix via corr()
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:e4f407165a5ec0fca1c8925895942238553f39d895db8b0abc98efb9fe502b58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# threshold-based-edge-filtering

## Summary

Build a metabolite correlation network by computing pairwise metabolite correlations (Pearson, Kendall Tau, or Spearman) on the Viime intensity table and drawing edges only for metabolite pairs whose absolute correlation exceeds a configurable minimum threshold.

## When to use

Use when you have a processed samples-by-metabolites intensity table from Viime and want a network diagram revealing correlated metabolite behavior, including cross-platform (MS/NMR fused) relationships.

## When NOT to use

- Your goal is a heatmap or volcano plot view of significant metabolites rather than pairwise metabolite-metabolite edge structure — use Viime's Heatmaps or volcano plot pages instead.
- The input table is still raw/unnormalized: Viime expects the workflow to pick up after preprocessing, imputation (MCAR/MNAR), normalization, transformation, and scaling, before correlation-based network construction.
- You need pairwise correlations between samples rather than between metabolites; the skill computes metabolite x metabolite correlations only.

## Inputs

- processed metabolite intensity table (samples x metabolites) in CSV format as loaded by Viime
- configurable minimum correlation threshold value
- choice of correlation method: Pearson, Kendall Tau, or Spearman rank

## Outputs

- metabolite_correlation_edges.csv — filtered weighted edge list with metabolite identifiers and signed correlation weights
- network diagram of metabolite nodes linked by edges (red = negative, gray = positive; edge width encodes correlation strength)

## How to apply

Load the processed metabolite intensity table (samples x metabolites) from the input CSV into a pandas data frame, then compute the full pairwise metabolite correlation matrix using the selected method — Pearson, Kendall Tau, or Spearman rank — via pandas' corr() on the metabolite columns. Reshape the matrix into signed edge records where each edge weight carries the correlation's sign (negative correlations render red, positive render gray) and width encodes correlation strength. Filter the edge list, keeping only pairs whose absolute correlation exceeds the configurable minimum threshold, then write the filtered weighted edge list (both metabolite identifiers plus the signed correlation weight) to metabolite_correlation_edges.csv. In the Viime frontend, the Node Filter and Node Color options further allow nodes to be selected or colored by data source or significance, complementing the edge threshold as the main pruning decision. The rationale is that the threshold prunes weak, likely spurious correlations so the rendered network reflects only strong metabolite–metabolite relationships, which is especially meaningful on fused datasets where metabolites detected by different platforms (MS and NMR) can show correlated behavior.

## Related tools

- **Viime** (web application that hosts the correlation network diagram page, exposes the minimum correlation threshold and node filter/color controls, and renders the signed, width-encoded edges) — https://github.com/girder/viime
- **Pandas** (backend library used by Viime for raw file parsing and for computing the pairwise metabolite correlation matrix via corr())

## Examples

```
df.corr(method='spearman')  # on the samples x metabolites table, then filter |rho| > min_threshold
```

## Evaluation signals

- Every emitted edge in metabolite_correlation_edges.csv must have an absolute correlation strictly above the configured minimum threshold.
- Edge weights must retain the correct sign of the computed correlation (negative vs. positive), matching the red/gray edge coloring in the diagram.
- All metabolite pairs present in the input table must have been evaluated before filtering — no metabolite silently dropped from the correlation matrix.
- Changing the minimum correlation threshold interactively must change which edges appear/disappear in the rendered network, confirming the threshold is the operative filter.
- The chosen method (Pearson, Kendall Tau, or Spearman) must be the one actually used to compute the weights, verifiable by recomputing the matrix on the same table.

## Limitations

- The minimum correlation threshold is user-configurable with no article-stated default, so threshold choice directly shapes network density and must be reported.
- Edge filtering is purely on absolute correlation magnitude; no article-described correction (e.g., FDR, p-value adjustment) is applied to correlations.
- Missing values should already be handled by Viime's upstream imputation (MNAR via missForest/random forest, MCAR via KNN); unimputed data would bias correlations.

## Evidence

- [abstract] edge threshold method: "The Node Filter and Node Color options enables the nodes to be selected or colored based on the data source or significance."
- [intro] pairwise correlation computation: "The backend leverages Pandas for raw file parsing and normalization."
- [other] network edge coloring: [evidence span withheld — anchor status: not_found]
- [abstract] fused dataset rationale: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
