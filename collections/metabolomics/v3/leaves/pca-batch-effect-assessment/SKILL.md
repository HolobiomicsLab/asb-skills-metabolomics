---
name: pca-batch-effect-assessment
description: Use when you need to judge whether a metabolomics normalization method
  (e.g. tGAM, QC-RLSC) removed batch effects before/after correction, or to spot subpar
  normalization.
when_to_use_negative:
- When the data have not yet been normalized with metanorm — you need both rawdata
  and normdat to make the pre/post comparison meaningful.
- When the only annotation available is the QC/sample-type vector and no batch labels
  exist, since the skill's core comparison is batch-labeled score separation.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: plotPCA
  role: Metanorm R function that renders the pre- and post-normalization PC score
    plots labeled by batch
  repo: github.com/UGent-LIMET/Metanorm
- name: metanorm
  role: Produces the normalized matrix (e.g. with model = 'tGAM' or 'QC-RLSC') whose
    batch-effect removal the PC plots assess
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:7f85bb74462c3c105a8cbb7d7ba944ca30512668e86f7e937aa7cfaaaba72887
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - fANCOVA
  - ggplot2
  - metanorm
  - mgcv
  - pak
  - pbapply
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# pca-batch-effect-assessment

## Summary

Generate pre- vs. post-normalization PC score plots (via the Metanorm package's plotPCA) labeled by batch to visually verify that normalization has removed batch effects and that remaining structure is genuine.

## When to use

Use when you need to judge whether a metabolomics normalization method (e.g. tGAM, QC-RLSC) removed batch effects before/after correction, or to spot subpar normalization.

## When NOT to use

- When the data have not yet been normalized with metanorm — you need both rawdata and normdat to make the pre/post comparison meaningful.
- When the only annotation available is the QC/sample-type vector and no batch labels exist, since the skill's core comparison is batch-labeled score separation.

## Inputs

- rawdata: numerical matrix of unnormalized LC-MS metabolite intensities (rows = compounds, columns = sample runs)
- batch: character vector of batch labels, one entry per sample run
- normdat: normalized intensity matrix of the same dimensions (post-metanorm call)

## Outputs

- Pre-normalization PC score plot labeled by batch
- Post-normalization PC score plot labeled by batch
- Per-compound pre- vs. post-normalization intensity-vs-order diagnostic plots in plotdir

## How to apply

Call plotPCA twice from the Metanorm R package: once on the unnormalized intensity matrix (rows = compounds, columns = sample runs) and once on the normalized matrix, passing type = batch so points are labeled by the batch annotation vector. Load the data in the Metanorm format — rawdata matrix plus the batch and sample-type vectors (e.g. 'QC'/'sample') — then compare the pre- and post-normalization PC score plots: batch separation should diminish after effective normalization. Inspect the per-compound pre- vs. post-normalization intensity-vs-order plots written to the plotdir directory to confirm that signal drift is removed in biological (non-QC) samples, not just in QCs — the README warns that PC score plots can look clean while biological-sample drift persists for individual compounds. This QC-oriented visualization is explicitly recommended by the article to identify subpar data quality or aberrant normalization.

## Related tools

- **plotPCA** (Metanorm R function that renders the pre- and post-normalization PC score plots labeled by batch) — github.com/UGent-LIMET/Metanorm
- **metanorm** (Produces the normalized matrix (e.g. with model = 'tGAM' or 'QC-RLSC') whose batch-effect removal the PC plots assess) — github.com/UGent-LIMET/Metanorm

## Examples

```
plotPCA(rawdata[1:5,], type = batch); plotPCA(normdat, type = batch)
```

## Evaluation signals

- PC score plots are generated for both the unnormalized rawdata and the normalized normdat, both colored/labeled by the batch vector
- Visible batch separation in the pre-normalization plot diminishes in the post-normalization plot
- Per-compound intensity-vs-order plots in plotdir show drift removal in biological (non-QC) samples — the README's example (plot number 4 under QC-RLSC) reveals drift in non-QC samples even when the PC score plot appears clean
- The same sample run order and batch annotation vector are passed to both pre- and post-normalization calls so the comparison is directly interpretable

## Limitations

- PC score plots can mask compound-level problems: the README shows a case where no batch effects remain in the PC score plot but per-compound plots reveal aberrant signal drift in biological samples.
- plotPCA is labeled by sample-type argument (type = batch) — the same function serves both QC/sample-type and batch labeling, so arguments must be chosen deliberately.
- The article notes pre- vs. post-normalization and PCA score plots are for efficient identification of subpar quality or aberrant normalization, not a formal quantitative batch-effect metric; multiple testing correction for downstream inference is out of scope.

## Evidence

- [discussion] Pre- vs. post-normalization and PCA score plot inspection for quality control: "The pre- vs. post-normalization and PCA score plots readily generated by the Metanorm package support an efficient identification of subpar data quality or aberrant normalization"
- [readme] Generate PC score plots to see whether batch effects have diminished: "We can generate a PC score plot to see whether batch effects have diminished"
- [readme] Individual compound plots allow finegrained assessment of normalization performance: "Individual compound pre- vs. post-normalization intensity vs. order plots can be retrieved from the *plotdir* directory. These allow finegrained assessment of normalization performance."
- [methods] Metanorm usage: normalize with QCcheck and batch, then plotPCA: "QCcheck = TRUE,      # check whether QCs are representative"
- [readme] README example: plotPCA calls on raw and normalized data labeled by batch: "plotPCA(rawdata[1:5,], type = batch)"
