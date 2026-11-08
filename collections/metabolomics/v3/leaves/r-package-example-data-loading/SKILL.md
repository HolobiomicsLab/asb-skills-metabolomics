---
name: r-package-example-data-loading
description: Use when reproducing the RGCxGC vignette/MPCA workflow on the MTBLS579
  MetaboLights dataset and you need the preprocessed 2D-TIC chromatograms already
  packaged as MTBLS.rda instead of re-importing raw NetCDF chromatograms.
when_to_use_negative:
- You have your own raw chromatograms in NetCDF format — use the package's read_chrom
  import-and-fold workflow instead of the example data
- You need supervised modeling on unfolded chromatograms (unfold_chrom + mixOmics)
  with data other than the MTBLS579 reference set
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: RGCxGC
  role: R package whose environment/import conventions the MTBLS.rda chromatogram
    object must be loaded into; provides m_prcomp, scores, and plot_loading consumed
    downstream of this loading step
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Runtime in which the .rda example data are loaded (load()/data()) and passed
    to RGCxGC functions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:8dc1045d1dfc20844596f9967dbca3f5f618809c63e4fdfe56ca235e53c28349
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/DanielQuiroz97/RGCxGC
  repo_url: https://github.com/DanielQuiroz97/RGCxGC
  tools:
  - R
  - RGCxGC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# r-package-example-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load the bundled MTBLS579 example two-dimensional chromatogram data (MTBLS.rda) shipped with the RGCxGC R package so that downstream steps (preprocessing checks, m_prcomp MPCA) run on the paper's reference GCxGC-MS dataset rather than user-supplied raw NetCDF files.

## When to use

Use when reproducing the RGCxGC vignette/MPCA workflow on the MTBLS579 MetaboLights dataset and you need the preprocessed 2D-TIC chromatograms already packaged as MTBLS.rda instead of re-importing raw NetCDF chromatograms.

## When NOT to use

- You have your own raw chromatograms in NetCDF format — use the package's read_chrom import-and-fold workflow instead of the example data
- You need supervised modeling on unfolded chromatograms (unfold_chrom + mixOmics) with data other than the MTBLS579 reference set

## Inputs

- MTBLS.rda file (bundled/preprocessed MTBLS579 GCxGC-MS 2D-TIC chromatogram data) from github.com/DanielQuiroz97/MTBLS579
- RGCxGC R package installed from CRAN or GitHub master branch

## Outputs

- In-memory RGCxGC two-dimensional chromatogram (2D-TIC) object of the MTBLS579 dataset, ready for m_prcomp / multivariate analysis

## How to apply

Obtain MTBLS.rda from the DanielQuiroz97/MTBLS579 repository linked in the RGCxGC introduction vignette, then load it into the R session alongside the RGCxGC package so the object becomes an RGCxGC two-dimensional chromatogram (2D-TIC). This bypasses read_chrom-based import of NetCDF files and folding, because the packaged data are already preprocessed (smoothing, asymmetric least squares baseline correction, 2D-COW peak alignment per the package pipeline). The loaded chromatogram object is then passed directly to m_prcomp (e.g., center = TRUE, scale = FALSE, as in the vignette) to fit the multiway PCA over the three-dimensional 2D-TIC data. Confirm the load succeeded by inspecting the chromatogram object's dimensions (samples × first-dimension × second-dimension retention-time slices) before fitting multivariate models.

## Related tools

- **RGCxGC** (R package whose environment/import conventions the MTBLS.rda chromatogram object must be loaded into; provides m_prcomp, scores, and plot_loading consumed downstream of this loading step) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Runtime in which the .rda example data are loaded (load()/data()) and passed to RGCxGC functions)

## Examples

```
Rscript -e 'library(RGCxGC); load("MTBLS.rda"); mpca <- m_prcomp(MTBLS, center = TRUE, scale = FALSE)'
```

## Evaluation signals

- The MTBLS.rda load completes without error and the resulting object is recognized as an RGCxGC two-dimensional chromatogram (print/plot of the 2D-TIC renders via the filled.contour-based plotting)
- Dimensions of the loaded chromatogram match the MTBLS579 sample set and retention-time grid expected by the vignette before calling m_prcomp
- Subsequent m_prcomp(MTBLS, center = TRUE, scale = FALSE) fits successfully and scores(mpca) returns a matrix with one row per sample
- Downstream exports (mpca_scores.csv, mpca_pc1_loadings.csv) have the expected row structure (samples × retained components for scores; second-dimension retention-time slices for PC1 loadings)

## Limitations

- MTBLS.rda contains already-preprocessed chromatograms, so baseline correction (baseline_corr), smoothing, and 2D-COW alignment steps are skipped and cannot be re-tuned on this object
- The example data are distributed via a separate GitHub repository (DanielQuiroz97/MTBLS579), not CRAN, so availability depends on that repo remaining accessible
- Only the full MTBLS579 dataset is referenced by identifier; no per-file raw chromatogram loading path is described for the bundled example

## Evidence

    ]"
