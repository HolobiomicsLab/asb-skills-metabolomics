---
name: isotope-pattern-similarity-scoring
description: Use when multiple peaks are detected in an EIC for a targeted metabolite
  (or its isotopologues) and the correct peak must be selected using isotope-pattern
  agreement rather than intensity alone.
when_to_use_negative:
- The analysis goal is untargeted feature detection (e.g. XCMS-centWave on the MM14
  dataset) rather than targeted metabolite/isotopologue quantification
- Only one candidate peak exists per EIC and no isotope-pattern disambiguation is
  needed
- MS/MS library verification or data-independent acquisition data is the input — these
  are planned future features, not supported by TarMet
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: TarMet
  role: Implements the targeted extraction pipeline including Eq. 1 isotopic similarity
    scoring and peak selection; available as a shiny app and R package
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Computes theoretical isotope patterns of compounds at the instrument resolution,
    the reference side of the similarity comparison
- name: airPLS
  role: Removes the baselines of all the EICs before peak detection and similarity
    scoring
- name: MassSpecWavelet
  role: Detects peak positions in baseline-corrected EICs; combined with Haar-wavelet
    derivative peak-bound estimation
- name: shiny
  role: Provides TarMet's reactive graphical user interface for interactive isotopologue
    analysis and quantification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:e3fd547f83940432314e5f3e5beb9a3f6555ee4ca42ccb63821c31c6bfd4ce52
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Isotope-pattern similarity scoring

## Summary

Scores candidate peaks in extracted-ion chromatograms by the isotopic similarity (Eq. 1 of TarMet) between theoretical isotope patterns computed by enviPat at the instrument resolution and the detected isotope patterns, keeping the maximum-similarity peak as the target metabolite.

## When to use

Use when multiple peaks are detected in an EIC for a targeted metabolite (or its isotopologues) and the correct peak must be selected using isotope-pattern agreement rather than intensity alone.

## When NOT to use

- The analysis goal is untargeted feature detection (e.g. XCMS-centWave on the MM14 dataset) rather than targeted metabolite/isotopologue quantification
- Only one candidate peak exists per EIC and no isotope-pattern disambiguation is needed
- MS/MS library verification or data-independent acquisition data is the input — these are planned future features, not supported by TarMet

## Inputs

- Extracted-ion chromatograms (EICs) for the target metabolite and its isotopologues from LC-MS data
- Theoretical isotope patterns of the compounds computed by enviPat at the MS instrument resolution

## Outputs

- Peak-position/bounds/area table for detected peaks
- Per-peak isotopic similarity values annotated onto the quantification table
- Selected target-metabolite peak (maximum isotopic similarity) with its quantified area

## How to apply

Load the extracted-ion chromatograms for the target metabolite together with theoretical isotope patterns computed by enviPat at the MS resolution. Remove the baseline of each EIC with airPLS, detect peak positions with MassSpecWavelet, and estimate peak bounds using a Haar-wavelet derivative for improved SNR (taking the valley between the target position and the peak with the largest intensity). Calculate peak areas for the detected peaks. Compute the isotopic similarity of Eq. 1 between theoretical and detected isotope patterns and keep the peak with maximum similarity as the target-metabolite peak, annotating the output table with per-peak similarity values.

## Related tools

- **TarMet** (Implements the targeted extraction pipeline including Eq. 1 isotopic similarity scoring and peak selection; available as a shiny app and R package) — https://github.com/hcji/TarMet
- **enviPat** (Computes theoretical isotope patterns of compounds at the instrument resolution, the reference side of the similarity comparison)
- **airPLS** (Removes the baselines of all the EICs before peak detection and similarity scoring)
- **MassSpecWavelet** (Detects peak positions in baseline-corrected EICs; combined with Haar-wavelet derivative peak-bound estimation)
- **shiny** (Provides TarMet's reactive graphical user interface for interactive isotopologue analysis and quantification)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- The selected peak must show the maximum isotopic similarity among all candidate peaks in the EIC
- Quantification of the selected peak should reflect TarMet's reported behavior of detecting more isotopologues and quantifying them better than state-of-the-art methods
- Peak bounds should lie at a valley detected by the Haar-wavelet derivative between the target position and the largest-intensity neighboring peak
- Theoretical patterns used for comparison must be computed by enviPat at the actual instrument resolution, not a default resolution
- Stable isotope tracer assays should be processed correctly (isotopologues of the same metabolite coherently quantified)

## Limitations

- Support for data-independent acquisition is a planned future feature, not currently implemented
- Support for MS/MS library verification is a planned future feature, not currently implemented
- Baseline correction, peak detection, and similarity scoring operate on EICs, so results depend on prior target metabolite definition and isotopologue analysis steps

## Evidence
