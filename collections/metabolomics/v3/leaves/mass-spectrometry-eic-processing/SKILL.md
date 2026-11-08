---
name: mass-spectrometry-eic-processing
description: Use when raw targeted LC-MS peak tables contain EICs whose retention-time
  shifts vary across samples and the goal is to compute per-sample peak areas for
  a specific isotopologue for downstream metabolite quantification.
when_to_use_negative:
- When the input is already an aligned feature table with per-sample peak areas —
  this skill operates on raw EIC traces that still need retention-time alignment.
- When the analysis goal is isotope-pattern prediction rather than quantification
  — TarMet's isotope-pattern stage is handled separately via enviPat before quantification.
- When the data are untargeted / DDA — the README explicitly lists data-independent-acquisition
  support as a future feature, not current capability.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TarMet
  role: Hosts the whole EIC-alignment and peak-area quantification workflow inside
    its reactive shiny GUI and R command-line vignette
  repo: https://github.com/hcji/TarMet
- name: PAFFT
  role: Cross-correlation method used to align the retention-time shifts of sample
    EICs to a reference sample
- name: airPLS
  role: Removes baselines of all the EICs before peak positioning and alignment
- name: MassSpecWavelet
  role: Detects peak positions in the baseline-corrected EICs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:0a2ca8c0cbc366cfad7a7f93ad16b55c6cf57498575dd44f14db5692a2c58911
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrometry-eic-processing

## Summary

TarMet's quantitative-analysis stage: align sample EIC retention-time shifts with the PAFFT cross-correlation method and integrate per-sample peak areas for a chosen isotopologue, producing a per-sample quantification table.

## When to use

Use when raw targeted LC-MS peak tables contain EICs whose retention-time shifts vary across samples and the goal is to compute per-sample peak areas for a specific isotopologue for downstream metabolite quantification.

## When NOT to use

- When the input is already an aligned feature table with per-sample peak areas — this skill operates on raw EIC traces that still need retention-time alignment.
- When the analysis goal is isotope-pattern prediction rather than quantification — TarMet's isotope-pattern stage is handled separately via enviPat before quantification.
- When the data are untargeted / DDA — the README explicitly lists data-independent-acquisition support as a future feature, not current capability.

## Inputs

- EICs (extracted-ion chromatograms) for the targeted isotopologue, extracted by TarMet from the MS run after airPLS baseline correction and MassSpecWavelet peak positioning
- Peak-info table (outputPeakInfo) with Start and End retention-time boundaries for the chosen isotopologue peak
- Sample index (sampleInd), maximum shift window (shift), and segment definition (segment) for PAFFT alignment

## Outputs

- Aligned EICs for all samples (returned by getAlignedEICs)
- quant_results.csv — a per-sample table whose columns are named by sample and whose values are the integrated peak areas for the chosen isotopologue

## How to apply

Load the extracted-ion chromatograms (EICs) for the targeted isotopologue from the peak tables that TarMet produced after airPLS baseline removal and MassSpecWavelet peak positioning. For each sample, call getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE) to align the sample EICs to a reference sample using the PAFFT cross-correlation method (Wong et al. 2005). Then call getQuantifiedResult(alignedEICs, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak]) to integrate the aligned peak area for the chosen isotopologue in each sample, using the start and end boundaries recorded in the peak-info table. Assemble the per-sample areas into a table whose columns are named by sample, and save it as quant_results.csv. This step runs inside TarMet's reactive shiny GUI (runTarMet()) or via the R-command-line vignette, and is described as part of TarMet's '2.3 Quantitative analysis' workflow.

## Related tools

- **TarMet** (Hosts the whole EIC-alignment and peak-area quantification workflow inside its reactive shiny GUI and R command-line vignette) — https://github.com/hcji/TarMet
- **PAFFT** (Cross-correlation method used to align the retention-time shifts of sample EICs to a reference sample)
- **airPLS** (Removes baselines of all the EICs before peak positioning and alignment)
- **MassSpecWavelet** (Detects peak positions in the baseline-corrected EICs)

## Examples

```
alignedEICs <- getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE); quant <- getQuantifiedResult(alignedEICs, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak]); write.csv(quant, 'quant_results.csv')
```

## Evaluation signals

- quant_results.csv exists after the workflow finishes and is readable
- The CSV contains one row per sample with aligned peak-area values for the selected isotopologue
- Values fall within a plausible signal-intensity range for the instrument (not zero and not saturated)
- Column names of the output table correspond exactly to the sample identifiers supplied to getAlignedEICs
- Re-running getAlignedEICs with the same sampleInd, shift, and segment reproduces the same aligned traces (PAFFT alignment is deterministic for fixed parameters)

## Limitations

- Retention-time shift tolerance is bounded by the shift and segment parameters supplied to getAlignedEICs; large drift beyond this window will be misaligned.
- Peak integration boundaries are taken from outputPeakInfo$Start and outputPeakInfo$End, so an incorrect MassSpecWavelet peak position propagates into the quantified area.
- Data-independent acquisition and MS/MS library verification are not yet supported (listed as future features in the README).

## Evidence
