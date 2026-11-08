---
name: wavelet-transform-peak-bounds-detection
description: Use when quantifying targeted metabolites/isotopologues from LC-MS EICs
  and you need wavelet-based peak positions plus valley-determined bounds on baseline-corrected
  chromatograms.
when_to_use_negative:
- Input is already a peak-picked feature table (peaks and areas exist) — the wavelet
  detection step would be redundant
- The goal is untargeted feature detection across samples rather than targeted quantification
  of known metabolites and their isotopologues
- Data are MS/MS (data-independent acquisition) spectra, which TarMet explicitly does
  not yet support
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: MassSpecWavelet
  role: Detects peak positions in baseline-corrected EICs
- name: airPLS
  role: Removes EIC baselines prior to wavelet peak detection
- name: enviPat
  role: Computes theoretical isotope patterns at the MS resolution for the similarity-based
    peak selection
- name: TarMet
  role: Shiny application embedding the EIC-to-peak pipeline (airPLS, MassSpecWavelet,
    Haar-wavelet bounds, isotopic similarity)
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: Provides the reactive GUI through which the peak-detection workflow is run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:bd6a575956382032e52c40424393cc420b62fa93356f588469566eda978e5405
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# wavelet-transform-peak-bounds-detection

## Summary

Detects peak positions and peak bounds in extracted-ion chromatograms (EICs) using the MassSpecWavelet package, after airPLS baseline removal, with a Haar-wavelet derivative used to find valley-based bounds and improve SNR — the core peak-detection mechanism inside TarMet's isotopologue quantification pipeline.

## When to use

Use when quantifying targeted metabolites/isotopologues from LC-MS EICs and you need wavelet-based peak positions plus valley-determined bounds on baseline-corrected chromatograms.

## When NOT to use

- Input is already a peak-picked feature table (peaks and areas exist) — the wavelet detection step would be redundant
- The goal is untargeted feature detection across samples rather than targeted quantification of known metabolites and their isotopologues
- Data are MS/MS (data-independent acquisition) spectra, which TarMet explicitly does not yet support

## Inputs

- Extracted-ion chromatograms (EICs) for the target metabolite and its isotopologues
- Theoretical isotope patterns of the compound computed by enviPat at the instrument resolution
- Raw LC-MS data readable via mzR

## Outputs

- Table of detected peak positions, peak bounds, and peak areas per EIC
- Per-peak isotopic similarity values used to select the target-metabolite peak

## How to apply

First load the EICs for the target metabolite and its theoretical isotope patterns (computed by enviPat at the MS instrument resolution). Remove each EIC's baseline with the airPLS method so peaks stand out from drift. Detect peak positions with the MassSpecWavelet package; when multiple positions are found, the true target peak is later chosen by isotopic similarity between theoretical and detected isotope patterns. For each detected position, determine peak bounds from the valley identified by a Haar-wavelet derivative between the target position and the neighboring peak with the largest intensity, then integrate to obtain peak areas. Output a table of peak positions, bounds, and areas annotated with per-peak isotopic-similarity values.

## Related tools

- **MassSpecWavelet** (Detects peak positions in baseline-corrected EICs)
- **airPLS** (Removes EIC baselines prior to wavelet peak detection)
- **enviPat** (Computes theoretical isotope patterns at the MS resolution for the similarity-based peak selection)
- **TarMet** (Shiny application embedding the EIC-to-peak pipeline (airPLS, MassSpecWavelet, Haar-wavelet bounds, isotopic similarity)) — https://github.com/hcji/TarMet
- **shiny** (Provides the reactive GUI through which the peak-detection workflow is run)

## Examples

```
BiocManager::install("MassSpecWavelet")
```

## Evaluation signals

- The peak selected as the target metabolite must show the maximum isotopic similarity between theoretical and detected isotope patterns among all detected candidates
- Every peak position reported by MassSpecWavelet has associated bounds derived from the Haar-wavelet valley search (no unbounded peaks in the output table)
- Baseline-corrected EICs show no residual baseline drift before peak calling (airPLS applied to each EIC)
- Peak-area quantification is consistent with TarMet's reported behavior of detecting and quantifying more isotopologues than state-of-the-art methods (e.g., on the MM14, human blood, and cellular metabolites datasets)

## Limitations

- When multiple peaks are detected per EIC, correctness depends on isotopic-pattern similarity; ambiguous isotope patterns may lead to wrong peak selection
- Peak bounds are anchored to the valley between the target position and the adjacent largest-intensity peak, so heavily co-eluting peaks can distort bounds
- Currently limited to targeted MS analyses: data-independent acquisition and MS/MS library verification are only planned future features

## Evidence
