---
name: qc-based-signal-drift-normalization
description: Use when unnormalized LC-MS metabolomics data (compound x sample intensity
  matrix) show signal drift, batch effects, or outliers that degrade replicate concordance
  and downstream differential-abundance power.
when_to_use_negative:
- QC samples are not representative of the biological samples (QCcheck fails) — QC-based
  drift correction will be invalid
- You need to remove outliers manually first — the article notes automated outlier
  detection is non-trivial and manual identification is subjective and labor-intensive
  for large feature counts
- Downstream multiple-testing-corrected inference is the goal itself — normalization
  is a prerequisite, not a substitute
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metanorm (Metanorm R package)
  role: Implements tGAM, rGAM, rLOESS, and QC-RLSC normalization models for the intensity
    matrix; also provides plotPCA for batch-effect visualization
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Generalized additive model–based QC normalization with flexible non-linear
    modeling, hypothesis testing, and observation weighting; recommended default for
    strongest performance across experimental datasets
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Robust GAM-based QC normalization; faster alternative to tGAM with slightly
    weaker outlier resistance
  repo: github.com/UGent-LIMET/Metanorm
- name: rLOESS
  role: Cross-validated robust LOESS QC normalization; fastest of the robust methods
  repo: github.com/UGent-LIMET/Metanorm
- name: QC-RLSC
  role: Baseline QC-only normalization used for comparison; vulnerable to outliers
    in biological samples
  repo: github.com/UGent-LIMET/Metanorm
- name: plotPCA
  role: Generates pre- vs. post-normalization PC score plots labeled by batch to judge
    whether batch effects diminished
  repo: github.com/UGent-LIMET/Metanorm
- name: R (>= 4.4.0)
  role: Runtime environment required to install and run the Metanorm package
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:e3f1377bdec4471020859153afde91302b1d2f98d32f5f575742463b13f7586a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm
  - QC-RLSC
  - R
  - R (>= 4.4.0)
  - metanorm
  - metanorm (Metanorm R package)
  - pak
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- qc-sample-based-signal-drift-normalization
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# qc-based-signal-drift-normalization

## Summary

Normalizes LC-MS metabolomics intensity matrices against signal drift and batch effects using QC-sample-based models (tGAM, rGAM, rLOESS, QC-RLSC) implemented in the Metanorm R package, with tGAM as the recommended default because of its outlier robustness.

## When to use

Use when unnormalized LC-MS metabolomics data (compound x sample intensity matrix) show signal drift, batch effects, or outliers that degrade replicate concordance and downstream differential-abundance power.

## When NOT to use

- QC samples are not representative of the biological samples (QCcheck fails) — QC-based drift correction will be invalid
- You need to remove outliers manually first — the article notes automated outlier detection is non-trivial and manual identification is subjective and labor-intensive for large feature counts
- Downstream multiple-testing-corrected inference is the goal itself — normalization is a prerequisite, not a substitute

## Inputs

- Numerical intensity matrix (rows = compounds, columns = samples), e.g. from a BioHEART in-replicate LC-MS dataset or the metanorm example.RData
- Batch vector assigning each sample run to an acquisition batch (e.g. 'batch1')
- Sample-type vector marking each run as 'QC' or biological sample (metanorm.qc)

## Outputs

- Normalized intensity matrix of the same dimensions as the input
- Per-compound pre- vs. post-normalization intensity-vs-order diagnostic plots written to plotdir
- Pre- and post-normalization PC score plots labeled by batch (via plotPCA)

## How to apply

Load the data in the metanorm format: a numerical intensity matrix (rows = compounds, columns = samples), a batch vector, and a sample-type vector marking QC vs. biological samples. Call metanorm() with model='tGAM' (the recommended default), type=sample-type vector, QCcheck=TRUE (to verify QCs are representative of biological samples before trusting them as drift anchors), and batch=batch vector for batch-wise correction; write per-compound intensity-vs-order plots to a plotdir. In parallel, run model='QC-RLSC' with QConly=TRUE for comparison, since QC-RLSC as prescribed uses only QC samples and is vulnerable to outliers in non-QC samples. Judge success via pre- vs. post-normalization plotPCA labeled by batch (batch effects should diminish) plus manual inspection of individual compound drift plots; an RSD reduction in QC samples alone is an explicitly discouraged proxy. If drift remains, consider robust alternatives rGAM or rLOESS (faster but slightly weaker) or manual outlier removal before normalization.

## Related tools

- **metanorm (Metanorm R package)** (Implements tGAM, rGAM, rLOESS, and QC-RLSC normalization models for the intensity matrix; also provides plotPCA for batch-effect visualization) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Generalized additive model–based QC normalization with flexible non-linear modeling, hypothesis testing, and observation weighting; recommended default for strongest performance across experimental datasets) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Robust GAM-based QC normalization; faster alternative to tGAM with slightly weaker outlier resistance) — github.com/UGent-LIMET/Metanorm
- **rLOESS** (Cross-validated robust LOESS QC normalization; fastest of the robust methods) — github.com/UGent-LIMET/Metanorm
- **QC-RLSC** (Baseline QC-only normalization used for comparison; vulnerable to outliers in biological samples) — github.com/UGent-LIMET/Metanorm
- **plotPCA** (Generates pre- vs. post-normalization PC score plots labeled by batch to judge whether batch effects diminished) — github.com/UGent-LIMET/Metanorm
- **R (>= 4.4.0)** (Runtime environment required to install and run the Metanorm package) — https://cloud.r-project.org/index.html

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Pre- vs. post-normalization PC score plot labeled by batch shows diminished batch clustering after metanorm
- Per-compound intensity-vs-order diagnostic plots show drift correction without distortion in biological (non-QC) samples
- SERFF replicate concordance improves (or remains high) after normalization; tGAM is stable across clean vs. outlier-contaminated conditions
- QCcheck passes (QCs are representative of biological samples) before trusting QC-based models
- No RSD-only shortcut: performance is judged by drift/batch plots and replicate concordance, not just QC RSD reduction

## Limitations

- tGAM is computationally more expensive than rGAM and rLOESS, though it completes within minutes for thousands of metabolites and hundreds of samples
- QC-RLSC uses only QC samples and can be degraded by outliers in biological samples, as shown on the BioHEART in-replicate dataset
- Automated outlier detection remains non-trivial; manual outlier identification is subjective and labor-intensive for large feature counts
- Metanorm's GAM-based procedures adaptively revert to sample-based normalization when QC samples are unrepresentative, so results depend on QC representativeness

## Evidence

- [abstract] tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice.: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [methods] QCcheck parameter controls whether QC samples are validated as representative before normalization.: "QCcheck = TRUE,      # check whether QCs are representative"
- [readme] plotPCA provides pre/post batch-effect verification.: "make a PC score plot of the data before and after normalization, label by batch"
- [abstract] Robust methods were evaluated against existing strategies for replicate concordance, drift, and batch effects.: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [intro] Outliers degrade existing QC-based normalization methods.: "the presence of occasional, disproportionately high or low metabolite intensities (outliers) can deteriorate the performance of existing normalization methods"
