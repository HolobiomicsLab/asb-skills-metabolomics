---
name: retention-time-alignment
description: Use when LC-MS targeted metabolomics EICs show retention-time drift between
  samples or replicate groups and the same isotopologue peak must be matched across
  samples before peak-area integration.
when_to_use_negative:
- Input is already an aligned, row-per-feature peak table from XCMS/centWave feature
  matching rather than raw per-sample EICs
- Data are MS/MS or data-independent acquisition spectra, which TarMet does not yet
  support (listed as future work in the article intro)
- No retention-time shift exists between samples, e.g. a single-sample run where alignment
  has no reference to correct against
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TarMet
  role: Hosts the alignment/quantification workflow; provides getAlignedEICs() and
    getQuantifiedResult() used to align EICs and integrate peak areas
  repo: https://github.com/hcji/TarMet
- name: PAFFT
  role: Cross-correlation method (Wong et al. 2005) used to estimate and correct retention-time
    shifts between sample EICs
- name: airPLS
  role: Removes EIC baselines before alignment so cross-correlation and peak integration
    operate on corrected chromatograms
- name: MassSpecWavelet
  role: Detects peak positions that define the Start/End boundaries integrated after
    alignment
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_006
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:adcfacede815e24c43874a6528f81f21efd21206bb1525a44dbb46f378689fd9
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# retention-time-alignment

## Summary

Aligns retention-time shifts of extracted-ion chromatograms (EICs) across samples using the PAFFT cross-correlation method so that the same targeted isotopologue can be matched and quantified consistently in every sample within TarMet.

## When to use

Use when LC-MS targeted metabolomics EICs show retention-time drift between samples or replicate groups and the same isotopologue peak must be matched across samples before peak-area integration.

## When NOT to use

- Input is already an aligned, row-per-feature peak table from XCMS/centWave feature matching rather than raw per-sample EICs
- Data are MS/MS or data-independent acquisition spectra, which TarMet does not yet support (listed as future work in the article intro)
- No retention-time shift exists between samples, e.g. a single-sample run where alignment has no reference to correct against

## Inputs

- Extracted-ion chromatograms (EICs) for targeted isotopologues produced by TarMet's airPLS + MassSpecWavelet peak-detection pipeline
- Sample indices (sampleInd) identifying which EIC belongs to which sample
- Peak boundary table (outputPeakInfo with Start/End columns) from MassSpecWavelet peak positioning

## Outputs

- Aligned EICs for all samples on a common retention-time axis
- Per-sample peak-area quantification table (e.g. quant_results.csv) with one row per sample and columns named by sample

## How to apply

After baseline removal (airPLS) and peak-position detection (MassSpecWavelet), extract the EIC of the chosen isotopologue from each sample. Apply the PAFFT cross-correlation method (Wong et al. 2005) via getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE) to shift each sample's EIC onto a reference sample's time axis. Once aligned, integrate the peak area between the previously detected start and end boundaries (outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak]) with getQuantifiedResult(alignedEICs, ...), collecting one area per sample into a table whose columns are named by sample. The rationale is that centWave-style raw runs have systematic RT shifts, so quantification without alignment would mismatch peaks across parallel samples; in the tracer case study, alignment was applied separately to the 5 min and 60 min 13C6-glucose labelled sample groups.

## Related tools

- **TarMet** (Hosts the alignment/quantification workflow; provides getAlignedEICs() and getQuantifiedResult() used to align EICs and integrate peak areas) — https://github.com/hcji/TarMet
- **PAFFT** (Cross-correlation method (Wong et al. 2005) used to estimate and correct retention-time shifts between sample EICs)
- **airPLS** (Removes EIC baselines before alignment so cross-correlation and peak integration operate on corrected chromatograms)
- **MassSpecWavelet** (Detects peak positions that define the Start/End boundaries integrated after alignment)

## Examples

```
alignedEICs <- getAlignedEICs(targetEICs, sampleInd, shift, segment, align=TRUE); quant <- getQuantifiedResult(alignedEICs, outputPeakInfo$Start[whPeak], outputPeakInfo$End[whPeak])
```

## Evaluation signals

- After getAlignedEICs(..., align=TRUE), the isotopologue peak of the same compound in all samples should co-occur at the same aligned retention time
- quant_results.csv contains exactly one row per sample with aligned peak-area values for the selected isotopologue
- Peak-area values fall within a plausible signal-intensity range for the instrument
- In the 13C6-glucose tracer dataset, parallel (replicate) samples show high consistency in isotopologue areas after alignment, and 60 min labelled samples show clearly higher 13C incorporation than 5 min samples
- Aligned peak areas agree with values reported in Table S1/S2/S3-equivalent outputs for the MM14, human blood, and cellular metabolites datasets

## Limitations

- PAFFT alignment operates on EICs of targeted isotopologues only; it is not an untargeted global feature-alignment method like XCMS grouping
- TarMet currently targets GC-MS/LC-MS targeted assays; support for data-independent acquisition is listed only as a planned future feature
- Shift and segment parameters of getAlignedEICs must be chosen appropriately; mis-set segments can misalign EICs and corrupt downstream peak-area integration

## Evidence

- [methods] getAlignedEICs aligns the sample EICs via the PAFFT method: [evidence span withheld — anchor status: not_found]
- [methods] getQuantifiedResult returns peak areas of the chosen isotopologue for all samples: [evidence span withheld — anchor status: not_found]
