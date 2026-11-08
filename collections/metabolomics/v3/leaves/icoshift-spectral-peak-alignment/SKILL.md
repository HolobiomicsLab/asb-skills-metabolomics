---
name: icoshift-spectral-peak-alignment
description: Use when a set of 1H-NMR spectra (already phase-corrected, internally
  referenced, baseline-corrected, region-removed) still shows peak drift between samples,
  and binned feature tables would blur shifted metabolite peaks if alignment were
  skipped.
when_to_use_negative:
- Input is still raw FID/time-domain data — FID-level steps (group_delay_removal,
  solvent_residuals_removal, apodization, zero_filling, fourier_transform) must come
  first; alignment operates on spectra.
- Data has not been internally referenced or baseline-corrected — shifting unreferenced
  or drift-baselined spectra aligns noise rather than metabolite peaks.
- Input is already a binned/normalized feature table — alignment must precede binning
  (bin_size=0.1) and TotalArea normalization, not follow them.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: Protomix
  role: Python package exposing the Icoshift class used to run peak alignment within
    the preprocessing pipeline
  repo: https://github.com/mzniber/protomix
- name: pyicoshift
  role: Python rewrite of the icoshift MATLAB algorithm performing correlation-based
    interval-wise peak alignment
  repo: github.com/sekro/pyicoshift
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:872ed3849813d66cc21ee54b72f004ba80f67d4982492ea11ce0124f3ab46c5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - pyicoshift
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# icoshift-spectral-peak-alignment

## Summary

Align 1H-NMR spectra in the Protomix pipeline using the pyicoshift rewrite of the icoshift MATLAB algorithm (Savorani, Tomasi, Engelsen), correcting peak misalignment caused by pH/ionic-strength variation between urine samples before binning and normalization.

## When to use

Use when a set of 1H-NMR spectra (already phase-corrected, internally referenced, baseline-corrected, region-removed) still shows peak drift between samples, and binned feature tables would blur shifted metabolite peaks if alignment were skipped.

## When NOT to use

- Input is still raw FID/time-domain data — FID-level steps (group_delay_removal, solvent_residuals_removal, apodization, zero_filling, fourier_transform) must come first; alignment operates on spectra.
- Data has not been internally referenced or baseline-corrected — shifting unreferenced or drift-baselined spectra aligns noise rather than metabolite peaks.
- Input is already a binned/normalized feature table — alignment must precede binning (bin_size=0.1) and TotalArea normalization, not follow them.

## Inputs

- spectra DataFrame (samples x ppm points) after Protomix phase_correction, internal_referencing, baseline_correction(method='airpls'), region_removal, and negative_values_zeroing
- acquisition/ppm axis of the 1H-NMR spectra from the Bruker-derived dataset (Zenodo record 13467227)

## Outputs

- icoshift-aligned spectra DataFrame suitable for window_selection and px.binning(bin_size=0.1, method='rectangular')
- per-spectrum interval shift information returned by the Icoshift object for QC inspection

## How to apply

Instantiate Protomix's Icoshift wrapper (px.Icoshift(), which calls the pyicoshift Python rewrite of the icoshift MATLAB algorithm) on the spectra DataFrame that emerges after phase_correction, internal_referencing, baseline_correction(method='airpls'), region_removal (e.g., 4.5–6.1 ppm water region) and negative_values_zeroing. Configure correlation-based interval alignment with target='median' (align each spectrum to the median spectrum), interval splitting via inter=('n_intervals', 100), enable a coarse global pre-alignment (global_pre_align=True) before per-interval shifts, and let the optimizer choose the shift bound via max_shift='best'. Call run() to obtain the aligned spectra, then pass the result to window_selection and downstream binning (bin_size=0.1, method='rectangular') and TotalArea normalization. Rationale: icoshift shifts spectral intervals (not whole spectra) to maximize correlation with the target, so urinary metabolite peaks that drift across the 70-sample prostate cancer/BPH dataset line up without distorting the baseline; pyicoshift additionally supports a highest-correlation target mode, auto interval picking, and experimental multi-threading for the shift-finding phase.

## Related tools

- **Protomix** (Python package exposing the Icoshift class used to run peak alignment within the preprocessing pipeline) — https://github.com/mzniber/protomix
- **pyicoshift** (Python rewrite of the icoshift MATLAB algorithm performing correlation-based interval-wise peak alignment) — github.com/sekro/pyicoshift

## Examples

```
icoshift = px.Icoshift(); icoshift.target = 'median'; icoshift.inter = ('n_intervals', 100); icoshift.global_pre_align = True; icoshift.max_shift = 'best'; aligned_df = icoshift.run(spectra_df)
```

## Evaluation signals

- Visually overlay aligned spectra: previously drifting peaks should coincide with the median target after run(), with per-interval shifts within the max_shift='best' bound.
- Mean pairwise correlation of spectra in the 100 icoshift intervals should increase relative to pre-alignment spectra.
- Subsequent binning (bin_size=0.1, rectangular) should yield sharper, less smeared binned peaks than binning unaligned spectra.
- Peak positions relative to the internal reference should remain chemically plausible — no intervals shifted into the removed 4.5–6.1 ppm region.
- The aligned spectra table should retain the same samples-by-ppm shape as its input (e.g., 70 samples from the prostate cancer/BPH urine dataset).

## Limitations

- pyicoshift's multi-threading for the optimal shift-finding phase and synthetic HMDB-target generation are explicitly labelled experimental in its README.
- icoshift-style interval alignment can misalign very low-SNR intervals or broad, weak peaks where the correlation-to-target criterion is unreliable.
- Alignment quality depends on prior steps: unreferenced or poorly phase-corrected spectra propagate errors into the shifts.

## Evidence

- [readme] pyicoshift is a Python rewrite of icoshift for MATLAB: [evidence span withheld — anchor status: external_doc]
- [readme] pyicoshift usage pattern: instance Icoshift, assign signals, choose intervals and target, call run(): [evidence span withheld — anchor status: external_doc]
- [readme] Pyicoshift supports experimental multi-threading for shift finding: [evidence span withheld — anchor status: external_doc]
- [methods] Peak alignment is performed via the Icoshift class in the Protomix workflow: "icoshift = px.Icoshift()"
- [readme] pyicoshift offers new target modes and interval picking beyond the MATLAB original: [evidence span withheld — anchor status: external_doc]
