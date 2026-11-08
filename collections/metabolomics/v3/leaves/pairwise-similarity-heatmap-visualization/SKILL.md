---
name: pairwise-similarity-heatmap-visualization
description: Use when several normalization outputs (e.g. tGAM, rGAM, rLOESS, QC-RLSC,
  SERFF, TIGER) of the same LC-MS metabolomics matrix must be compared, to visualize
  cross-method agreement via pairwise Spearman correlation heatmaps alongside DAM
  overlap percentages.
when_to_use_negative:
- When the goal is to judge normalization quality of a single strategy on its own
  (use the individual pre-/post-normalization intensity-vs-order and PCA plots instead)
  rather than to compare several strategies head-to-head
- When datasets contain no common QC annotations or when strategies are applied on
  differently aligned/detected metabolite sets, making Spearman correlation across
  the full feature space meaningless
- As a substitute for the RSD/CV-based QC filtering or tGAM model selection steps
  — this is a downstream visualization/comparison step, not a normalization or filtering
  step
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm R package
  role: Source of metanorm() and plotPCA() calls that generate the normalized matrices
    and PCA plots whose pairwise similarities are visualized
  repo: https://github.com/UGent-LIMET/Metanorm
- name: ggplot2
  role: Rendering layer for the pairwise similarity heatmap of Figure 5
- name: cowplot
  role: Layout/composition of the Figure 5 panel (heatmap plus PCA score plots)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:7ae63712388fd79e10982c05f9823af897d9bde1168318f804072432c48233da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm R package
  - R
  - cowplot
  - ggplot2
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# pairwise-similarity-heatmap-visualization

## Summary

Renders a heatmap of pairwise similarities (Spearman correlations) between normalized metabolomics data matrices across normalization strategies, as used in the Metanorm paper's Figure 5 to visually compare tGAM/rGAM/rLOESS, QC-RLSC, SERFF and TIGER agreement on the ENVIRONAGE dataset.

## When to use

Use when several normalization outputs (e.g. tGAM, rGAM, rLOESS, QC-RLSC, SERFF, TIGER) of the same LC-MS metabolomics matrix must be compared, to visualize cross-method agreement via pairwise Spearman correlation heatmaps alongside DAM overlap percentages.

## When NOT to use

- When the goal is to judge normalization quality of a single strategy on its own (use the individual pre-/post-normalization intensity-vs-order and PCA plots instead) rather than to compare several strategies head-to-head
- When datasets contain no common QC annotations or when strategies are applied on differently aligned/detected metabolite sets, making Spearman correlation across the full feature space meaningless
- As a substitute for the RSD/CV-based QC filtering or tGAM model selection steps — this is a downstream visualization/comparison step, not a normalization or filtering step

## Inputs

- Normalized (and raw) metabolomics data matrices from multiple normalization strategies (rows = compounds, columns = samples), e.g. output of metanorm() for tGAM, rGAM, rLOESS, QC-RLSC, SERFF, TIGER
- Sample-type vector (QC vs biological sample), batch vector, and batch-wise metanorm.qc annotations
- DAM sets per strategy (percent overlap values to be annotated alongside the heatmap)

## Outputs

- Figure5_heatmap.png — pairwise Spearman correlation heatmap between normalization strategies
- DAM_overlap_summary.csv — summary of DAM overlap percentages per strategy
- PCA score plots (plotPCA) before and after normalization, labelled by batch

## How to apply

Run each candidate normalization strategy (robust tGAM default, plus rGAM, rLOESS, and non-robust QC-RLSC with QConly = TRUE, SERFF, TIGER) through metanorm() on the ENVIRONAGE urine matrix (393 runs × 269 metabolites) with identical batch and QC settings. Compute the percentage overlap of differentially abundant metabolites (DAMs) against the reference strategy's DAM set, and additionally compute pairwise Spearman correlations between the resulting normalized data matrices. Assemble these similarity values into a heatmap labeled by strategy (Fig. 5 in the paper) using ggplot2/cowplot, co-rendering the PCA score plots via plotPCA for batch-effect inspection. Validate that observed agreement percentages fall within reported ranges (robust 74–88%, QC-RLSC 47–51%, SERFF 68–70%, TIGER 64–67%).

## Related tools

- **Metanorm R package** (Source of metanorm() and plotPCA() calls that generate the normalized matrices and PCA plots whose pairwise similarities are visualized) — https://github.com/UGent-LIMET/Metanorm
- **ggplot2** (Rendering layer for the pairwise similarity heatmap of Figure 5)
- **cowplot** (Layout/composition of the Figure 5 panel (heatmap plus PCA score plots))

## Examples

```
Rscript -e 'library(metanorm); normdat <- metanorm(rawdata, model="tGAM", type=metanorm.qc, QCcheck=TRUE, batch=batch); plotPCA(normdat, type=batch)'
```

## Evaluation signals

- Per-strategy DAM overlap percentages fall within the reported ranges: robust 74–88%, QC-RLSC 47–51%, SERFF 68–70%, TIGER 64–67% on the ENVIRONAGE urine dataset
- Heatmap reproduces the clustering pattern of Figure 5, with robust tGAM/rGAM/rLOESS mutually most similar and QC-RLSC weakest (47–51% overlap)
- All strategies were run with identical batch/QC settings (QCcheck = TRUE, batchwise normalization) before computing similarities, so any disagreement reflects the normalization model, not data handling
- Pairwise Spearman correlations between normalized matrices are computed feature-wise (over metabolites) on the same sample runs

## Limitations

- tGAM has a higher computational cost than rLOESS and rGAM, though normalization of thousands of metabolites and hundreds of samples still completes within minutes
- Metanorm's GAM-based procedures revert to sample-based normalization adaptively; QC-RLSC shows the weakest agreement with robust methods (47–51% DAM overlap) on the ENVIRONAGE data
- Multiple testing correction is essential for downstream statistical inference but is not the scope of the comparison itself

## Evidence

- [abstract] tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice.: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [readme] Install the R package and load example data: "The package can be installed directly from R as follows:

```r
if (!require("pak", quietly = TRUE))
    install.packages("pak")

pak::pak("UGent-LIMET/Metanorm")
```"
- [readme] metanorm call with QCcheck and batch args: "normdat <- metanorm(rawdata[1:5,],       # numerical data matrix to normalize
	                model = "tGAM",      # default tGAM method
                    type = metanorm.qc,  # vector with sample"
- [readme] PCA plot of pre/post normalization: "plotPCA(rawdata[1:5,], type = batch)
plotPCA(normdat[1:5,], type = batch)"
- [readme] QC-RLSC with QConly argument: "normdat2 <- metanorm(rawdata[1:5,],       # numerical data matrix to normalize
                     model = "QC-RLSC",   # use QC-RLSC
                     type = metanorm.qc,  # vector with sample"
