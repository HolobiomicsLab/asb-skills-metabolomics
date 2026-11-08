---
name: differential-metabolite-abundance-testing
description: Use when you have matrices normalized by different strategies (tGAM/rGAM/rLOESS,
  QC-RLSC, SERFF, TIGER) and need to quantify how the choice of normalization changes
  the differential abundance hit set.
when_to_use_negative:
- Input is raw unnormalized data — the skill presupposes matrices already normalized
  with metanorm() under matched QCcheck = TRUE and batchwise settings
- Goal is to assess normalization quality directly (QC drift, batch-effect removal
  in PCA score plots) rather than its effect on downstream differential abundance
  hit sets
- The dataset has no replicate or contrast structure supporting differential abundance
  testing
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: metanorm
  role: Produces the alternative normalized matrices (tGAM default, rGAM, rLOESS,
    QC-RLSC with QConly = TRUE, SERFF, TIGER) whose DAM sets are compared
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime (>= 4.4.0) for running differential abundance testing, overlap statistics,
    and Spearman/Euclidean comparisons
- name: ggplot2
  role: Rendering the Fig. 5-style heatmap of cross-strategy correlations/distances
- name: cowplot
  role: Assembling heatmap/PCA panels into the comparison figure
- name: plotPCA
  role: Generates PCA score plots to verify batch effects diminished before DAM comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:9681fe8c8c4f0315b391b510d3132c4e856d488d4717cf8c88b902ca7a04eff4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - R
  - cowplot
  - ggplot2
  - metanorm
  - pak
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# differential-metabolite-abundance-testing

## Summary

Runs differential metabolite abundance (DAM) testing on normalized LC-MS metabolomics matrices (e.g., ENVIRONAGE urine, 393 runs × 269 metabolites) produced by alternative normalization strategies, then quantifies agreement between strategies via DAM overlap percentages, pairwise Spearman correlations, and Euclidean distances to benchmark normalization methods on downstream statistical power.

## When to use

Use when you have matrices normalized by different strategies (tGAM/rGAM/rLOESS, QC-RLSC, SERFF, TIGER) and need to quantify how the choice of normalization changes the differential abundance hit set.

## When NOT to use

- Input is raw unnormalized data — the skill presupposes matrices already normalized with metanorm() under matched QCcheck = TRUE and batchwise settings
- Goal is to assess normalization quality directly (QC drift, batch-effect removal in PCA score plots) rather than its effect on downstream differential abundance hit sets
- The dataset has no replicate or contrast structure supporting differential abundance testing

## Inputs

- Normalized metabolite abundance matrices (numerical matrix, compounds × sample runs) from tGAM/rGAM/rLOESS, QC-RLSC, SERFF and TIGER normalization
- Batch annotation vector per sample run
- QC annotation vector (metanorm.qc-style 'QC' vs other sample types)
- Reference DAM set from the default (tGAM) strategy for overlap computation

## Outputs

- DAM_overlap_summary.csv (overlap percentages per strategy vs reference)
- Pairwise Spearman correlation matrix and Euclidean distance matrix across normalized data matrices
- Figure5_heatmap.png (clustering of strategies by matrix similarity)

## How to apply

Load each normalized metabolite-by-sample matrix (rows = compounds, columns = runs) generated with metanorm() under identical batch and QC settings, then run differential metabolite abundance testing separately on each strategy's matrix. For each strategy compute the percentage overlap of its DAM set with the reference strategy's DAM set, assembling the percentages into a summary table with expected ranges (robust 74–88%, QC-RLSC 47–51%, SERFF 68–70%, TIGER 64–67%). Complement the overlap analysis with pairwise Spearman correlations and Euclidean distances between the normalized data matrices across strategies, and render a heatmap (Fig. 5-style) using plotPCA/ggplot2/cowplot. Apply multiple testing correction for the downstream statistical inference, which the authors flag as essential even though it was outside their benchmark scope. Validation: check that overlap percentages fall within the reported ranges and that QC-RLSC emerges as the weakest-agreement strategy on ENVIRONAGE.

## Related tools

- **metanorm** (Produces the alternative normalized matrices (tGAM default, rGAM, rLOESS, QC-RLSC with QConly = TRUE, SERFF, TIGER) whose DAM sets are compared) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime (>= 4.4.0) for running differential abundance testing, overlap statistics, and Spearman/Euclidean comparisons)
- **ggplot2** (Rendering the Fig. 5-style heatmap of cross-strategy correlations/distances)
- **cowplot** (Assembling heatmap/PCA panels into the comparison figure)
- **plotPCA** (Generates PCA score plots to verify batch effects diminished before DAM comparison)

## Evaluation signals

- DAM overlap percentages fall within the reported ranges: robust 74–88%, QC-RLSC 47–51%, SERFF 68–70%, TIGER 64–67%
- QC-RLSC shows the weakest agreement with the robust reference methods (47–51% DAM overlap), while SERFF (68–70%) and TIGER (64–67%) differ less strongly
- The rendered heatmap reproduces the clustering pattern of Fig. 5 (Spearman correlations / Euclidean distances between strategies)
- All strategies were normalized with identical batch and QC settings (QCcheck = TRUE, batchwise), so DAM differences reflect normalization, not preprocessing asymmetry
- Outputs saved as DAM_overlap_summary.csv and Figure5_heatmap.png

## Limitations

- Multiple testing correction is essential for downstream statistical inference but was explicitly not in the scope of the source work
- Outliers carried through inadequate normalization reduce statistical power in downstream analyses such as differential metabolite abundance testing, biasing DAM sets
- Differences between QC and biological samples may invalidate RSD-based filtering performed on QC samples prior to testing
- No exact Metanorm package version is pinned in the article's discussion; R >= 4.4.0 is the only stated requirement

## Evidence

- [intro] Outliers reduce downstream statistical power: "We subsequently show that such outliers go on to reduce statistical power in downstream analyses, such as differential metabolite abundance testing"
- [discussion] Multiple testing correction requirement: "While not in the scope of this work, multiple testing correction is essential for downstream statistical inference."
- [discussion] QC RSD filtering caveat: "differences between QC and biological samples may invalidate filtering on e.g., relative standard deviation (RSD) observed in QC samples"
- [methods] QCcheck setting for normalization preceding DAM testing: "QCcheck = TRUE,      # check whether QCs are representative"
- [readme] Example dataset structure (compounds × samples matrix): "rawdata, a numerical matrix containing the unnormalized data (rows = compounds, columns = samples)"
