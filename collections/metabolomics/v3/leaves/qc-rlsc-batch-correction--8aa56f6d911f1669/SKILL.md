---
name: qc-rlsc-batch-correction
description: Use when an unnormalized LC-MS metabolomics intensity matrix with QC-sample
  and batch annotations must be drift-corrected with QC-RLSC, or when benchmarking
  it against robust methods on DAM overlap and replicate concordance.
when_to_use_negative:
- 'When occasional disproportionately high or low intensities (outliers) are present:
  the article shows outliers deteriorate existing QC-based methods, so tGAM/rGAM/rLOESS
  are preferred'
- When the data have no QC samples or QCs are not representative of biological samples
  — QC-RLSC is QC-only by prescription and QCcheck = TRUE will flag discrepancies
- When the goal is a single recommended default normalization rather than reproducing
  the QC-RLSC baseline for comparison (tGAM is the tentatively recommended default)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3275
- http://edamontology.org/topic_2269
tools:
- name: metanorm
  role: R package providing metanorm(model = "QC-RLSC", QConly = TRUE, batch = ...,
    QCcheck = TRUE) for batchwise QC-RLSC correction and plotPCA/diagnostic plots
  repo: github.com/UGent-LIMET/Metanorm
- name: pak
  role: Installs the Metanorm package from GitHub in R >= 4.4.0
- name: R
  role: Runtime environment (>= 4.4.0) in which QC-RLSC normalization and downstream
    DAM testing are executed
- name: tGAM
  role: Robust reference method (Metanorm default) against which QC-RLSC's drift correction
    and DAM overlap are benchmarked
  repo: github.com/UGent-LIMET/Metanorm
- name: ggplot2
  role: Rendering of the Fig. 5 cross-method Spearman-correlation/Euclidean-distance
    heatmap comparing QC-RLSC with other strategies
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:71f7825398d9f11fb4ff2e8b4d17f875b53d15481b26732c593e3f44a8d73594
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QC-RLSC
  - R
  - cowplot
  - fANCOVA
  - ggplot2
  - metanorm
  - mgcv
  - pak
  - pbapply
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3275
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# qc-rlsc-batch-correction

## Summary

Applies QC-RLSC quality-control-sample-based signal-drift correction to LC-MS metabolomics intensity matrices via the Metanorm R package, using QC samples only and batchwise normalization, and diagnoses its behavior against robust alternatives (tGAM, rGAM, rLOESS).

## When to use

Use when an unnormalized LC-MS metabolomics intensity matrix with QC-sample and batch annotations must be drift-corrected with QC-RLSC, or when benchmarking it against robust methods on DAM overlap and replicate concordance.

## When NOT to use

- When occasional disproportionately high or low intensities (outliers) are present: the article shows outliers deteriorate existing QC-based methods, so tGAM/rGAM/rLOESS are preferred
- When the data have no QC samples or QCs are not representative of biological samples — QC-RLSC is QC-only by prescription and QCcheck = TRUE will flag discrepancies
- When the goal is a single recommended default normalization rather than reproducing the QC-RLSC baseline for comparison (tGAM is the tentatively recommended default)

## Inputs

- Unnormalized LC-MS metabolomics intensity matrix (rows = compounds, columns = sample runs), e.g. rawdata in example.RData
- Vector of sample types per run flagging QC vs. biological samples (metanorm.qc)
- Vector of batch assignments per sample run (batch)
- Optionally, a replicate-annotation vector for concordance evaluation and a phenotype/group vector for DAM testing

## Outputs

- QC-RLSC-normalized intensity matrix (normdat2)
- Pre/post-normalization PCA score plots colored by batch (plotPCA)
- Per-compound pre- vs. post-normalization intensity-vs-order diagnostic plots written to plotdir
- Differential abundance testing results and DAM overlap percentages against reference strategies (e.g. DAM_overlap_summary.csv, Figure5_heatmap.png)

## How to apply

