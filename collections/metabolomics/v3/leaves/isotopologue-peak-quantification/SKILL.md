---
name: isotopologue-peak-quantification
description: Use when you have EICs for a targeted isotopologue across multiple tracer
  samples (e.g. 13C6-glucose labelling) and need retention-time-aligned peak areas
  per sample for isotopologue quantification.
when_to_use_negative:
- The goal is untargeted peak picking rather than targeted isotopologue quantification
  of known metabolites
- Retention-time shifts are already negligible (e.g. same run, no cross-sample alignment
  needed), making PAFFT alignment unnecessary
- Isotopologue patterns must be derived from raw spectra rather than quantified from
  pre-detected EICs — use theoretical pattern calculation via enviPat instead
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TarMet
  role: Provides getAlignedEICs and getQuantifiedResult functions used to align EICs
    and integrate peak areas per sample
  repo: https://github.com/hcji/TarMet
- name: PAFFT
  role: Cross-correlation method used by getAlignedEICs to align retention-time shifts
    across samples (Wong et al. 2005)
- name: airPLS
  role: Removes EIC baselines upstream so aligned peaks are quantifiable
- name: MassSpecWavelet
  role: Detects peak positions whose Start/End indices feed getQuantifiedResult
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:11482363799f1ea1c2adb32e080b4bcfb4b033e4aa50001f84f714b162bd2e64
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# isotopologue-peak-quantification

## Summary

Integrate aligned extracted-ion chromatogram (EIC) peak areas for a chosen metabolite isotopologue across samples using TarMet's PAFFT alignment and getQuantifiedResult integration, producing a per-sample peak-area table for stable isotope tracer quantification.

## When to use

Use when you have EICs for a targeted isotopologue across multiple tracer samples (e.g. 13C6-glucose labelling) and need retention-time-aligned peak areas per sample for isotopologue quantification.

## When NOT to use

- The goal is untargeted peak picking rather than targeted isotopologue quantification of known metabolites
- Retention-time shifts are already negligible (e.g. same run, no cross-sample alignment needed), making PAFFT alignment unnecessary
- Isotopologue patterns must be derived from raw spectra rather than quantified from pre-detected EICs — use theoretical pattern calculation via enviPat instead

## Inputs

- Extracted-ion chromatograms (EICs) for the targeted isotopologue, per sample, from TarMet's peak-detection pipeline
- Peak Start and End indices from TarMet's peak-info table (outputPeakInfo$Start, outputPeakInfo$End)
- Sample index and PAFFT alignment parameters (shift, segment)

## Outputs

- Per-sample table of integrated peak areas for the selected isotopologue (columns named by sample)
- quant_results.csv with one row per sample of aligned peak-area values

## How to apply

Start from EICs extracted for each isotopologue of the target metabolite (produced by TarMet's airPLS baseline removal and MassSpecWavelet peak positioning). Pass the sample EICs and the peak Start/End indices from the peak-info output to getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE), which applies the PAFFT cross-correlation method (Wong et al. 2005) to align retention-time shifts across samples to a reference. Then call getQuantifiedResult(alignedEICs, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak]) to integrate the peak area of the chosen isotopologue in every sample. Collect the returned per-sample areas into a table whose columns are named by sample and save as quant_results.csv. Validate by checking the CSV exists, has one row per sample, and that values fall within a plausible signal-intensity range.

## Related tools

- **TarMet** (Provides getAlignedEICs and getQuantifiedResult functions used to align EICs and integrate peak areas per sample) — https://github.com/hcji/TarMet
- **PAFFT** (Cross-correlation method used by getAlignedEICs to align retention-time shifts across samples (Wong et al. 2005))
- **airPLS** (Removes EIC baselines upstream so aligned peaks are quantifiable)
- **MassSpecWavelet** (Detects peak positions whose Start/End indices feed getQuantifiedResult)

## Examples

```
aligned <- getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE); areas <- getQuantifiedResult(aligned, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak])
```

## Evaluation signals

- quant_results.csv exists and contains one row per sample with aligned peak-area values for the selected isotopologue
- Peak-area values fall within a plausible signal-intensity range
- Parallel replicate samples show high consistency in isotopologue areas (as seen in TarMet's Fig. 2 tracer workflow)
- 13C-labelled time points (e.g. 60 min vs 5 min 13C6-glucose) show clearly higher 13C incorporation, matching the expected labelling pattern

## Limitations

- Relies on PAFFT alignment, so severe retention-time drift or mis-assigned reference may shift integration windows
- Isotopologue quantification depends on prior airPLS baseline removal and MassSpecWavelet peak positioning; errors there propagate
- Currently limited to MS1 targeted/isotopologue workflows; MS/MS library verification and data-independent acquisition are planned future features

## Evidence

- [methods] PAFFT alignment call: [evidence span withheld — anchor status: not_found]
- [methods] Peak integration call: [evidence span withheld — anchor status: not_found]
- [results] Tracer validation result: [evidence span withheld — anchor status: not_found]
