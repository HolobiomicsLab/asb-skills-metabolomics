---
name: retention-time-axis-construction
description: Use when raw GCxGC-MS acquisition data arrives as a NetCDF (.cdf) file
  and you must build the two-dimensional retention-time axis (first dimension vs modulation-period
  second dimension) before smoothing, baseline correction, alignment, or MPCA.
when_to_use_negative:
- Input is already a folded 2D chromatogram or preprocessed RGCxGC S4 object rather
  than a raw NetCDF file — re-reading would discard prior folding/preprocessing.
- Data are 1D GC-MS runs without modulation, where a second retention-time axis cannot
  be constructed.
- You need unfolded matrices for supervised modeling — use unfold_chrom downstream
  instead of re-importing.
edam_topics:
- http://edamontology.org/topic_3388
tools:
- name: RGCxGC
  role: Provides read_chrom, which imports the NetCDF raw chromatogram and folds it
    into the 2D-TIC S4 object with retention-time axes
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: RNetCDF
  role: Underlies the import of the raw chromatogram from the NetCDF file before folding
- name: R
  role: Runtime environment in which read_chrom is called and the resulting chromatogram
    object is saved as .rda
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:25b9a69e977744dabb29dfd7aab0dc58f4a500ccb46bef7eb56ba368e20a343a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3388
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# retention-time-axis-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Import a raw GCxGC-MS chromatogram from a NetCDF file with RGCxGC's read_chrom and fold the 1D signal into the two-dimensional Total Intensity Chromatogram (2D-TIC) using the modulation time, producing an S4 chromatogram object with retention-time axes ready for preprocessing.

## When to use

Use when raw GCxGC-MS acquisition data arrives as a NetCDF (.cdf) file and you must build the two-dimensional retention-time axis (first dimension vs modulation-period second dimension) before smoothing, baseline correction, alignment, or MPCA.

## When NOT to use

- Input is already a folded 2D chromatogram or preprocessed RGCxGC S4 object rather than a raw NetCDF file — re-reading would discard prior folding/preprocessing.
- Data are 1D GC-MS runs without modulation, where a second retention-time axis cannot be constructed.
- You need unfolded matrices for supervised modeling — use unfold_chrom downstream instead of re-importing.

## Inputs

- Raw GCxGC-MS chromatogram as a NetCDF (.cdf) file (e.g. 08GB.cdf from the MTBLS08/MTBLS579 dataset)
- Modulation time of the GCxGC acquisition (mod_time, in seconds, e.g. 5)

## Outputs

- RGCxGC S4 chromatogram object containing the folded 2D-TIC matrix, first-dimension retention times, NetCDF file name, and modulation time (saved as e.g. 08GB_raw_chrom.rda)

## How to apply

Call read_chrom on the NetCDF file, passing the modulation time (mod_time; e.g. mod_time = 5 s for the packaged 08GB.cdf from MTBLS08) so the importer folds the continuous 1D signal into the 2D-TIC matrix. Optionally set sam_rate, per_eval, and x_cut/y_cut to control sampling rate, evaluation period, and retention-time window trimming during import. Verify the returned S4 object exposes its four slots: the folded 2D-TIC chromatogram, the first-dimension retention times, the source NetCDF file name, and the modulation time. Save the object (e.g. as an .rda) since all downstream RGCxGC preprocessing (baseline_corr, batch_2DCOW, m_prcomp) consumes this folded chromatogram object.

## Related tools

- **RGCxGC** (Provides read_chrom, which imports the NetCDF raw chromatogram and folds it into the 2D-TIC S4 object with retention-time axes) — https://github.com/DanielQuiroz97/RGCxGC
- **RNetCDF** (Underlies the import of the raw chromatogram from the NetCDF file before folding)
- **R** (Runtime environment in which read_chrom is called and the resulting chromatogram object is saved as .rda)

## Examples

```
chrom_08 <- read_chrom("08GB.cdf", mod_time = 5); save(chrom_08, file = "08GB_raw_chrom.rda")
```

## Evaluation signals

- The returned object is an RGCxGC S4 chromatogram with four slots: folded 2D-TIC chromatogram, first-dimension retention times, NetCDF file name, and modulation time.
- The modulation-time slot matches the value passed to mod_time (e.g. 5 for the 08GB.cdf demo file), confirming correct second-dimension folding.
- The stored file-name slot matches the source NetCDF path, confirming the intended acquisition was imported.
- Second-dimension retention times per modulation period do not exceed mod_time; first-dimension retention times fall within any x_cut/y_cut trimming applied.
- Downstream functions (baseline_corr, batch_2DCOW, m_prcomp) accept the object without reshaping errors, confirming correct folding.

## Limitations

- Folding correctness depends on an accurate mod_time; a wrong modulation time misassigns the second retention-time axis of the entire 2D-TIC.
- x_cut/y_cut trimming at import permanently restricts the retention-time window available to downstream preprocessing and alignment.
- No changelog found for the package, so behavior changes across versions are undocumented.

## Evidence

    ]"
- [discussion] raw 08GB import task finding: [evidence span withheld — anchor status: not_found]
