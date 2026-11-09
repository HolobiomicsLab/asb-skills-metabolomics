---
name: lock-mass-spectral-recalibration
description: Use when imported centroided .imzML lipid MSI files exhibit m/z drift
  across pixels and you need mass-axis stabilization before dynamic binning, lipid
  annotation, or quantitation in pmol/mm².
when_to_use_negative:
- The input data are already accurately recalibrated or come from a platform whose
  recalibration parameters have already been validated for that acquisition
- The only available reference signal is a low-abundance peak (e.g. <8000 a.u. on
  timsTOF instruments), which may lead to incorrect signal assignment
- The task is adduct/isobar correction or internal-standard normalization rather than
  mass-axis stabilization — LipidQMap handles those in separate workflow steps
edam_topics: []
tools:
- name: LipidQMap
  role: 'Implements lock-mass (online) recalibration in its imzML import dialog: user
    supplies a reference m/z, a ppm tolerance, and a minimum intensity, and each qualifying
    pixel spectrum is shifted to the reference mass.'
  repo: github.com/swinnenteam/LipidQMap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:793dae32d12b6594f1062ac717122776dee51c3c8dc865fea30833c8d6c8efcc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipidQMap
  - alexandrovteam/pyimzML
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lock-mass-spectral-recalibration

## Summary

Corrects pixel-level m/z drift in centroided MSI data by shifting each pixel spectrum against user-selected reference (lock) masses, as implemented in LipidQMap's online calibration step, so subsequent peak binning and lipid annotation operate on a stable mass axis.

## When to use

Use when imported centroided .imzML lipid MSI files exhibit m/z drift across pixels and you need mass-axis stabilization before dynamic binning, lipid annotation, or quantitation in pmol/mm².

## When NOT to use

- The input data are already accurately recalibrated or come from a platform whose recalibration parameters have already been validated for that acquisition
- The only available reference signal is a low-abundance peak (e.g. <8000 a.u. on timsTOF instruments), which may lead to incorrect signal assignment
- The task is adduct/isobar correction or internal-standard normalization rather than mass-axis stabilization — LipidQMap handles those in separate workflow steps

## Inputs

- Centroided .imzML files (vendor-neutral MSI input; AnnData .h5ad and MuData .h5mu also supported by LipidQMap)
- User-selected lock-mass reference m/z value(s)
- Peak assignment tolerance in ppm (default 30 ppm)
- Minimum reference-peak intensity (default 10 000)

## Outputs

- Lock-mass-recalibrated pixel spectra with m/z axes shifted to match the reference mass
- Stabilized centroided peak lists feeding downstream dynamic binning and ion-image extraction

## How to apply

In LipidQMap's imzML import dialog, enable online (lock-mass) calibration and supply a reference m/z value together with a tolerance (in ppm) for the maximum allowed deviation from that reference mass and a minimum intensity threshold for the reference peak; when a pixel's spectrum meets these criteria, its spectrum is shifted to match the reference m/z. The article's defaults are a peak assignment tolerance of 30 ppm and a minimum intensity of 10 000 for lock-mass recalibration. Rationale: only peaks that are confidently assigned to the reference mass should drive the shift, so low-abundance signals (e.g. <8000 a.u. on timsTOF instruments) must be avoided as references because they lead to incorrect signal assignment and compromised data. Apply recalibration before the dynamic binning step that aligns centroided peaks across all pixels at constant ppm resolution, and adjust the recalibration parameters (tolerance, minimum intensity, reference mass) per instrument platform, since defaults tuned for one platform may not transfer to others.

## Related tools

- **LipidQMap** (Implements lock-mass (online) recalibration in its imzML import dialog: user supplies a reference m/z, a ppm tolerance, and a minimum intensity, and each qualifying pixel spectrum is shifted to the reference mass.) — github.com/swinnenteam/LipidQMap

## Evaluation signals

- After recalibration, the selected reference m/z falls within the configured tolerance (default 30 ppm) in every processed pixel
- Only pixels whose reference-peak intensity exceeds the minimum threshold (default 10 000) are shifted; low-intensity pixels are not forced onto the reference mass
- Downstream dynamic binning shows peaks co-locating in the same ppm bin across all pixels, with constant ppm resolution maintained across the mass range
- Recalibration parameters (tolerance, minimum intensity) were reviewed and adjusted when the dataset came from a different mass spectrometer platform
- Subsequent lipid annotation against the .csv database assigns peaks at the expected m/z values without systematic off-target matches

## Limitations

- Low-abundance signals — such as intensities <8000 a.u. on timsTOF instruments — should be avoided for recalibration because they may lead to incorrect signal assignment and compromise the data
- Recalibration parameters may need to be adjusted for datasets acquired on different mass spectrometer platforms
- The bin size used for the average spectrum (default 5 mDa for TOF instruments) should be decreased for higher-resolution instruments, which interacts with the recalibration setup

## Evidence

- [readme] Online calibration requires a reference m/z, a ppm tolerance, and a minimum intensity: [evidence span withheld — anchor status: external_doc]
