---
name: reference-spectrum-selection
description: Use when a series of 1D-NMR spectra shows chemical-shift drift and MetaboLabPy's
  segmental alignment is about to be run on a user-selected spectral region, requiring
  a reference spectrum to be designated.
when_to_use_negative:
- The selected spectral region lacks sufficient signal in the candidate reference
  — segmental alignment performs no alignment when reference signal is lacking, so
  a different region must be chosen instead.
- Input is not a series of 1D-NMR spectra (e.g. already-binned feature tables or 2D
  spectra), where correlation-based shift determination against a reference does not
  apply.
- Spectra are already aligned relative to a prior reference and only downstream steps
  (spectral scaling, variance stabilisation) are needed.
edam_topics:
- http://edamontology.org/topic_3822
tools:
- name: MetaboLabPy 0.9.66
  role: Provides the segmental alignment routine whose optimum-shift determination
    runs against the selected reference spectrum
  repo: https://github.com/ludwigc/MetaboLabPy
- name: metabolabpy
  role: Python package (installable from PyPI) whose nmrData objects hold the spectra
    series from which the reference is designated
  repo: https://github.com/ludwigc/metabolabpy
- name: numpy
  role: Backend toolbox used for the simple-correlation computation of optimum shifts
    against the reference
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:df81bf1f049383115897addaca77b2ab2e6098224e616ba1a357595cf1a1b218
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - metabolabpy
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3822
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# reference-spectrum-selection

## Summary

Selects the spectrum that serves as the alignment reference in MetaboLabPy's segmental alignment of a 1D-NMR spectra series. The optimum shift of every other spectrum is computed by simple correlation with this reference, so choosing the spectrum with the highest signal-to-noise ratio in the selected region directly determines alignment quality.

## When to use

Use when a series of 1D-NMR spectra shows chemical-shift drift and MetaboLabPy's segmental alignment is about to be run on a user-selected spectral region, requiring a reference spectrum to be designated.

## When NOT to use

- The selected spectral region lacks sufficient signal in the candidate reference — segmental alignment performs no alignment when reference signal is lacking, so a different region must be chosen instead.
- Input is not a series of 1D-NMR spectra (e.g. already-binned feature tables or 2D spectra), where correlation-based shift determination against a reference does not apply.
- Spectra are already aligned relative to a prior reference and only downstream steps (spectral scaling, variance stabilisation) are needed.

## Inputs

- Series of 1D-NMR spectra (e.g. Daphnia magna extracts) loaded into MetaboLabPy 0.9.66
- User-selected spectral region used for segmental alignment
- Per-spectrum signal-to-noise assessment in the selected region

## Outputs

- Designated reference spectrum (identity/index) for the selected region
- Per-spectrum optimum shift table written to segmental_alignment_shifts.csv
- Post-alignment correlation coefficients of each spectrum vs the reference (alignment_correlations.csv)

## How to apply

Load the 1D-NMR spectra series into MetaboLabPy (0.9.66, Python with numpy/scipy/pandas backend) and select the spectral region to align. Inspect signal-to-noise in that region across spectra and designate the spectrum with the highest signal-to-noise ratio as the alignment reference. Rationale: MetaboLabPy computes the optimum shift for each spectrum by simple correlation with the chosen reference across candidate shifts, so a low-SNR reference degrades every computed shift. Before running alignment, verify the reference segment contains sufficient signal — segmental alignment performs no alignment for segments lacking reference signal. After alignment, confirm that the reference itself is unchanged (shift 0) and that other spectra were shifted toward it.

## Related tools

- **MetaboLabPy 0.9.66** (Provides the segmental alignment routine whose optimum-shift determination runs against the selected reference spectrum) — https://github.com/ludwigc/MetaboLabPy
- **metabolabpy** (Python package (installable from PyPI) whose nmrData objects hold the spectra series from which the reference is designated) — https://github.com/ludwigc/metabolabpy
- **numpy** (Backend toolbox used for the simple-correlation computation of optimum shifts against the reference)

## Examples

```
$ metabolabpy --help
```

## Evaluation signals

- Pearson correlation coefficients between each spectrum and the reference in the selected segment are generally below 0.3 before alignment.
- After segmental alignment, the same correlations rise to >0.95.
- segmental_alignment_shifts.csv exists with one optimum-shift entry per processed spectrum.
- By expert review, aligned segments in the selected region overlay the reference spectrum.
- The reference spectrum itself should have an optimum shift of zero (it is the target of alignment, not shifted).

## Limitations

- No alignment occurs when the reference signal is lacking in the chosen region — a low-signal reference silently disables the alignment.
- Optimum shifts are found by simple correlation with the reference, so the outcome is sensitive to reference quality (noise levels) in the selected region.
- Alignment is confined to the user-selected spectral region; drift elsewhere is not corrected.

## Evidence

- [full_text] Designate the highest-SNR spectrum as reference: [evidence span withheld — anchor status: not_found]
- [abstract] Workflow includes segmental alignment: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms, and analysis of metabolic"
- [results] Correlation improvement after alignment: [evidence span withheld — anchor status: not_found]
- [readme] Package availability: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
