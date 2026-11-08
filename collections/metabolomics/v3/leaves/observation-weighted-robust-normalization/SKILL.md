---
name: observation-weighted-robust-normalization
description: Use when LC-MS metabolomics data contain occasional outlier runs (disproportionately
  high/low metabolite intensities) that would bias mean-based or standard QC-based
  drift correction; normalization must stay robust without discarding samples outright.
when_to_use_negative:
- Data are already filtered or pre-cleaned with outlier runs removed, making robust
  weighting unnecessary — simple QC-RLSC or QC-RSC suffices.
- The analysis goal is batch-integration or feature-level filtering rather than per-compound
  signal-drift correction; the skill addresses intensity normalization, not RSD/CV
  filtering on QC samples.
- Downstream evaluation metrics are RSD-of-QC reduction or count of differentially
  abundant metabolites; the article flags these as common benchmarking pitfalls rather
  than evidence the skill was applied correctly.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metanorm
  role: R package implementing the robust normalization methods (tGAM, rGAM, rLOESS)
    and the metanorm() call with weights and QCcheck arguments used by this skill
  repo: github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Dependency used by metanorm to fit the thin-plate-spline GAM underlying tGAM
    by REML
- name: calcWeights
  role: metanorm function that converts model residuals into observation weights,
    downweighting high-residual outlier runs for the robust fit
  repo: github.com/UGent-LIMET/Metanorm
- name: plotPCA
  role: metanorm helper to generate pre- and post-normalization PC score plots labeled
    by batch for diagnostic verification
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:e8704794a6c30877d6265612118c946588cd169606ba93a119d36dba20ca63da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - calcWeights
  - changepoint
  - cowplot
  - fANCOVA
  - ggplot2
  - metanorm
  - mgcv
  - pbapply
  - plotPCA
  - rLOESS
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6206896551724138
n_claims: 29
---

# observation-weighted-robust-normalization

## Summary

Fit the metanorm tGAM/rGAM/rLOESS robust normalization models with per-observation weights derived from residuals (calcWeights), so runs with disproportionately high or low metabolite intensities are downweighted rather than excluded, preserving statistical power for downstream differential abundance testing.

## When to use

Use when LC-MS metabolomics data contain occasional outlier runs (disproportionately high/low metabolite intensities) that would bias mean-based or standard QC-based drift correction; normalization must stay robust without discarding samples outright.

## When NOT to use

- Data are already filtered or pre-cleaned with outlier runs removed, making robust weighting unnecessary — simple QC-RLSC or QC-RSC suffices.
- The analysis goal is batch-integration or feature-level filtering rather than per-compound signal-drift correction; the skill addresses intensity normalization, not RSD/CV filtering on QC samples.
- Downstream evaluation metrics are RSD-of-QC reduction or count of differentially abundant metabolites; the article flags these as common benchmarking pitfalls rather than evidence the skill was applied correctly.

## Inputs

- rawdata: numerical R matrix of unnormalized metabolite intensities (rows = compounds, columns = sample runs)
- batch: vector assigning each sample run to a batch (e.g. "batch1")
- metanorm.qc: vector with sample types per run, marking which runs are "QC" and which are other sample types
- residuals from an initial model fit, used as input to calcWeights() to derive observation weights

## Outputs

- normdat: normalized data matrix of the same dimensions as rawdata
- Pre- vs post-normalization intensity-vs-order diagnostic plots per compound (written to plotdir)
- plotPCA() PC score plots of raw and normalized data labeled by batch

## How to apply

Load the numerical rawdata matrix (rows = compounds, columns = sample runs) and call metanorm() with model = "tGAM" (or "rGAM"/"rLOESS" for speed), type = metanorm.qc, and batch = batch so the mgcv GAM fits signal drift per batch. Compute observation weights from model residuals via calcWeights(residuals) and pass them through the weights argument so high-residual outlier runs are downweighted in the fit rather than removed. Enable QCcheck = TRUE so the function tests whether QC samples are representative of biological samples; if the test fails, re-fitting uses biological samples only. Supply plotdir so pre- vs post-normalization intensity-vs-order diagnostic plots are written per compound, and verify with plotPCA() score plots labeled by batch that drift and batch effects have diminished.

## Related tools

- **metanorm** (R package implementing the robust normalization methods (tGAM, rGAM, rLOESS) and the metanorm() call with weights and QCcheck arguments used by this skill) — github.com/UGent-LIMET/Metanorm
- **mgcv** (Dependency used by metanorm to fit the thin-plate-spline GAM underlying tGAM by REML)
- **calcWeights** (metanorm function that converts model residuals into observation weights, downweighting high-residual outlier runs for the robust fit) — github.com/UGent-LIMET/Metanorm
- **plotPCA** (metanorm helper to generate pre- and post-normalization PC score plots labeled by batch for diagnostic verification) — github.com/UGent-LIMET/Metanorm

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- The returned normdat matrix has the same dimensions as the input rawdata (one normalized intensity per compound per sample run).
- PC score plots labeled by batch show diminished batch effects and signal drift after normalization compared with rawdata.
- Per-compound pre- vs post-normalization intensity-vs-order plots (written to plotdir) show the fitted drift curve following the biological samples rather than being pulled by outlier runs.
- When QCcheck = TRUE, the QC-vs-biological-sample representativeness test is executed and its outcome drives whether QC or sample-based normalization is used.
- Observation weights from calcWeights() are finite, non-negative, and derived from model residuals; outlier runs carry lower weights than typical runs.

## Limitations

- tGAM carries a higher computational cost than rGAM and rLOESS, though normalization remains feasible within minutes for thousands of metabolites and hundreds of samples.
- Outlier detection/weighting is feature-level; multivariate outlier detection could be a future improvement over the current per-metabolite treatment.
- Performance on experimental data was strongest for tGAM, but rLOESS, rGAM, and tGAM were comparable on simulated data; the method was benchmarked against several popular existing strategies, and normalization performance depends on having both QC and biological samples to detect discrepancies.
- Automated outlier identification is non-trivial; manual identification is subjective and labor-intensive for large feature counts.

## Evidence

- [abstract] robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [abstract] downweighting or accommodating outliers via rLOESS/rGAM/tGAM: "We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them"
- [intro] outliers reduce statistical power in differential abundance testing: "We subsequently show that such outliers go on to reduce statistical power in downstream analyses, such as differential metabolite abundance testing"
- [readme] metanorm() call with model, type, QCcheck, batch, plotdir: "normdat <- metanorm(rawdata[1:5,],       # numerical data matrix to normalize
	                model = "tGAM",      # default tGAM method
                    type = metanorm.qc,  # vector with sample"
- [methods] batchwise fitting with separate additive model per batch: [evidence span withheld — anchor status: not_found]
