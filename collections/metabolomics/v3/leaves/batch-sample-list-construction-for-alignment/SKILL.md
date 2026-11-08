---
name: batch-sample-list-construction-for-alignment
description: Use when several raw GCxGC NetCDF/CDF chromatograms must be peak-aligned
  against a single reference chromatogram in one batch operation, and the alignment
  function requires samples bundled as a named R list.
when_to_use_negative:
- When only a single sample chromatogram needs alignment — use the single-pair 2D-COW
  path (twod_cow) rather than assembling a batch list.
- When the data are already preprocessed peak tables or unfolded matrices rather than
  raw NetCDF chromatograms; read_chrom expects NetCDF files and batch_2DCOW operates
  on folded 2D-TIC objects.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: RGCxGC
  role: Provides read_chrom to import NetCDF chromatograms as 2D-TIC objects and batch_2DCOW
    to perform simultaneous 2D-COW alignment of the named sample list against a reference
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Underlies the import of the raw chromatogram from a NetCDF file before it
    is folded into the 2D-TIC object
- name: ptw
  role: Supplies the base warping functions (with1 & with2) that batch_2DCOW couples
    to warp each sample chromatogram onto the reference
- name: R
  role: Runtime environment in which the named sample list is assembled and the batch
    alignment is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:2ed5cfcef3554228f9cde25f7f4d4976374e77e9f8fcb665ef523e57b12ab8eb
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
  - RNetCDF
  - ptw
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# batch-sample-list-construction-for-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a named list of GCxGC 2D-TIC chromatogram objects so that multiple samples can be simultaneously aligned to a reference chromatogram via batch_2DCOW (2D Correlation Optimized Warping) in the RGCxGC package.

## When to use

Use when several raw GCxGC NetCDF/CDF chromatograms must be peak-aligned against a single reference chromatogram in one batch operation, and the alignment function requires samples bundled as a named R list.

## When NOT to use

- When only a single sample chromatogram needs alignment — use the single-pair 2D-COW path (twod_cow) rather than assembling a batch list.
- When the data are already preprocessed peak tables or unfolded matrices rather than raw NetCDF chromatograms; read_chrom expects NetCDF files and batch_2DCOW operates on folded 2D-TIC objects.

## Inputs

- Raw GCxGC chromatogram files in NetCDF/CDF format (e.g., 08GB.cdf, 09GB.cdf)
- 2D-TIC chromatogram objects produced by read_chrom in R
- A reference chromatogram object (e.g., MTBLS09)

## Outputs

- A named R list of 2D-TIC chromatogram objects (e.g., batch_samples with Chrom1/Chrom2/Chrom3 entries)
- A batch_2DCOW alignment object whose Batch_2DCOW slot holds the named aligned chromatograms (e.g., saved as batch_2dcow_alignment.rda)

## How to apply

Read each raw chromatogram file (e.g., 08GB.cdf and 09GB.cdf from the RGCxGC example data) with read_chrom, which imports the NetCDF data and folds it into a two-dimensional Total Intensity Chromatogram (2D-TIC) object. Assemble the sample chromatograms into a named list — matching the vignette's batch example structure with entries such as Chrom1, Chrom2, and Chrom3 all pointing to the MTBLS08 chromatogram — because batch_2DCOW preserves these names in its output. Call batch_2DCOW(reference, batch_samples, segments, max_warp), passing the reference chromatogram (e.g., MTBLS09), the named sample list, segment boundaries such as c(10, 40), and a maximum warp such as c(1, 10); alignment is performed per sample by the ptw-package-coupled with1 & with2 warping functions. Verify that the returned object's Batch_2DCOW slot contains one named aligned chromatogram per input before saving the result.

## Related tools

- **RGCxGC** (Provides read_chrom to import NetCDF chromatograms as 2D-TIC objects and batch_2DCOW to perform simultaneous 2D-COW alignment of the named sample list against a reference) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Underlies the import of the raw chromatogram from a NetCDF file before it is folded into the 2D-TIC object)
- **ptw** (Supplies the base warping functions (with1 & with2) that batch_2DCOW couples to warp each sample chromatogram onto the reference)
- **R** (Runtime environment in which the named sample list is assembled and the batch alignment is executed)

## Examples

```
batch_samples <- list(Chrom1 = MTBLS08, Chrom2 = MTBLS08, Chrom3 = MTBLS08); aln <- batch_2DCOW(MTBLS09, batch_samples, c(10, 40), c(1, 10))
```

## Evaluation signals

- The returned object has a populated Batch_2DCOW slot containing exactly as many aligned chromatograms as entries in the input named list.
- The names in the Batch_2DCOW slot match the names of the input list entries (e.g., Chrom1, Chrom2, Chrom3), confirming the named-list construction was correct.
- Each input file loaded via read_chrom yields a folded two-dimensional Total Intensity Chromatogram (2D-TIC) object without import errors.
- Aligned chromatograms show reduced retention-time offset relative to the reference within the specified max_warp bounds (c(1, 10) in the vignette example).
- The alignment object can be saved to disk (e.g., batch_2dcow_alignment.rda) and reloaded intact.

## Limitations

- Alignment quality depends on the segment boundaries and max_warp parameters; the vignette example uses c(10, 40) segments and c(1, 10) max warp, and other datasets may need retuning.
- The batch list entries must be chromatogram objects produced from NetCDF import (read_chrom); the workflow in the article builds all sample entries from the same MTBLS08 chromatogram in its triplicate demonstration.
- Only batch_2DCOW / twod_cow parameters (reference, sample_chroms, segments, max_warp, add_ref) are documented in the article's function signature extract; behavior of undocumented options is not covered by the cited context.

## Evidence

- [other] Batch alignment is reproduced by building a named list of the triplicated MTBLS08 sample chromatograms and calling batch_2DCOW: [evidence span withheld — anchor status: not_found]
- [intro] the raw chromatogram is importing from a **NetCDF** file and is folded into the two-dimensional Total Intensity Chromatogram (2D-TIC): "the raw chromatogram is importing from a **NetCDF** file and is folded into the two-dimensional Total Intensity Chromatogram (2D-TIC)"
- [intro] The base functions (with1 & with2) are coupled from the *ptw* package: "The base functions (with1 & with2) are coupled from the *ptw* package"
- [abstract] "batch_2DCOW": ["reference", "sample_chroms", "segments", "max_warp", "add_ref"]: ""batch_2DCOW": [
        "reference",
        "sample_chroms",
        "segments",
        "max_warp",
        "add_ref"
      ]"
