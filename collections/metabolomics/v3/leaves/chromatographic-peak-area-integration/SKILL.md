---
name: chromatographic-peak-area-integration
description: Use when processing targeted MS data where peak detection has produced
  EICs (after airPLS baseline correction and MassSpecWavelet peak positioning) and
  per-sample peak areas must be computed for a chosen isotopologue or metabolite.
when_to_use_negative:
- The data are untargeted feature-based workflows where raw features are already aligned
  and quantified by XCMS-centWave — TarMet is targeted and uses its own EIC extraction
  and PAFFT alignment
- Retention-time alignment is not needed because only a single sample or a single
  injected run is analysed
- The user needs MS/MS library verification or data-independent acquisition support
  — TarMet explicitly lists these as future, not current, features
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: TarMet
  role: Shiny application that performs the targeted quantification workflow, including
    EIC extraction, PAFFT alignment, and peak-area integration for isotopologues
  repo: https://github.com/hcji/TarMet
- name: PAFFT
  role: Cross-correlation method used to align retention-time shifts of EICs across
    samples to a reference sample before integration
  repo: http://msbi.ipb-halle.de/msbi/centwave
- name: airPLS
  role: Baseline-removal preprocessing applied to all EICs prior to peak positioning
    and integration
- name: MassSpecWavelet
  role: Peak-position detection package whose Start/End boundaries define the integration
    window for each isotopologue
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:f3830b5d4d5e467fd2d750124546d28757fcc2f8ea966080c02fe6da6981950c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chromatographic-peak-area-integration

## Summary

Integrate aligned extracted-ion chromatogram (EIC) peak areas for targeted metabolites and their isotopologues across samples, as implemented in TarMet's quantitative analysis stage, to produce per-sample quantification tables for metabolite and stable-isotope-tracer assays.

## When to use

Use when processing targeted MS data where peak detection has produced EICs (after airPLS baseline correction and MassSpecWavelet peak positioning) and per-sample peak areas must be computed for a chosen isotopologue or metabolite.

## When NOT to use

- The data are untargeted feature-based workflows where raw features are already aligned and quantified by XCMS-centWave — TarMet is targeted and uses its own EIC extraction and PAFFT alignment
- Retention-time alignment is not needed because only a single sample or a single injected run is analysed
- The user needs MS/MS library verification or data-independent acquisition support — TarMet explicitly lists these as future, not current, features

## Inputs

- Extracted-ion chromatograms (EICs) for the targeted isotopologue, from TarMet peak tables (post airPLS baseline correction and MassSpecWavelet peak positioning)
- Peak-boundary positions (Start/End) from outputPeakInfo for the selected isotopologue
- Sample index (sampleInd) and alignment parameters (shift, segment) for PAFFT

## Outputs

- A per-sample table of integrated peak areas for the chosen isotopologue, columns named by sample
- quant_results.csv containing the aligned peak-area values

## How to apply

Load the extracted-ion chromatograms for the targeted isotopologue from the peak tables produced by TarMet's peak detection pipeline (airPLS baseline correction followed by MassSpecWavelet peak positioning). Align retention-time shifts across samples to a reference sample using the PAFFT cross-correlation method (Wong et al. 2005) via getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE). Then call getQuantifiedResult(alignedEICs, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak]) to integrate the peak area of the chosen isotopologue over the detected Start/End boundaries for every sample. Assemble the resulting per-sample areas into a table whose columns are named by sample and save it as quant_results.csv. The rationale is that PAFFT alignment compensates for retention-time drift so that the same isotopologue peak is integrated consistently across samples, which is essential for accurate isotopologue abundance comparisons in tracer assays.

## Related tools

- **TarMet** (Shiny application that performs the targeted quantification workflow, including EIC extraction, PAFFT alignment, and peak-area integration for isotopologues) — https://github.com/hcji/TarMet
- **PAFFT** (Cross-correlation method used to align retention-time shifts of EICs across samples to a reference sample before integration) — http://msbi.ipb-halle.de/msbi/centwave
- **airPLS** (Baseline-removal preprocessing applied to all EICs prior to peak positioning and integration)
- **MassSpecWavelet** (Peak-position detection package whose Start/End boundaries define the integration window for each isotopologue)

## Examples

```
library(TarMet); alignedEICs <- getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE); quant <- getQuantifiedResult(alignedEICs, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak]); write.csv(quant, 'quant_results.csv')
```

## Evaluation signals

- quant_results.csv exists and contains one row per sample with aligned peak-area values for the selected isotopologue
- Column names of the output table match the sample identifiers used in the run
- Peak-area values fall within a plausible signal-intensity range consistent with the instrument response for that metabolite
- Each integration window corresponds to the Start/End boundaries reported by MassSpecWavelet peak positioning for the chosen isotopologue
- PAFFT alignment has been applied (getAlignedEICs with align=TRUE) before integration, so retention-time shifts across samples are compensated

## Limitations

- TarMet currently does not support data-independent acquisition; support is listed as a future feature
- TarMet currently does not support MS/MS library verification; support is listed as a future feature
- Quantification quality depends on the quality of upstream EIC extraction and MassSpecWavelet peak positioning; incorrect peak boundaries propagate directly into the integrated area

## Evidence

- [methods] getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE) aligns the sample EICs via the PAFFT method, and getQuantifiedResult(alignedEICs, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak]) returns peak areas: [evidence span withheld — anchor status: not_found]
- [abstract] TarMet will calculate the theoretical isotope patterns of the compounds based on the resolution of the MS via enviPat package: "TarMet can quantify targeted metabolites as well as their isotopologues through a reactive and user-friendly graphical user interface."
- [abstract] baselines of all the EICs are removed by airPLS method: "baselines of all the EICs are removed by airPLS method"
- [abstract] Peak positions are detected by MassSpecWavelet package: "Peak positions are detected by MassSpecWavelet package"
- [abstract] TarMet uses PAFFT method (Wong et al. 2005 ) to align the retention time shifts across samples: "TarMet uses PAFFT method (Wong et al. 2005 ) to align the retention time shifts across samples"
- [abstract] Quantitative analysis: "2.3 Quantitative analysis"
