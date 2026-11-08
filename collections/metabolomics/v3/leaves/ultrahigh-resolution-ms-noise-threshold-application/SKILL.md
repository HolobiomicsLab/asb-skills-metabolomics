---
name: ultrahigh-resolution-ms-noise-threshold-application
description: Use when a raw two-column ion mass / intensity peak list from ultrahigh-resolution
  MS needs a defensible noise cutoff prior to formula assignment, and the analyst
  must choose and validate an S/N threshold.
when_to_use_negative:
- Input spectra are not ultrahigh-resolution mass spectra (e.g. low-resolution unit-mass
  data) where Kendrick-mass-defect noise separation is unreliable
- A noise threshold has already been established and validated for the dataset — re-running
  noise estimation on pre-filtered Mono peak lists is out of scope
- The data are very discontinuous MS/MS spectra; the MFAssign 'MSMS' parameter path
  bypasses pre-filtering of ions below the de novo threshold instead
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MFAssignR
  role: R package providing KMDNoise(), HistNoise(), and SNplot() for noise estimation
    and S/N threshold validation
  repo: github.com/skschum/MFAssignR
- name: KMDNoise
  role: Custom noise-estimation function based on raw-data Kendrick mass defect analysis
  repo: github.com/skschum/MFAssignR
- name: HistNoise
  role: Alternative noise-estimation function based on the natural log intensity histogram
    method of Zhurov et al. (2014)
  repo: github.com/skschum/MFAssignR
- name: SNplot
  role: Diagnostic plotting function to check the effectiveness of the chosen S/N
    threshold on the spectrum
  repo: github.com/skschum/MFAssignR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.envres.2020.110114
    title: 'MFAssignR: Molecular formula assignment software for ultrahigh resolution
      mass spectrometry analysis of environmental complex mixtures'
  source_content_hash: sha256:2a3a96438399e4386d2d2e4f0bd64c3f2ed5a54d7d1b52eb9abddc192d6249d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HistNoise
  - KMDNoise
  - MFAssign
  - MFAssignCHO
  - MFAssignR
  - R
  - SNplot
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ultrahigh-resolution-ms-noise-threshold-application

## Summary

Estimate the noise level of an ultrahigh-resolution mass spectrum (FT-ICR / Orbitrap) using MFAssignR's KMDNoise (Kendrick mass defect based) or HistNoise (natural log intensity histogram, Zhurov et al. 2014) methods, then apply the resulting S/N threshold to retain only peaks above noise before isotope filtering and molecular formula assignment.

## When to use

Use when a raw two-column ion mass / intensity peak list from ultrahigh-resolution MS needs a defensible noise cutoff prior to formula assignment, and the analyst must choose and validate an S/N threshold.

## When NOT to use

- Input spectra are not ultrahigh-resolution mass spectra (e.g. low-resolution unit-mass data) where Kendrick-mass-defect noise separation is unreliable
- A noise threshold has already been established and validated for the dataset — re-running noise estimation on pre-filtered Mono peak lists is out of scope
- The data are very discontinuous MS/MS spectra; the MFAssign 'MSMS' parameter path bypasses pre-filtering of ions below the de novo threshold instead

## Inputs

- Two-column dataframe of ion mass and intensity (raw peak list) from ultrahigh resolution mass spectrometry (FT-ICR or Orbitrap)
- User-defined S/N multiplier used to convert the KMDNoise/HistNoise noise estimate into a peak threshold

## Outputs

- Estimated noise level for the spectrum (numeric S/N threshold)
- SNplot() diagnostic plot showing masses below (red) and above (blue) the threshold
- S/N-filtered peak list passed downstream to IsoFiltR() and MFAssignCHO()/MFAssign()

## How to apply

Run KMDNoise() on the raw peak list to determine the noise level — a custom method based on Kendrick mass defect (CH2 base) analysis of raw data — or alternatively HistNoise(), which follows Zhurov et al. (2014) and uses the histogram distribution of natural log intensities. Check the effectiveness of the resulting S/N threshold with SNplot(), which displays the spectrum with masses below the threshold in red and above in blue, iterating on the threshold until low-mass and baseline regions are cleanly separated. The accepted S/N threshold is then carried forward as an input to IsoFiltR() and MFAssignCHO()/MFAssign() so that only above-noise masses enter formula assignment. The rationale is that noise estimation and polyisotopic filtering are prerequisites in the MFAssignR pipeline: an unreliable threshold propagates noise peaks into spurious MF assignments.

## Related tools

- **MFAssignR** (R package providing KMDNoise(), HistNoise(), and SNplot() for noise estimation and S/N threshold validation) — github.com/skschum/MFAssignR
- **KMDNoise** (Custom noise-estimation function based on raw-data Kendrick mass defect analysis) — github.com/skschum/MFAssignR
- **HistNoise** (Alternative noise-estimation function based on the natural log intensity histogram method of Zhurov et al. (2014)) — github.com/skschum/MFAssignR
- **SNplot** (Diagnostic plotting function to check the effectiveness of the chosen S/N threshold on the spectrum) — github.com/skschum/MFAssignR

## Examples

```
noise_level <- KMDNoise(peak_list); SNplot(peak_list, noise_level)
```

## Evaluation signals

- SNplot() output shows a clean separation: masses below the threshold (red) concentrated in the baseline/noise region, above-threshold masses (blue) forming the real spectral peaks
- Noise estimates from KMDNoise and HistNoise (Zhurov histogram method) are compared; large divergence between the two methods flags a threshold that needs manual inspection
- Downstream MFAssignCHO() run with the accepted S/N threshold yields plausible mass accuracy on recalibrant CH2 homologous series, indicating noise peaks did not contaminate assignments
- The S/N threshold used to feed IsoFiltR() and MFAssignCHO() is documented and reproducible, matching the pipeline step order (noise estimation precedes isotope filtering and assignment)

## Limitations

- User caution with parameter settings and output evaluation is required; several function outputs are provided to assist the user with these evaluations
- KMDNoise is a custom method based on observations of raw data rather than a formally validated statistical model; HistNoise is an alternative whose results may differ
- The threshold effectiveness is not automatic — it must be visually verified with SNplot() before proceeding

## Evidence
