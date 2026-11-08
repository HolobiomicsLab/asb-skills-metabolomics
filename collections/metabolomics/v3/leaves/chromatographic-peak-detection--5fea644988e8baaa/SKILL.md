---
name: chromatographic-peak-detection
description: Use when raw LC-MS targeted metabolomics EICs (e.g. centWave-exported
  chromatograms such as the MM14 or human blood datasets) need peak positions, bounds
  and areas quantified per compound, especially when multiple candidate peaks must
  be resolved to the true M+H/M+Na/M+K isotopologue peak.
when_to_use_negative:
- Input is already a quantified feature table or peak table — no chromatogram peak
  detection is needed.
- The analysis goal is untargeted feature finding (e.g. XCMS centWave peak picking
  itself) rather than targeted isotopologue extraction on predefined compounds.
- Only MS/MS spectral identification is required without chromatographic quantification.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: airPLS
  role: Removes the baseline of each extracted-ion chromatogram before peak detection
- name: MassSpecWavelet
  role: Detects peak positions in the baseline-corrected EICs; bounds are estimated
    via a Haar wavelet derivative for improved SNR
- name: enviPat
  role: Calculates theoretical isotope patterns of the target compounds at the MS
    resolution for isotopic-similarity scoring
- name: PAFFT
  role: Aligns retention-time shifts across samples so matched isotopologues can be
    integrated and quantified
- name: TarMet
  role: shiny application that packages the airPLS + MassSpecWavelet + enviPat pipeline
    into the peak-detection workflow
  repo: https://github.com/hcji/TarMet
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:92fdaa80432d5df18c93b525dc711a7b40ed1cd51e51c8b2bf310724240bea87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatographic-peak-detection

## Summary

Detects chromatographic peak positions, bounds and areas in extracted-ion chromatograms (EICs) for targeted metabolites and their isotopologues, using airPLS baseline removal and MassSpecWavelet peak detection as implemented in TarMet, with isotopic-similarity-based selection of the true target peak.

## When to use

Use when raw LC-MS targeted metabolomics EICs (e.g. centWave-exported chromatograms such as the MM14 or human blood datasets) need peak positions, bounds and areas quantified per compound, especially when multiple candidate peaks must be resolved to the true M+H/M+Na/M+K isotopologue peak.

## When NOT to use

- Input is already a quantified feature table or peak table — no chromatogram peak detection is needed.
- The analysis goal is untargeted feature finding (e.g. XCMS centWave peak picking itself) rather than targeted isotopologue extraction on predefined compounds.
- Only MS/MS spectral identification is required without chromatographic quantification.

## Inputs

- Extracted-ion chromatograms (EICs) for target metabolites, e.g. centWave-exported chromatograms from the MM14 dataset (http://msbi.ipb-halle.de/msbi/centwave)
- Target metabolite definitions (compound list with molecular formula) for isotope-pattern calculation
- MS instrument resolution (used by enviPat to compute theoretical isotope patterns)

## Outputs

- Peak-position/bounds/area table per target metabolite annotated with per-peak isotopic similarity scores
- Selected target-metabolite peaks (the candidate with maximum isotopic similarity)
- Per-compound feature counts and average isotopic similarities per adduct type (M+H, M+Na, M+K), e.g. Table S1-style summary

## How to apply

First compute theoretical isotope patterns for each target compound with the enviPat package at the instrument's MS resolution. Load the extracted-ion chromatograms and remove their baselines using the airPLS method, then detect peak positions with the MassSpecWavelet package. Determine peak bounds from a Haar-wavelet-detected valley between the target position and the neighbouring peak with the largest intensity, then calculate peak areas for each detected peak. When several peaks are found, compute the isotopic similarity between the theoretical and detected isotope patterns (Eq. 1) and keep the candidate with the maximum similarity as the peak of the target metabolite; this is what lets TarMet detect more isotopologues and quantify them better than state-of-the-art methods. Finally align retention-time shifts across samples with the PAFFT method so matched isotopologues can be integrated into a per-compound peak table (e.g. the Table S1 counts of 14 M+H, 13 M+Na and 8 M+K features with average similarities of 0.95/0.93/0.90).

## Related tools

- **airPLS** (Removes the baseline of each extracted-ion chromatogram before peak detection)
- **MassSpecWavelet** (Detects peak positions in the baseline-corrected EICs; bounds are estimated via a Haar wavelet derivative for improved SNR)
- **enviPat** (Calculates theoretical isotope patterns of the target compounds at the MS resolution for isotopic-similarity scoring)
- **PAFFT** (Aligns retention-time shifts across samples so matched isotopologues can be integrated and quantified)
- **TarMet** (shiny application that packages the airPLS + MassSpecWavelet + enviPat pipeline into the peak-detection workflow) — https://github.com/hcji/TarMet

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- The selected peak for each metabolite must be the candidate with the maximum isotopic similarity among all detected peaks.
- Reproduction on the MM14 dataset should recover TarMet's reported counts of 14 M+H, 13 M+Na and 8 M+K features of the 14 compounds, with average isotopic similarities of 0.95, 0.93 and 0.90 respectively.
- On the human blood metabolites dataset, TarMet should detect all 76 metabolites with their isotopologues, with 56 metabolites showing isotopic similarity > 0.90 and 69 > 0.80.
- Detected peak positions must yield bounds whose Haar-wavelet valley lies between the target position and the neighbouring higher-intensity peak.
- Quantification should outperform AssayR's counts (36 metabolites with similarity > 0.90 and 61 > 0.80) as reported for the same dataset.

## Limitations

- Peak-bound estimation relies on the Haar wavelet derivative, so heavily co-eluting or shoulder peaks may be mis-bounded.
- The isotopic-similarity selection assumes a correct theoretical isotope pattern from enviPat; wrong compound definitions or resolution settings degrade selection.
- Retention-time shifts must be aligned by PAFFT across samples; large uncorrected shifts will break cross-sample integration.
- The README notes planned future features (data-independent acquisition support and MS/MS library verification) that are not yet part of the pipeline.

## Evidence

- [abstract] baselines of all the EICs are removed by airPLS method: "baselines of all the EICs are removed by airPLS method"
- [abstract] Peak positions are detected by MassSpecWavelet package: "Peak positions are detected by MassSpecWavelet package"
- [abstract] TarMet will calculate the theoretical isotope patterns of the compounds based on the resolution of the MS via enviPat package: "TarMet will calculate the theoretical isotope patterns of the compounds based on the resolution of the MS via enviPat package"
- [abstract] TarMet uses PAFFT method (Wong et al. 2005 ) to align the retention time shifts across samples: "TarMet uses PAFFT method (Wong et al. 2005 ) to align the retention time shifts across samples"
- [methods] remove the baseline of each EIC using the airPLS method: [evidence span withheld — anchor status: not_found]
- [methods] detect peak positions with the MassSpecWavelet package: [evidence span withheld — anchor status: not_found]
- [intro] TarMet is a shiny application for targeted metabolic analyses based on mass spectrometry: "TarMet is a shiny application for targeted metabolic analyses based on mass spectrometry"
- [abstract] The MM14 dataset was released with the XCMS-centWave method: "The MM14 dataset was released with the XCMS-centWave method"
