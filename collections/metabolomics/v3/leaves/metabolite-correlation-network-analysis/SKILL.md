---
name: metabolite-correlation-network-analysis
description: Use when you have a processed samples-by-metabolites table (e.g., from
  Viime's normalized/transformed/scaled CSV output) and need to identify and visualize
  pairs of metabolites whose abundance profiles co-vary across samples, e.g., correlated
  behavior of metabolites measured on different platforms.
when_to_use_negative:
- The data has not been imputed, normalized, transformed, and scaled in Viime — correlations
  computed on raw concentration columns with missing values or heterogeneous scaling
  are not comparable.
- The analysis goal is univariate group comparison (Wilcoxon rank sum test or multivariate
  ANOVA with post-hoc Tukey) rather than pairwise metabolite co-variation — use Viime's
  univariate/multivariate analyses instead.
- Only a single metabolite column (no pairwise structure) is present, so no pairwise
  correlation can be computed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3417
- http://edamontology.org/topic_2269
tools:
- name: Viime
  role: source application whose network diagram module this skill reconstructs; provides
    the Node Filter and minimum correlation threshold controls for the correlation
    network
  repo: girder/viime
- name: Pandas
  role: computes the pairwise metabolite correlation matrix via corr() on the metabolite
    columns of the loaded samples x metabolites data frame
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:d5a5fdefe7e1c1d4c2da92f19eaf09827c10a24f8b51325963c76502afcb856a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3417
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolite-correlation-network-analysis

## Summary

Constructs a metabolite correlation network from a processed samples-by-metabolites intensity table by computing a full pairwise correlation matrix (Pearson, Kendall Tau, or Spearman) and drawing edges only between metabolite pairs whose absolute correlation exceeds a configurable minimum threshold, with signed, width-encoded edge weights.

## When to use

Use when you have a processed samples-by-metabolites table (e.g., from Viime's normalized/transformed/scaled CSV output) and need to identify and visualize pairs of metabolites whose abundance profiles co-vary across samples, e.g., correlated behavior of metabolites measured on different platforms.

## When NOT to use

- The data has not been imputed, normalized, transformed, and scaled in Viime — correlations computed on raw concentration columns with missing values or heterogeneous scaling are not comparable.
- The analysis goal is univariate group comparison (Wilcoxon rank sum test or multivariate ANOVA with post-hoc Tukey) rather than pairwise metabolite co-variation — use Viime's univariate/multivariate analyses instead.
- Only a single metabolite column (no pairwise structure) is present, so no pairwise correlation can be computed.

## Inputs

- processed metabolite intensity table as samples x metabolites CSV (post-normalization, transformation, and scaling in Viime)
- configuration parameters: correlation method (Pearson | Kendall Tau | Spearman) and minimum correlation threshold

## Outputs

- filtered weighted edge list metabolite_correlation_edges.csv with columns for both metabolite identifiers and the signed correlation weight
- network diagram with metabolite nodes joined by edges whose color encodes correlation sign (red negative, gray positive) and width encodes strength

## How to apply

Load the processed metabolite intensity table (samples x metabolites) from the input CSV into a pandas data frame, then compute the full pairwise metabolite correlation matrix with pandas' corr() using the selected method — Pearson, Kendall Tau, or Spearman rank. Reshape the matrix into pairwise edge records, retaining the signed correlation coefficient as the edge weight for each metabolite pair. Filter the edges, keeping only pairs whose absolute correlation exceeds the configurable minimum correlation threshold. Write the filtered weighted edge list (metabolite identifiers plus signed correlation weight) to metabolite_correlation_edges.csv; when rendered as a network diagram, negative correlations are drawn in red, positive in gray, and edge width encodes correlation strength. The rationale is that Viime's fused datasets expand metabolome coverage, so edges exceeding the threshold reveal correlated behavior of metabolites detected by different platforms.

## Related tools

- **Viime** (source application whose network diagram module this skill reconstructs; provides the Node Filter and minimum correlation threshold controls for the correlation network) — girder/viime
- **Pandas** (computes the pairwise metabolite correlation matrix via corr() on the metabolite columns of the loaded samples x metabolites data frame)

## Examples

```
import pandas as pd; df = pd.read_csv('metabolites.csv'); corr = df.corr(method='spearman'); edges = corr.stack()[(corr.stack().abs() > min_corr)]
```

## Evaluation signals

- Every emitted edge in metabolite_correlation_edges.csv has an absolute correlation strictly above the configured minimum threshold.
- Each edge weight carries the correct sign of the computed correlation (negative weights correspond to negative correlations).
- All metabolite pairs present in the input table were evaluated (no pair was silently dropped before thresholding).
- The correlation matrix matches pandas corr() output for the chosen method (Pearson, Kendall Tau, or Spearman) on the same input columns.
- In the rendered network, red edges are exactly the negative-correlation pairs and gray edges the positive ones, with widths monotonic in correlation magnitude.

## Limitations

- Edges are thresholded at a configurable minimum correlation, so weak correlations below the cutoff are never drawn; the threshold must be tuned per dataset.
- Missing values must be resolved first — Viime imputes missing data with MNAR/MCAR classification before analysis, and correlation on unimputed data will degrade or fail.
- Correlation alone does not establish causality and is computed only between metabolite columns in the table; annotations are not required but platform-specific batch effects can produce spurious edges.

## Evidence

- [abstract] The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms.: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
- [abstract] network node filter and minimum correlation threshold adjustment: "The Node Filter and Node Color options enables the nodes to be selected or colored based on the data source or significance."
- [full_text] correlation computation and thresholding workflow: [evidence span withheld — anchor status: not_found]
- [full_text] edge rendering semantics: [evidence span withheld — anchor status: not_found]
- [abstract] backend leverage of pandas: "The backend leverages Pandas for raw file parsing and normalization."
