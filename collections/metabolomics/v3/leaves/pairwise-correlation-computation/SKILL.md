---
name: pairwise-correlation-computation
description: Use when you have a processed samples-by-metabolites table (post Viime
  imputation/normalization/transformation/scaling) and need to explore correlated
  behavior of metabolites, e.g. to build a metabolite network diagram whose edges
  encode signed correlation strength.
when_to_use_negative:
- The input table has not yet gone through Viime's imputation, normalization, transformation,
  and scaling steps — correlations on raw concentrations will be dominated by platform-dependent
  magnitude effects.
- The goal is identifying significantly altered metabolites between groups; use Viime's
  Wilcoxon rank sum test or ANOVA with post-hoc Tukey instead of pairwise correlations.
- The input is a fused multi-platform dataset and you only need concatenative/mid-level/multi-block
  data fusion, not inter-metabolite edge extraction.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Viime
  role: web application whose network diagram consumes the threshold-filtered signed
    correlation edges; exposes the minimum correlation threshold adjustment
  repo: https://github.com/girder/viime
- name: Pandas
  role: provides corr() for computing the pairwise metabolite correlation matrix (Pearson,
    Kendall Tau, Spearman) on the metabolite columns of the processed CSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:3c1f8a6c4af30bc33375d093b9667a3a6feafb839e3f0aed33c0b4c88b474cfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pairwise-correlation-computation

## Summary

Compute pairwise metabolite correlations on the processed Viime metabolite intensity table using Pearson, Kendall Tau, or Spearman rank methods (via Pandas corr()), then emit a threshold-filtered signed weighted edge list for the network diagram.

## When to use

Use when you have a processed samples-by-metabolites table (post Viime imputation/normalization/transformation/scaling) and need to explore correlated behavior of metabolites, e.g. to build a metabolite network diagram whose edges encode signed correlation strength.

## When NOT to use

- The input table has not yet gone through Viime's imputation, normalization, transformation, and scaling steps — correlations on raw concentrations will be dominated by platform-dependent magnitude effects.
- The goal is identifying significantly altered metabolites between groups; use Viime's Wilcoxon rank sum test or ANOVA with post-hoc Tukey instead of pairwise correlations.
- The input is a fused multi-platform dataset and you only need concatenative/mid-level/multi-block data fusion, not inter-metabolite edge extraction.

## Inputs

- processed metabolite intensity table as CSV (samples x metabolites columns), after Viime imputation, normalization, transformation, and scaling
- configurable minimum correlation threshold
- choice of correlation method: Pearson, Kendall Tau, or Spearman rank

## Outputs

- metabolite_correlation_edges.csv — a filtered weighted edge list with columns for both metabolite identifiers and the signed correlation weight
- edges that feed the Viime network diagram (red edges = negative correlation, gray edges = positive, width encodes strength)

## How to apply

Load the processed metabolite intensity table (samples x metabolites) from the input CSV into a Pandas DataFrame and compute the full pairwise metabolite correlation matrix with pandas corr(), selecting Pearson, Kendall Tau, or Spearman rank as the method. Reshape the correlation matrix into pairwise edge records, retaining the signed correlation coefficient as the edge weight for each metabolite pair. Filter these records, keeping only pairs whose absolute correlation exceeds the configurable minimum correlation threshold exposed by the network diagram's threshold adjustment control. Write the filtered weighted edge list — columns for both metabolite identifiers plus the signed correlation weight — to metabolite_correlation_edges.csv. In the Viime network visualization, negative correlations render as red edges and positive ones as gray, with edge width encoding strength; the Node Filter and Node Color options can further select or color nodes by data source or significance.

## Related tools

- **Viime** (web application whose network diagram consumes the threshold-filtered signed correlation edges; exposes the minimum correlation threshold adjustment) — https://github.com/girder/viime
- **Pandas** (provides corr() for computing the pairwise metabolite correlation matrix (Pearson, Kendall Tau, Spearman) on the metabolite columns of the processed CSV)

## Examples

```
import pandas as pd; corr = pd.read_csv('metabolites.csv').corr(method='spearman')
```

## Evaluation signals

- Every emitted edge in metabolite_correlation_edges.csv has an absolute correlation strictly above the configured minimum threshold.
- Edge weights carry the correct sign of the underlying correlation (negative values correspond to red edges, positive to gray in the diagram).
- All metabolite pairs present in the input table were evaluated — no metabolite columns are silently dropped from the pairwise computation.
- The edge list schema contains columns for both metabolite identifiers plus the signed correlation weight, matching the metabolite_correlation_edges.csv format.
- Switching the method between Pearson, Kendall Tau, and Spearman rank reproduces the same edge set structure but yields method-appropriate coefficient values.

## Limitations

- Edges are only drawn when the pairwise correlation exceeds the configurable minimum threshold, so weakly correlated metabolite pairs are excluded from the network and their behavior is not visible in the diagram.
- The R processing functions exposed via OpenCPU expect a CSV file path and return a data frame; functions added to the viime R package require a Docker image rebuild before they appear in the API.
- The minimum fold change / significance coloring on nodes depends on prior univariate selection (Wilcoxon rank sum / ANOVA with post-hoc Tukey), so network node colors are only meaningful after that upstream step.

## Evidence

- [abstract] correlation threshold: [evidence span withheld — anchor status: not_found]
- [abstract] correlation methods: [evidence span withheld — anchor status: not_found]
- [abstract] edge color and width encoding: [evidence span withheld — anchor status: not_found]
- [abstract] Pandas backend role: "The backend leverages Pandas for raw file parsing and normalization."
- [abstract] correlated behavior across platforms: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
- [abstract] node filter and coloring options: "The Node Filter and Node Color options enables the nodes to be selected or colored based on the data source or significance."
- [readme] README: functions accept CSV and return data frame: "Functions contained in this package
should accept a path to a CSV file as an argument and return a data frame."
- [readme] README: pandas/R frame communication: "The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py)
handles the communication between pandas and R data frames."
