---
name: r-package-normalization-pipeline-execution
description: Use when an unnormalized LC-MS metabolomics intensity matrix (rows =
  compounds, columns = samples) must be drift/batch-corrected via QC-based normalization,
  with diagnostic plots to verify performance.
when_to_use_negative:
- Data has already been drift/batch-normalized or the task is downstream differential
  abundance testing with multiple testing correction — this skill only performs normalization
- QC samples are not representative of biological samples (metanorm's QCcheck flags
  discrepancies) — QC-based normalization is then inappropriate
- No QC samples exist in the run order — all methods here are QC sample-based normalization
  strategies
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metanorm
  role: R function invoked to normalize the rawdata matrix under a chosen model (tGAM/rGAM/rLOESS/QC-RLSC)
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Package providing metanorm() and plotPCA(), integrating robust normalization
    with visualization for performance verification
  repo: github.com/UGent-LIMET/Metanorm
- name: pak
  role: Installer used to fetch the package from GitHub (pak::pak("UGent-LIMET/Metanorm"))
- name: R
  role: Runtime environment, version >= 4.4.0 required
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:41e604d8fd31cbadd080089450580d42b163fea715a5b6703ec215da2d885fd1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - R
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# r-package-normalization-pipeline-execution

## Summary

Executes the Metanorm R package (R >= 4.4.0) normalization pipeline for LC-MS metabolomics drift/batch correction, calling the metanorm function with a chosen model (tGAM, rGAM, rLOESS, QC-RLSC) plus diagnostic visualization (plotPCA, per-compound pre/post plots).

## When to use

Use when an unnormalized LC-MS metabolomics intensity matrix (rows = compounds, columns = samples) must be drift/batch-corrected via QC-based normalization, with diagnostic plots to verify performance.

## When NOT to use

- Data has already been drift/batch-normalized or the task is downstream differential abundance testing with multiple testing correction — this skill only performs normalization
- QC samples are not representative of biological samples (metanorm's QCcheck flags discrepancies) — QC-based normalization is then inappropriate
- No QC samples exist in the run order — all methods here are QC sample-based normalization strategies

## Inputs

- rawdata: numerical matrix of unnormalized log-intensities (rows = compounds, columns = samples)
- type: vector labeling each sample run as "QC" or other sample type (e.g. metanorm.qc)
- batch: vector assigning each sample run to a batch (e.g. "batch1")
- model choice string: "tGAM", "rGAM", "rLOESS", or "QC-RLSC"

## Outputs

- normdat: normalized numerical matrix of the same dimensions as rawdata
- PC score plots of data before and after normalization, labeled by batch
- Per-compound pre- vs. post-normalization intensity vs. run-order plots written to plotdir

## How to apply

Install R >= 4.4.0 and the package via pak::pak("UGent-LIMET/Metanorm"), then load rawdata (numerical matrix, compounds x samples), batch, and metanorm.qc sample-type vectors. Call metanorm(rawdata, model = "tGAM" (default/recommended), type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = ...) — set QCcheck = TRUE so metanorm verifies QCs are representative of biological samples, and supply batch to normalize per batch. For QC-RLSC specifically, pass QConly = TRUE since it is prescribed to use QC samples only. Judge success via the generated PC score plots (batch effects should diminish pre- vs. post-normalization) and by inspecting a decent number of the per-compound intensity-vs-run-order plots written to plotdir, which reveal residual drift in non-QC samples.

## Related tools

- **metanorm** (R function invoked to normalize the rawdata matrix under a chosen model (tGAM/rGAM/rLOESS/QC-RLSC)) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Package providing metanorm() and plotPCA(), integrating robust normalization with visualization for performance verification) — github.com/UGent-LIMET/Metanorm
- **pak** (Installer used to fetch the package from GitHub (pak::pak("UGent-LIMET/Metanorm")))
- **R** (Runtime environment, version >= 4.4.0 required)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- metanorm returns a finite normalized matrix (normdat) with the same compounds x samples dimensions as rawdata
- QCcheck = TRUE passes, i.e. QCs are confirmed representative of the biological samples
- plotPCA of normdat labeled by batch shows diminished batch effects relative to plotPCA of rawdata
- Per-compound pre- vs. post-normalization intensity-vs-order plots show no residual drift in biological (non-QC) samples
- Re-running with the same inputs reproduces the same normalized values

## Limitations

- tGAM has a higher computational cost than rGAM and rLOESS, though it completes within minutes for thousands of metabolites and hundreds of samples
- QC-RLSC uses QC samples only (QConly = TRUE) and can deteriorate in the presence of outliers; inspect per-compound plots, since PCA plots alone may look clean while drift remains in non-QC samples
- Requires R >= 4.4.0; the exact installed Metanorm package version is not pinned in the documentation

## Evidence

- [abstract] Implementations of these methods are gathered in the Metanorm R package, integrating robust normalization with visualization for performance verification: "Implementations of these methods are gathered in the Metanorm R package, integrating robust normalization with visualization for performance verification"
- [methods] Install command: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [methods] metanorm call with QCcheck: "QCcheck = TRUE,      # check whether QCs are representative"
- [methods] Example dataset loading: "load(system.file("extdata", "example.RData", package = "metanorm"))"
- [readme] Normalization call in README example: "normdat <- metanorm(rawdata[1:5,],       # numerical data matrix to normalize
	                model = "tGAM",      # default tGAM method"
- [readme] Inspect pre/post plots: "These allow finegrained assessment of normalization performance. It is highly recommended to look at a decent number of these plots"
- [discussion] High-throughput implementation: "their implementation in the Metanorm R package facilitates robust, high-throughput, and efficient normalization"
