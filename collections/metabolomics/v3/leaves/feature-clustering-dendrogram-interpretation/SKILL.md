---
name: feature-clustering-dendrogram-interpretation
description: Use when a metaboprep Metaboprep object has passed quality_control and
  you need to inspect the hierarchical clustering of metabolite features (qc_tree)
  before cutting clusters at a tree_cut_height or reporting QC results in an HTML
  report.
when_to_use_negative:
- The Metaboprep object has not yet been through quality_control, so no qc_tree attribute
  exists on feature_summary
- The dataset is a generic feature table outside the metaboprep object model (e.g.
  plain CSV of metabolite abundances) — construct a Metaboprep object first
- You only need sample-level QC summaries (sample_summary, PCA outlier flags) rather
  than feature clustering structure
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: Computes the feature clustering tree during quality_control and stores it
    as the 'qc_tree' attribute on the feature summary; the tree_cut_height parameter
    governs cluster cutting
  repo: https://github.com/MRCIEU/metaboprep
- name: dendextend
  role: R package used in the metaboprep methods/vignette for manipulating and rendering
    the feature dendrogram
- name: R
  role: Runtime environment in which the tree is extracted and plotted with base graphics
    (plot(tree, hang=-1))
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:98a33b0a14936f0f7aeb8ea81564c99f93653dc4bdca0b2e5a2be45ca70c1967
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - dendextend
  - metaboprep
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# feature-clustering-dendrogram-interpretation

## Summary

Visualize and interpret the feature clustering tree produced by the metaboprep quality_control pipeline to assess metabolite correlation structure and feature independence, using the dendrogram stored on the feature summary object and dendextend for rendering.

## When to use

Use when a metaboprep Metaboprep object has passed quality_control and you need to inspect the hierarchical clustering of metabolite features (qc_tree) before cutting clusters at a tree_cut_height or reporting QC results in an HTML report.

## When NOT to use

- The Metaboprep object has not yet been through quality_control, so no qc_tree attribute exists on feature_summary
- The dataset is a generic feature table outside the metaboprep object model (e.g. plain CSV of metabolite abundances) — construct a Metaboprep object first
- You only need sample-level QC summaries (sample_summary, PCA outlier flags) rather than feature clustering structure

## Inputs

- Metaboprep S7 object with input and qc data layers after quality_control
- feature_summary object carrying the 'qc_tree' clustering attribute

## Outputs

- Dendrogram plot of the feature tree (e.g. 'Example Dataset Feature Tree')
- Interpreted cluster structure at the user-defined tree_cut_height (0.5 in the reference workflow)
- Dendrogram panel embedded in the rendered qc_report HTML

## How to apply

Run the metaboprep quality_control pipeline (e.g. with sample_missingness=0.2, feature_missingness=0.2, tree_cut_height=0.5) so that a feature tree is computed and attached to the feature summary as the 'qc_tree' attribute. Extract it with attr(mydata@feature_summary, "qc_tree") and plot it as a dendrogram with hang = -1 and cex = 0.75 to display how metabolites group by correlation/missingness structure. Interpret the dendrogram jointly with the tree_cut_height parameter: clusters cut at the specified height define correlated feature groups, and feature_selection='least_missingness' then picks representatives within clusters. The rendered dendrogram is also a required panel of the generated HTML qc_report, alongside PC plots and batch-variable eta-squared boxplots.

## Related tools

- **metaboprep** (Computes the feature clustering tree during quality_control and stores it as the 'qc_tree' attribute on the feature summary; the tree_cut_height parameter governs cluster cutting) — https://github.com/MRCIEU/metaboprep
- **dendextend** (R package used in the metaboprep methods/vignette for manipulating and rendering the feature dendrogram)
- **R** (Runtime environment in which the tree is extracted and plotted with base graphics (plot(tree, hang=-1))) — https://github.com/MRCIEU/metaboprep

## Examples

```
tree <- attr(mydata@feature_summary, "qc_tree"); plot(tree, hang = -1, cex = 0.75, main = "Feature Tree", sub = "", xlab = "")
```

## Evaluation signals

- attr(mydata@feature_summary, "qc_tree") returns a non-NULL tree object, confirming quality_control populated the feature clustering
- The dendrogram leaves correspond to feature_ids retained in the qc layer; excluded features (e.g. extreme/user-defined missingness) should not appear
- The plotted tree uses hang = -1 so all leaves align on a common baseline, and cex = 0.75 keeps labels legible
- The dendrogram appears in the generated HTML report as the feature-clustering panel together with PC plots and Shapiro W-statistic histograms, matching the deposited Supplementary Data reports
- Clusters cut at tree_cut_height=0.5 are consistent with the feature_selection='least_missingness' representative choices

## Limitations

- The tree only exists after quality_control has run; before that, the feature summary carries no qc_tree attribute
- The number of informative PCs and cluster structure depend on dataset size — in the README example only 2 informative PCs were available, below the stated max_num_pcs=10, and similar small-dataset effects can affect clustering
- Cluster interpretation is threshold-dependent: different tree_cut_height values yield different feature groupings

## Evidence

- [readme] tree extraction from feature summary: [evidence span withheld — anchor status: external_doc]
- [readme] dendrogram plotting parameters: [evidence span withheld — anchor status: external_doc]
- [methods] tree_cut_height threshold: "tree_cut_height     = 0.5"
- [methods] feature selection within clusters: "feature_selection   = "least_missingness""
- [supplementary] report includes dendrogram panel: [evidence span withheld — anchor status: not_found]
