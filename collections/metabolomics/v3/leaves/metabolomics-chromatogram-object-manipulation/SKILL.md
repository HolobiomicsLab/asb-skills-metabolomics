---
name: metabolomics-chromatogram-object-manipulation
description: Use when two or more separately imported/preprocessed 2D chromatogram
  objects (e.g., NetCDF-derived 2D-TICs from MetaboLights studies) must be merged
  into one R structure before multivariate analysis such as MPCA via m_prcomp.
when_to_use_negative:
- You have only a single chromatogram object — joining is unnecessary and the object
  can be passed directly to preprocessing/m_prcomp
- The chromatograms have not yet been imported and folded into 2D-TIC objects (e.g.,
  raw NetCDF files) — read_chrom must run first
- The goal is supervised modeling on an unfolded matrix — use unfold_chrom to export
  2D chromatograms for mixOmics instead of joining
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: RGCxGC
  role: Provides the join_chromatograms function used to merge multiple 2D-TIC chromatogram
    objects into one joined R object for multivariate analysis
  repo: https://github.com/DanielQuiroz97/RGCxGC/archive/master.zip
- name: R
  role: Runtime environment in which the joined chromatogram object is created, inspected,
    and saved (e.g., as .rds)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:e2150bef4aed0fff48d143a0923eacf1815b11a5b985b05c7e8bae8dd5eb08c8
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolomics-chromatogram-object-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines multiple GC×GC-MS chromatogram objects (e.g., MTBLS09 and MTBLS08 2D-TIC data) into a single joined R object using RGCxGC's join_chromatograms, so the pooled samples can feed downstream MPCA and other multivariate analysis steps of the RGCxGC workflow.

## When to use

Use when two or more separately imported/preprocessed 2D chromatogram objects (e.g., NetCDF-derived 2D-TICs from MetaboLights studies) must be merged into one R structure before multivariate analysis such as MPCA via m_prcomp.

## When NOT to use

- You have only a single chromatogram object — joining is unnecessary and the object can be passed directly to preprocessing/m_prcomp
- The chromatograms have not yet been imported and folded into 2D-TIC objects (e.g., raw NetCDF files) — read_chrom must run first
- The goal is supervised modeling on an unfolded matrix — use unfold_chrom to export 2D chromatograms for mixOmics instead of joining

## Inputs

- Two or more RGCxGC chromatogram objects (e.g., MTBLS09 and MTBLS08, imported from NetCDF and preprocessed)
- R session with the RGCxGC package installed (CRAN or devtools::install_github("DanielQuiroz97/RGCxGC"))

## Outputs

- A single joined chromatogram R object (e.g., allChrom/joined_chrom) containing both chromatograms, ready for MPCA
- Optionally, a serialized .rds file of the joined object for reproducibility

## How to apply

After reading each chromatogram (e.g., via read_chrom from NetCDF files, folded into 2D-TIC) and completing preprocessing (smoothing, baseline correction, peak alignment), call join_chromatograms with the chromatogram objects as arguments to produce one joined object holding all samples in a single structure. Inspect the joined object to confirm it contains all contributing chromatograms before proceeding. This follows the RGCxGC basic workflow of data import → preprocessing → multivariate analysis, where the joined object is the required container for MPCA (m_prcomp, with centering/scaling options). The joined object can be saved (e.g., as .rds) for reproducibility.

## Related tools

- **RGCxGC** (Provides the join_chromatograms function used to merge multiple 2D-TIC chromatogram objects into one joined R object for multivariate analysis) — https://github.com/DanielQuiroz97/RGCxGC/archive/master.zip
- **R** (Runtime environment in which the joined chromatogram object is created, inspected, and saved (e.g., as .rds))

## Examples

```
joined_chrom <- join_chromatograms(MTBLS09, MTBLS08); saveRDS(joined_chrom, "joined_chrom.rds")
```

## Evaluation signals

- join_chromatograms completes without error and returns a single joined chromatogram object
- The joined object contains both (all) contributing chromatograms, matching the vignette's described state before preprocessing/multivariate analysis
- The joined object is accepted downstream by m_prcomp with center/scale options (mean-centered and unit-scaled) without schema errors
- A saved joined_chrom.rds can be reloaded and re-used to reproduce the subsequent MPCA results

## Limitations

- The vignette demonstrates joining on specific package-provided data (MTBLS09 and MTBLS08); chromatograms from other sources must first be imported and preprocessed to a compatible RGCxGC structure
- Joining produces the container for analysis only — noise reduction still depends on upstream preprocessing (smoothing, baseline correction via asymmetric least squares, 2D-COW peak alignment)

## Evidence

- [other] Vignette join_chromatograms produces allChrom: [evidence span withheld — anchor status: not_found]
