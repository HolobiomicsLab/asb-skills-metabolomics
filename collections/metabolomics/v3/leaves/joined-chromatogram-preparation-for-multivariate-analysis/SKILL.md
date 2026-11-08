---
name: joined-chromatogram-preparation-for-multivariate-analysis
description: Use when several 2D-TIC chromatograms (e.g. NetCDF-imported samples)
  must be stacked into one R object before preprocessing and multivariate analysis
  such as MPCA.
when_to_use_negative:
- You already have a single chromatogram object and only need preprocessing (baseline
  correction, smoothing, 2D-COW alignment) or MPCA — no join is required
- The data are already unfolded/exported for supervised modeling via unfold_chrom
  for mixOmics — joining folded chromatograms is the wrong stage
- The input is raw NetCDF files not yet imported; use read_chrom first to obtain 2D-TIC
  chromatogram objects
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: RGCxGC
  role: Provides the join_chromatograms function that combines individual chromatogram
    objects into one joined object, and the downstream m_prcomp MPCA that consumes
    it
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Runtime environment in which the RGCxGC package functions (read_chrom, join_chromatograms,
    m_prcomp) are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:b5ec18c70f99ba58dfbf31243c93d1facddb70b3d9d4937349852b8725ea6e01
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# joined-chromatogram-preparation-for-multivariate-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines multiple imported GCxGC-MS chromatograms into a single joined R object using join_chromatograms, so that the multiway principal component analysis (MPCA) implemented in RGCxGC can be run over all samples at once.

## When to use

Use when several 2D-TIC chromatograms (e.g. NetCDF-imported samples) must be stacked into one R object before preprocessing and multivariate analysis such as MPCA.

## When NOT to use

- You already have a single chromatogram object and only need preprocessing (baseline correction, smoothing, 2D-COW alignment) or MPCA — no join is required
- The data are already unfolded/exported for supervised modeling via unfold_chrom for mixOmics — joining folded chromatograms is the wrong stage
- The input is raw NetCDF files not yet imported; use read_chrom first to obtain 2D-TIC chromatogram objects

## Inputs

- Two or more RGCxGC chromatogram objects imported from NetCDF files (e.g. MTBLS09, MTBLS08)
- 2D-TIC chromatograms folded from raw GCxGC-MS NetCDF data

## Outputs

- A single joined chromatogram R object (e.g. joined_chrom) containing all constituent chromatograms, ready for preprocessing and MPCA
- Optionally a serialized .rds file of the joined object for reproducibility

## How to apply

First import each raw chromatogram from a NetCDF file with read_chrom, which folds it into a two-dimensional Total Intensity Chromatogram (2D-TIC). Then call join_chromatograms on the individual chromatogram objects (e.g. the vignette's MTBLS09 and MTBLS08 datasets) to produce a single joined object holding both chromatograms. Verify the joined object contains all constituent chromatograms and shares a common folded 2D structure, since MPCA via m_prcomp must handle the three dimensions typical of a 2D-TIC from GCxGC. This join happens before preprocessing (smoothing, baseline correction, peak alignment) and before multivariate analysis, so any per-chromatogram preprocessing is applied downstream to the joined container. Save the joined object (e.g. as an .rds file) for reproducibility.

## Related tools

- **RGCxGC** (Provides the join_chromatograms function that combines individual chromatogram objects into one joined object, and the downstream m_prcomp MPCA that consumes it) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Runtime environment in which the RGCxGC package functions (read_chrom, join_chromatograms, m_prcomp) are executed)

## Examples

```
joined_chrom <- join_chromatograms(MTBLS09, MTBLS08); saveRDS(joined_chrom, "joined_chrom.rds")
```

## Evaluation signals

- join_chromatograms(MTBLS09, MTBLS08) completes without error and returns the allChrom/joined_chrom object
- The joined object holds both constituent chromatograms in one structure, matching the vignette's described state before preprocessing
- The joined object retains the three-dimensional 2D-TIC structure so that m_prcomp can compute MPCA with center/scale arguments
- The saved joined_chrom.rds reloads in R and is identical to the in-memory object
- Downstream m_prcomp on the joined object yields score and loading matrices accessible via scores/loading functions

## Limitations

- Chromatograms must be imported and folded into 2D-TIC first; joining raw NetCDF files directly is not supported by the vignette workflow
- The join must precede preprocessing, so per-sample decisions about smoothing, baseline correction (asymetric least squares), and peak alignment (2D COW) are applied to the joined container rather than before joining
- No changelog was found for the package, so version-specific behavior of join_chromatograms cannot be traced from the provided context

## Evidence