Load the unnormalized intensity matrix (e.g., BioHEART 1359 plasma runs × 53 metabolites, or ENVIRONAGE 393 runs × 269 metabolites), the batch vector, and the QC/sample-type vector into R (>= 4.4.0) and install Metanorm via pak::pak("UGent-LIMET/Metanorm"). Call metanorm() with model = "QC-RLSC", QConly = TRUE (as prescribed, QC-RLSC uses QC samples only), type = the QC annotation vector, batch = the batch vector for batchwise normalization, QCcheck = TRUE to verify QCs are representative, and a plotdir for per-compound diagnostics. Inspect the pre- vs. post-normalization intensity-vs-order plots, especially in biological (non-QC) samples, since apparent batch-effect removal in PCA score plots can mask poor drift modeling; the README's example QC-RLSC run flags exactly this on compound 4. Benchmark against robust methods (tGAM default, also rGAM and rLOESS) via between-replicate MAD reduction and differential-abundance-metabolite (DAM) overlap: expected QC-RLSC replicate-concordance improvement is only 18–22% vs. 49–52% for robust methods, and DAM overlap with robust methods is 47–51% on ENVIRONAGE, so use these ranges to judge whether QC-RLSC is appropriate for the dataset.

## Related tools

- **metanorm** (R package providing metanorm(model = "QC-RLSC", QConly = TRUE, batch = ..., QCcheck = TRUE) for batchwise QC-RLSC correction and plotPCA/diagnostic plots) — github.com/UGent-LIMET/Metanorm
- **pak** (Installs the Metanorm package from GitHub in R >= 4.4.0)
- **R** (Runtime environment (>= 4.4.0) in which QC-RLSC normalization and downstream DAM testing are executed)
- **tGAM** (Robust reference method (Metanorm default) against which QC-RLSC's drift correction and DAM overlap are benchmarked) — github.com/UGent-LIMET/Metanorm
- **ggplot2** (Rendering of the Fig. 5 cross-method Spearman-correlation/Euclidean-distance heatmap comparing QC-RLSC with other strategies)

## Examples

```
normdat2 <- metanorm(rawdata[1:5,], model = "QC-RLSC", type = metanorm.qc, QConly = TRUE, batch = batch, QCcheck = TRUE, plotdir = "~/metanormExample2/")
```

## Evaluation signals

- Normalized matrix returned by metanorm() with model = "QC-RLSC" and QConly = TRUE converges without warnings and QCcheck = TRUE reports QCs representative of biological samples
- Between-replicate MAD reduction relative to unnormalized data falls in the reported 18–22% range (BioHEART), below the 49–52% achieved by robust methods
- DAM overlap with robust-method DAM sets falls in the reported 47–51% range (ENVIRONAGE Fig. 5b-5c), the weakest among tested strategies
- Pairwise Spearman correlation / Euclidean distance heatmap (Fig. 5) shows QC-RLSC clustering apart from SERFF, TIGER and robust methods
- Per-compound pre/post-normalization plots in plotdir show plausible drift curves in biological samples (README flags compound 4 as a QC-RLSC failure example)

## Limitations

- Outliers disproportionately deteriorate QC-RLSC and other existing QC-based normalization methods, reducing statistical power in downstream DAM testing
- QC-RLSC is prescribed to use QC samples only (QConly = TRUE), and the README shows this can miss drift behavior in biological samples that PCA score plots do not reveal
- Filtering on QC-sample RSD can be invalidated by differences between QC and biological samples, affecting downstream inference after QC-RLSC correction
- tGAM carries higher computational cost than alternatives, but QC-RLSC's poorer concordance/DAM-overlap performance means it is not the recommended default

## Evidence

- [readme] QC-RLSC uses QC samples only and is called through metanorm with batch normalization: "you can now normalize with QC-RLSC, using, as prescribed, QC samples for normalization (*QConly* argument)"
- [methods] QCcheck verifies QCs are representative before normalization: "QCcheck = TRUE,      # check whether QCs are representative"
- [intro] Outliers deteriorate existing QC-based normalization methods such as QC-RLSC: "we demonstrate that the presence of occasional, disproportionately high or low metabolite intensities (outliers) can deteriorate the performance of existing normalization methods"
- [abstract] Robust methods outperform existing strategies including QC-RLSC on replicate concordance and batch effects: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [readme] Per-compound pre/post plots are essential QC since PCA alone can hide QC-RLSC misfit in biological samples: "you might want to check the pre- versus post-normalization plot number 4, for example! What does the signal drift look like in the biological (= non-QC) samples?"
- [methods] Metanorm requires R >= 4.4.0: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
