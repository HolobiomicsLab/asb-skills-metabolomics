---
name: outlier-identification-from-drift-plots
description: Use when QC-based drift correction (e.g. QC-RLSC) leaves suspicious non-monotonic
  intensity excursions in biological (non-QC) samples, and you need to locate and
  remove outlier features before normalization.
when_to_use_negative:
- When the normalization method used is already outlier-robust (tGAM, rGAM, rLOESS)
  — the article shows these methods accommodate or downweight outliers without needing
  manual removal.
- When a large automated multivariate outlier-detection pipeline is available and
  preferred; the article notes manual outlier identification is subjective and labor-intensive
  for large feature counts.
- When the goal is QC-sample RSD filtering rather than drift-plot outlier diagnosis;
  QC-sample RSD filtering is invalidated when QC and biological samples differ.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metanorm
  role: Generates the per-compound intensity-vs-order diagnostic plots for both tGAM
    and QC-RLSC that serve as the visual basis for outlier identification.
  repo: github.com/UGent-LIMET/Metanorm
- name: QC-RLSC
  role: The QC-only drift-correction model whose failures in biological samples expose
    the outlier-contaminated features.
- name: tGAM
  role: Robust GAM normalization used as the reference fit against which QC-RLSC drift
    distortions are judged.
- name: R
  role: Runtime environment for the Metanorm package; requires version >= 4.4.0.
- name: pak
  role: Installs the Metanorm R package from GitHub.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:d1157c9a870cab06925ccc1d7bb4b858988c245d3a22089107e8589a10707fd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QC-RLSC
  - R
  - metanorm
  - pak
  - rLOESS
  - tGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# outlier-identification-from-drift-plots

## Summary

Identifies outlier-contaminated metabolite features in LC-MS metabolomics data by inspecting per-compound pre- vs. post-normalization intensity-vs-run-order drift plots, using QC-RLSC failure patterns in biological samples as the diagnostic signal, and manually removing those outliers so that robust methods like tGAM can be compared on clean data.

## When to use

Use when QC-based drift correction (e.g. QC-RLSC) leaves suspicious non-monotonic intensity excursions in biological (non-QC) samples, and you need to locate and remove outlier features before normalization.

## When NOT to use

- When the normalization method used is already outlier-robust (tGAM, rGAM, rLOESS) — the article shows these methods accommodate or downweight outliers without needing manual removal.
- When a large automated multivariate outlier-detection pipeline is available and preferred; the article notes manual outlier identification is subjective and labor-intensive for large feature counts.
- When the goal is QC-sample RSD filtering rather than drift-plot outlier diagnosis; QC-sample RSD filtering is invalidated when QC and biological samples differ.

## Inputs

- Numerical intensity matrix (rows = compounds, columns = samples) in the metanorm data format
- batch vector assigning each sample run to a batch
- metanorm.qc vector labeling each sample as QC or biological sample
- list of outlier feature indices (optional, for the manual-removal step)

## Outputs

- Per-compound pre- vs. post-normalization intensity-vs-order diagnostic plots in the plotdir directory
- PC score plots before and after normalization labelled by batch
- SERFF replicate concordance values per method (tGAM vs QC-RLSC) per condition (clean, contaminated, manually cleaned)
- Reduced variance in the previously outlier-contaminated features after manual removal

## How to apply

Normalize the numerical intensity matrix (rows = compounds, columns = samples, in metanorm data format with batch and QC/sample-type vectors) twice: once with the default tGAM model (QCcheck = TRUE, batch-wise correction) and once with model = 'QC-RLSC' and QConly = TRUE, writing each method's per-compound diagnostic plots to a dedicated plotdir. Inspect the intensity-vs-run-order plots for features where the QC-RLSC signal drift correction is distorted — specifically, where biological samples show abrupt high or low intensity spikes that the QC-only model cannot accommodate, while the tGAM robust fit remains flat. Confirm the identified features are the root cause by comparing SERFF replicate concordance for tGAM and QC-RLSC across three conditions: clean data, outlier-contaminated data, and data after manual outlier removal prior to QC-RLSC. The verification is that tGAM stays stable across all three conditions while QC-RLSC degrades under outliers and recovers after removal.

## Related tools

- **metanorm** (Generates the per-compound intensity-vs-order diagnostic plots for both tGAM and QC-RLSC that serve as the visual basis for outlier identification.) — github.com/UGent-LIMET/Metanorm
- **QC-RLSC** (The QC-only drift-correction model whose failures in biological samples expose the outlier-contaminated features.)
- **tGAM** (Robust GAM normalization used as the reference fit against which QC-RLSC drift distortions are judged.)
- **R** (Runtime environment for the Metanorm package; requires version >= 4.4.0.)
- **pak** (Installs the Metanorm R package from GitHub.)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "QC-RLSC", type = metanorm.qc, QConly = TRUE, batch = batch, plotdir = "~/Documents/metanormExample2/")
```

## Evaluation signals

- The pre- vs. post-normalization plots for the flagged features show reduced variance after outlier removal compared to before.
- SERFF replicate concordance for tGAM remains roughly constant across clean, contaminated, and manually cleaned conditions, while QC-RLSC concordance drops under contamination and recovers after removal.
- PC score plots labelled by batch show diminished batch effects after normalization, confirming the drift correction itself worked and the residual anomaly was isolated to flagged features.
- The number of features flagged as outliers matches visual inspection of the intensity-vs-order plots where QC-RLSC corrected drift deviates sharply from the tGAM robust fit in biological samples.

## Limitations

- Manual outlier identification is subjective and labor-intensive when the number of compounds is large.
- Automated outlier detection remains non-trivial and is not implemented in the Metanorm package itself.
- The article notes that multivariate outlier detection may improve on the current feature-level treatment, implying this skill's per-feature approach may miss correlated multi-feature outliers.
- Outlier removal must be applied to biological samples; removing outliers only from QC samples does not address drift distortions introduced by non-QC samples under QC-only models.

## Evidence

- [other] Manual identification and removal of outliers improved SERFF's normalization performance: [evidence span withheld — anchor status: not_found]
- [readme] Individual compound plots allow finegrained assessment: "Individual compound pre- vs. post-normalization intensity vs. order plots can be retrieved from the *plotdir* directory. These allow finegrained assessment of normalization performance."
- [intro] Outliers deteriorate existing normalization methods: "we demonstrate that the presence of occasional, disproportionately high or low metabolite intensities (outliers) can deteriorate the performance of existing normalization methods"
- [discussion] Alternative to robust methods is outlier detection and removal: "An alternative to the use of the outlier-robust normalization procedures introduced here could be outlier detection and removal prior to normalization."
- [abstract] Robust methods downweight or accommodate outliers: "We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them."
