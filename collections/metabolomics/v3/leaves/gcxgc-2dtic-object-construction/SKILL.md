---
name: gcxgc-2dtic-object-construction
description: Use when multiple raw GC×GC-MS chromatograms (e.g. NetCDF acquisitions
  from different samples such as MTBLS09 and MTBLS08) must be combined into one R
  object before multivariate analysis.
when_to_use_negative:
- You have only one chromatogram and simply need to import/fold it into a 2D-TIC —
  use read_chrom instead
- The data have already been unfolded into a feature-style matrix for supervised modeling
  (unfold_chrom / mixOmics path), where joining raw chromatogram objects is not the
  target structure
- Chromatograms have not yet been read into RGCxGC chromatogram objects or are not
  aligned/preprocessed enough to be comparable in a joint analysis
tools:
- name: RGCxGC
  role: Provides join_chromatograms to merge multiple chromatograms into one object,
    and read_chrom to fold raw NetCDF acquisitions into 2D-TIC objects beforehand
  repo: github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Runtime environment in which the RGCxGC vignette workflow (import, join, MPCA)
    is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:f60f1f375002ffdc2dc6c399fd7f2411b72e3b11f9a40460f962785538d31420
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/DanielQuiroz97/RGCxGC
  tools:
  - R
  - RGCxGC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gcxgc-2dtic-object-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a single R data object holding multiple GC×GC chromatograms (imported as 2D-TIC from NetCDF files) so they can be jointly preprocessed and subjected to multiway principal component analysis (MPCA) in the RGCxGC package.

## When to use

Use when multiple raw GC×GC-MS chromatograms (e.g. NetCDF acquisitions from different samples such as MTBLS09 and MTBLS08) must be combined into one R object before multivariate analysis.

## When NOT to use

- You have only one chromatogram and simply need to import/fold it into a 2D-TIC — use read_chrom instead
- The data have already been unfolded into a feature-style matrix for supervised modeling (unfold_chrom / mixOmics path), where joining raw chromatogram objects is not the target structure
- Chromatograms have not yet been read into RGCxGC chromatogram objects or are not aligned/preprocessed enough to be comparable in a joint analysis

## Inputs

- Raw GC×GC-MS chromatograms in NetCDF format
- Pre-imported RGCxGC chromatogram objects (2D-TIC), e.g. MTBLS09 and MTBLS08 from the vignette data

## Outputs

- A single joined chromatogram R object (e.g. allChrom) containing both chromatograms, suitable for MPCA via m_prcomp
- Optionally a serialized copy saved as joined_chrom.rds for reproducibility

## How to apply

First import each raw acquisition with read_chrom, supplying the modulation time (mod_time) so the 1D signal is folded into the two-dimensional Total Intensity Chromatogram (2D-TIC). Optionally apply preprocessing (smoothing, baseline correction via baseline_corr, peak alignment via 2D-COW) before joining so chromatograms share a comparable structure. Then call join_chromatograms on the chromatogram objects (e.g. join_chromatograms(MTBLS09, MTBLS08)) to produce a single joined object (allChrom) that stacks the samples. Verify the joined object holds all constituent chromatograms; it is then the direct input to MPCA via m_prcomp (with centering/scaling arguments) in the package's two-step workflow of preprocessing followed by multivariate analysis.

## Related tools

- **RGCxGC** (Provides join_chromatograms to merge multiple chromatograms into one object, and read_chrom to fold raw NetCDF acquisitions into 2D-TIC objects beforehand) — github.com/DanielQuiroz97/RGCxGC
- **R** (Runtime environment in which the RGCxGC vignette workflow (import, join, MPCA) is executed)

## Examples

```
joined_chrom <- join_chromatograms(MTBLS09, MTBLS08); saveRDS(joined_chrom, "joined_chrom.rds")
```

## Evaluation signals

- join_chromatograms(MTBLS09, MTBLS08) completes without error and returns a joined chromatogram object
- The joined object contains both constituent chromatograms, matching the vignette's described state before preprocessing and multivariate analysis
- The joined object is accepted directly by m_prcomp with center and scale arguments, yielding a valid MPCA result
- Each input chromatogram has correct 2D-TIC structure (folded by the modulation time supplied at read_chrom time)
- Saving to joined_chrom.rds and reloading reproduces an identical object

## Limitations

- The joined object is intended for the package's two-step preprocessing + multivariate analysis workflow; it is not itself a preprocessed or supervised-model-ready feature matrix
- Chromatograms must be compatible (comparable modulation/acquisition structure) for the join to be meaningful for downstream MPCA
- Data must originate from NetCDF acquisitions importable by read_chrom; other formats are not covered by the described workflow

## Evidence

- [other] The vignette demonstrates combining chromatograms into one R object by calling join_chromatograms: [evidence span withheld — anchor status: not_found]
