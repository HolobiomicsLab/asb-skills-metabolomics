---
name: robust-additive-model-fitting
description: Use when a raw metabolomics intensity matrix (compounds x samples, run-ordered)
  contains QC injections and shows signal drift, batch effects, or occasional extreme
  intensities that would bias QC-sample-based smoothers like QC-RLSC or QC-RSC.
when_to_use_negative:
- When you intend to use QC samples only (QConly) without checking that QC samples
  are representative of biological samples — Metanorm's QCcheck is specifically recommended
  to catch such discrepancies
- When the goal is multivariate outlier detection across metabolites — the article
  notes treatment is currently feature-level and multivariate outlier detection is
  not implemented
- When a simpler, faster fit suffices and data are clean — rGAM and rLOESS are faster
  than tGAM, so tGAM's extra robustness is unnecessary on outlier-free data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3572
- http://edamontology.org/topic_2269
tools:
- name: Metanorm R package
  role: Implements the robust additive-model and generalized-additive-model normalization
    methods (rGAM, tGAM) with parallelized high-throughput processing and diagnostic
    visualization
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (>= 4.4.0) in which Metanorm is installed via pak and
    the metanorm()/plotPCA() functions are called
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:697734e8dbcee708ffb7dfbcd06916733c1238bd53deb8d11d3dc52b410f92a2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3572
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# robust-additive-model-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Outlier-robust normalization of metabolomics feature intensities by fitting additive models (rGAM) or generalized additive models with observation weighting (tGAM) over run order, using QC and biological samples jointly to recover the underlying signal drift.

## When to use

Use when a raw metabolomics intensity matrix (compounds x samples, run-ordered) contains QC injections and shows signal drift, batch effects, or occasional extreme intensities that would bias QC-sample-based smoothers like QC-RLSC or QC-RSC.

## When NOT to use

- When you intend to use QC samples only (QConly) without checking that QC samples are representative of biological samples — Metanorm's QCcheck is specifically recommended to catch such discrepancies
- When the goal is multivariate outlier detection across metabolites — the article notes treatment is currently feature-level and multivariate outlier detection is not implemented
- When a simpler, faster fit suffices and data are clean — rGAM and rLOESS are faster than tGAM, so tGAM's extra robustness is unnecessary on outlier-free data

## Inputs

- Unnormalized metabolomics intensity matrix (numerical matrix, rows = compounds, columns = samples)
- Vector of sample types per run labeling QC vs. non-QC samples (metanorm.qc)
- Optional vector of batch labels per sample run (batch)

## Outputs

- Normalized intensity matrix (same dimensions as input) with drift and batch effects reduced
- Per-compound pre- vs. post-normalization intensity vs. run-order plots written to a plot directory
- PCA score plots of data before and after normalization, labeled by batch or sample type

## How to apply

Load the Metanorm R package (R >= 4.4.0) and prepare three objects: the numerical matrix rawdata (rows = compounds, columns = samples), a batch vector, and a type vector (metanorm.qc) labeling each run as 'QC' or other. Call metanorm() per feature with model = 'tGAM' (default, strongest performer across experimental data sets) or 'rGAM'/'rLOESS' when speed matters, keeping QCcheck = TRUE so Metanorm can flag discrepancies between QC and biological samples; the (G)AM framework enables hypothesis testing and observation weighting for data-driven joint or selective use of QC and biological samples. If batch structure exists, pass the batch argument so normalization is performed within batch. Diagnose with the per-compound pre- vs. post-normalization intensity-vs.-order plots written to plotdir and pre/post PCA score plots labeled by batch; do not judge fit quality by reduction of QC-sample RSD alone, as the paper flags this as a benchmarking pitfall.

## Related tools

- **Metanorm R package** (Implements the robust additive-model and generalized-additive-model normalization methods (rGAM, tGAM) with parallelized high-throughput processing and diagnostic visualization) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (>= 4.4.0) in which Metanorm is installed via pak and the metanorm()/plotPCA() functions are called) — https://cloud.r-project.org/index.html

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- Per-compound pre- vs. post-normalization intensity-vs.-order plots show the fitted drift smoothly tracking the non-outlier intensities rather than being pulled by outliers
- PCA score plots after normalization show diminished batch effects compared with the raw data (README demonstrates this check labeled by batch)
- Replicate concordance improves after normalization, matching the paper's finding that robust methods 'improved replicate concordance and reduced drift and batch effects'
- QCcheck warnings are reviewed: unexpected differences between QC and biological samples receive analyst scrutiny rather than silent acceptance
- Success is not claimed solely from reduced QC-sample relative standard deviation, since the paper identifies RSD reduction as a benchmarking pitfall

## Limitations

- Outlier treatment is feature-level; multivariate outlier detection is not implemented
- Automated outlier detection and removal prior to normalization is a potential alternative but nontrivial and not provided by the package
- tGAM is more robust but slower than rGAM and rLOESS, which may matter for very large matrices despite pleasingly parallel processing
- Reliability of such normalization for small data sets has not been broadly investigated
- RSD-based filtering of features may be invalidated if QC and biological samples differ unexpectedly

## Evidence

- [readme] README: recommend QC plus biological samples and QC-vs-biological discrepancy checking: [evidence span withheld — anchor status: external_doc]
- [readme] README: per-compound pre/post normalization plots from plotdir for fine-grained assessment: [evidence span withheld — anchor status: external_doc]
