---
name: two-dimensional-cow-peak-alignment
description: Use when multiple NetCDF-derived GCxGC 2D-TIC chromatograms show retention-time
  shifts that must be aligned to a reference chromatogram before downstream multivariate
  (MPCA) analysis or sample-group comparison.
when_to_use_negative:
- Input is not yet imported 2D-TIC chromatogram data (e.g. raw mass spectra or feature
  tables not derived from NetCDF/CDF chromatograms)
- The task is baseline correction or smoothing — use baseline_corr / Whittaker smoothing
  instead of warping
- Only a single chromatogram is to be aligned and no reference is available
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RGCxGC
  role: Provides batch_2DCOW and twod_cow functions for 2D-COW peak alignment of GCxGC
    chromatograms
  repo: github.com/DanielQuiroz97/RGCxGC
- name: ptw
  role: Supplies the coupled with1 & with2 warping base functions used by 2D-COW alignment
- name: RNetCDF
  role: Used to import raw chromatograms from NetCDF files into 2D-TIC objects prior
    to alignment
- name: R
  role: Execution environment for the RGCxGC alignment workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:64623717ac955b8d2bddfa7862c618922c1086859c0864a914fdeaf9aab8cd8f
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
  - RNetCDF
  - ptw
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# two-dimensional-cow-peak-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch alignment of GCxGC (two-dimensional) chromatograms against a reference using 2D Correlation Optimized Warping (2D-COW) via the RGCxGC package's batch_2DCOW/twod_cow functions, which couple the ptw package's with1 & with2 warping functions.

## When to use

Use when multiple NetCDF-derived GCxGC 2D-TIC chromatograms show retention-time shifts that must be aligned to a reference chromatogram before downstream multivariate (MPCA) analysis or sample-group comparison.

## When NOT to use

- Input is not yet imported 2D-TIC chromatogram data (e.g. raw mass spectra or feature tables not derived from NetCDF/CDF chromatograms)
- The task is baseline correction or smoothing — use baseline_corr / Whittaker smoothing instead of warping
- Only a single chromatogram is to be aligned and no reference is available

## Inputs

- Raw GCxGC chromatogram files in NetCDF/CDF format (e.g. 08GB.cdf, 09GB.cdf)
- A reference chromatogram (2D-TIC object, e.g. MTBLS09)
- A named list of sample chromatograms (2D-TIC objects)
- Segment boundaries (e.g. c(10, 40)) and maximum warp values (e.g. c(1, 10))

## Outputs

- An alignment object whose Batch_2DCOW slot contains the named aligned chromatograms (one per input sample)
- Optionally a saved .rda alignment object (e.g. batch_2dcow_alignment.rda)

## How to apply

Read each raw chromatogram from NetCDF/CDF files (e.g. 08GB.cdf, 09GB.cdf) with read_chrom, which folds them into two-dimensional Total Intensity Chromatogram (2D-TIC) objects. Assemble a named list of sample chromatograms (e.g. entries Chrom1, Chrom2, Chrom3 pointing to triplicated MTBLS08 chromatograms). Call batch_2DCOW with the reference chromatogram (e.g. MTBLS09), the named sample list, segment boundaries c(10, 40), and maximum warp c(1, 10); the underlying 2D-COW warping is done with with1 & with2 functions coupled from the ptw package. Verify the returned object's Batch_2DCOW slot holds one named aligned chromatogram per input sample, then optionally save the alignment object (.rda) for downstream preprocessing or m_prcomp MPCA.

## Related tools

- **RGCxGC** (Provides batch_2DCOW and twod_cow functions for 2D-COW peak alignment of GCxGC chromatograms) — github.com/DanielQuiroz97/RGCxGC
- **ptw** (Supplies the coupled with1 & with2 warping base functions used by 2D-COW alignment)
- **RNetCDF** (Used to import raw chromatograms from NetCDF files into 2D-TIC objects prior to alignment)
- **R** (Execution environment for the RGCxGC alignment workflow)

## Examples

```
batch_2DCOW(MTBLS09, batch_samples, c(10, 40), c(1, 10))
```

## Evaluation signals

- Returned object's Batch_2DCOW slot contains exactly one named aligned chromatogram per input sample (e.g. the three named entries Chrom1, Chrom2, Chrom3)
- Aligned chromatograms match the reference (e.g. MTBLS09) in retention-time structure within the specified maximum warp of c(1, 10) across segments c(10, 40)
- Aligned output chromatograms remain valid 2D-TIC objects suitable for downstream preprocessing and m_prcomp MPCA
- Alignment object can be saved and reloaded as an .rda file without loss of the Batch_2DCOW slot structure

## Limitations

- 2D-COW alignment requires a reference chromatogram and per-sample warping, so segment and max_warp parameters must be chosen appropriately for the retention shifts at hand
- Alignment is a preprocessing step intended to reduce noise before statistical analysis; it does not by itself reveal group differences

## Evidence

    ]"
- [readme] README 2D COW description: [evidence span withheld — anchor status: external_doc]
