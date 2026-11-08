---
name: qc-sample-based-signal-drift-modeling
description: Use when an untargeted LC-MS metabolomics run-order matrix (rawdata with
  rows=compounds, columns=samples) shows intensity drift/batch effects and interleaved
  QC runs are available to fit the drift correction.
when_to_use_negative:
- The dataset contains no interleaved QC runs or QCs are not representative of biological
  samples — QCcheck = TRUE exists precisely to flag discrepancies between QC and biological
  samples, and the article warns that such differences may also invalidate QC-RSD-based
  filtering.
- The input matrix is already normalized or the goal is multiple-testing-corrected
  downstream inference, which the article explicitly leaves out of scope.
- Outliers can be reliably detected and removed beforehand as an alternative strategy
  — the article treats robust normalization and pre-normalization outlier detection/removal
  as alternative routes.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: metanorm
  role: Core function called on rawdata with model, type, QCcheck, batch and plotdir
    arguments to fit the QC-sample-based drift model and return the normalized matrix
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Package providing metanorm() and plotPCA(), integrating robust normalization
    with visualization for performance verification
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Default and tentatively recommended drift-fitting model (generalized additive
    model with flexible non-linear modeling, hypothesis testing, and observation weighting)
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Faster robust alternative drift-fitting model based on additive models
  repo: github.com/UGent-LIMET/Metanorm
- name: rLOESS
  role: Faster robust alternative drift-fitting model based on cross-validated robust
    LOESS
  repo: github.com/UGent-LIMET/Metanorm
- name: QC-RLSC
  role: Existing non-robust comparator used with QConly = TRUE so only QC samples
    drive the fitted drift
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >= 4.4.0 required) for the Metanorm package
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:ccfbfca5b1d61c290340a7064eeb9b3b9c52274a8b1d891cc9cd1b29b73e0869
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - QC-RLSC
  - R
  - ggplot2
  - metanorm
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# qc-sample-based-signal-drift-modeling

## Summary

Applies the Metanorm R package's QC-sample-based signal drift models (tGAM default, plus rGAM, rLOESS, QC-RLSC) to an unnormalized LC-MS metabolomics intensity matrix (rawdata), optionally batch-wise, and produces a normalized matrix plus pre/post-normalization diagnostic plots.

## When to use

Use when an untargeted LC-MS metabolomics run-order matrix (rawdata with rows=compounds, columns=samples) shows intensity drift/batch effects and interleaved QC runs are available to fit the drift correction.

## When NOT to use

- The dataset contains no interleaved QC runs or QCs are not representative of biological samples — QCcheck = TRUE exists precisely to flag discrepancies between QC and biological samples, and the article warns that such differences may also invalidate QC-RSD-based filtering.
- The input matrix is already normalized or the goal is multiple-testing-corrected downstream inference, which the article explicitly leaves out of scope.
- Outliers can be reliably detected and removed beforehand as an alternative strategy — the article treats robust normalization and pre-normalization outlier detection/removal as alternative routes.

## Inputs

- rawdata: numerical matrix of unnormalized metabolite intensities (rows = compounds, columns = sample runs), e.g. the 618 runs x 8913 features FAME saliva matrix
- type: vector labeling each run as 'QC' or other sample type (metanorm.qc)
- batch: vector giving the batch of each sample run, for batch-wise normalization
- model choice: 'tGAM' (default), 'rGAM', 'rLOESS', or 'QC-RLSC'

## Outputs

- normdat: normalized numerical matrix of the same dimensions as rawdata
- Per-compound pre- vs post-normalization intensity vs run-order plots written to plotdir
- PC score plots of data before and after normalization labeled by batch via plotPCA()

## How to apply

Install the Metanorm package (R >= 4.4.0) via pak::pak("UGent-LIMET/Metanorm") and load rawdata (compounds x runs), batch, and the type vector labeling each run 'QC' or other. Call metanorm() with model = 'tGAM' (default, recommended for strongest performance), rGAM, or rLOESS — these robust models downweight or accommodate outliers — supplying type, QCcheck = TRUE (to check whether QCs are representative of biological samples), and batch for batch-wise normalization; write diagnostic plots to a plotdir. For the legacy non-robust comparator, call metanorm(model = 'QC-RLSC', QConly = TRUE) so that, as prescribed, only QC samples drive the fitted drift. Judge the result with plotPCA(rawdata, type = batch) vs plotPCA(normdat, type = batch) for batch-effect removal and by inspecting individual compound pre- vs post-normalization intensity-vs-order plots (e.g. plot 4 in the README example) to verify drift is corrected in biological, not only QC, samples.

## Related tools

- **metanorm** (Core function called on rawdata with model, type, QCcheck, batch and plotdir arguments to fit the QC-sample-based drift model and return the normalized matrix) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Package providing metanorm() and plotPCA(), integrating robust normalization with visualization for performance verification) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Default and tentatively recommended drift-fitting model (generalized additive model with flexible non-linear modeling, hypothesis testing, and observation weighting)) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Faster robust alternative drift-fitting model based on additive models) — github.com/UGent-LIMET/Metanorm
- **rLOESS** (Faster robust alternative drift-fitting model based on cross-validated robust LOESS) — github.com/UGent-LIMET/Metanorm
- **QC-RLSC** (Existing non-robust comparator used with QConly = TRUE so only QC samples drive the fitted drift) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >= 4.4.0 required) for the Metanorm package)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- QCcheck = TRUE passes, i.e. no discrepancy is flagged between QC and biological samples
- plotPCA before/after normalization labeled by batch shows diminished batch effects in the PC score plot
- Inspection of individual compound pre- vs post-normalization intensity-vs-order plots (e.g. plot 4 of the README example) shows signal drift corrected in biological samples, not only in QCs
- In the FAME reproduction scenario, differential-abundance overlaps between the robust models (rLOESS, rGAM, tGAM) fall in the 79-87% band while QC-RLSC overlaps sit in the weaker 55-59% band
- Under both GCV and LOOCV smoother-span selection, normalized matrices yield highly comparable DAM counts and p-value orderings

## Limitations

- tGAM has a higher computational cost than rGAM and rLOESS, although normalization completes within minutes on current hardware for thousands of metabolites and hundreds of samples
- QC-RLSC, used with QConly = TRUE, can show degraded agreement with the robust methods (55-59% DAM overlap vs 79-87%) and its drift fit may look fine in PC score plots while individual-compound plots reveal drift problems in biological samples
- Outliers in the intensity data can deteriorate the performance of existing (non-robust) QC-based normalization methods and reduce statistical power in downstream differential abundance testing
- Automated outlier detection is non-trivial, and manual outlier identification is subjective and labor-intensive for large feature counts

## Evidence

- [abstract] We introduce rLOESS, rGAM and tGAM, three robust normalization methods: "We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them."
- [abstract] tGAM recommended as default: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [methods] QCcheck argument: "QCcheck = TRUE,      # check whether QCs are representative"
- [discussion] PCA score plots for QC: "The pre- vs. post-normalization and PCA score plots readily generated by the Metanorm package support an efficient identification of subpar data quality or aberrant normalization"
- [readme] QC-RLSC uses QC samples only: "you can now normalize with QC-RLSC, using, as prescribed, QC samples for normalization (*QConly* argument):"
- [readme] R version requirement: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
