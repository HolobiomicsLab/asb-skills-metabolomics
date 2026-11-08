---
name: 13c-natom-equivalent-conversion
description: Use when HRMS lipidomics data from a 13C time-course labeling experiment
  contains PC(36:4) molecular species with resolved MS2 acyl-chain fragments and the
  goal is chain-resolved 13C enrichment in natom equivalents.
when_to_use_negative:
- Data were acquired at nominal mass resolution — dual-labeled isotopologues that
  differ by <0.01 Da cannot be resolved, so chain-resolved isotopologue deconvolution
  is not reliable
- MS2 spectra are unavailable for the resolved PC(36:4) isotopologues, so acyl-chain
  fragments cannot be quantified
- The goal is precursor-level (whole-lipid) MID reporting only, already handled by
  MIDsTableNoScale/NACorrectionFxnII without chain deconvolution
tools:
- name: SIMPEL
  role: R package performing isotopologue distribution generation, natural abundance
    correction, and natom-equivalent conversion for the lipidomics dataset
  repo: https://github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processes acquired HRMS data into the feature table that is imported into
    SIMPEL
- name: IsoCorrectoR
  role: Wrapped by NACorrectionFxnII to perform natural-abundance correction of the
    acyl-chain MIDs table
- name: LC-MS/MS
  role: Acquisition platform providing the high-resolution MS1 and MS2 spectra of
    PC(36:4) isotopologues
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:a52bb6dbeaf33dc3da9572112c35f0132a81ef820f359fb1ec8ad049cdada065
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SIMPELmetabolism/SIMPEL
  tools:
  - IsoCorrectoR
  - LC-MS/MS
  - R
  - SIMPEL
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# 13C-natom-equivalent-conversion

## Summary

A SIMPEL-based workflow that resolves 13C labeling within individual fatty acyl chains of PC(36:4) (18:1/18:2/18:3) from high-resolution MS2 spectra of resolved isotopologues, applies IsoCorrectoR natural-abundance correction, and converts corrected fractional enrichments into 13C natom equivalents scaled by measured lipid pool sizes.

## When to use

Use when HRMS lipidomics data from a 13C time-course labeling experiment contains PC(36:4) molecular species with resolved MS2 acyl-chain fragments and the goal is chain-resolved 13C enrichment in natom equivalents.

## When NOT to use

- Data were acquired at nominal mass resolution — dual-labeled isotopologues that differ by <0.01 Da cannot be resolved, so chain-resolved isotopologue deconvolution is not reliable
- MS2 spectra are unavailable for the resolved PC(36:4) isotopologues, so acyl-chain fragments cannot be quantified
- The goal is precursor-level (whole-lipid) MID reporting only, already handled by MIDsTableNoScale/NACorrectionFxnII without chain deconvolution

## Inputs

- XCMS-pre-processed HRMS feature table (LC-MS/MS lipidomics from 13C time-course labeling)
- High-resolution MS2 spectra of each PC(36:4) isotopologue (e.g., M0, M6) containing acyl-chain fragments
- SIMPEL label_scheme describing the tracer ([U-13C6] glucose time course, 0-32 h)
- Measured lipid pool sizes for PC(36:4)

## Outputs

- Corrected natom-equivalent 13C enrichment table for the 18:1, 18:2, 18:3 acyl chains of PC(36:4)
- MIDs_table_NA_corrected.csv (natural-abundance-corrected natom-equivalent table)
- Pair-summed chain-combination weights (M0/M6, M3/M3, ..., M6/M0) per acyl chain

## How to apply

Build the unscaled isotopologue distribution from the XCMS-pre-processed feature table using MIDsTableNoScale(XCMS_data, label_scheme). Analyze high-resolution MS2 spectra of each resolved PC(36:4) isotopologue (e.g., unlabeled M0 vs labeled M6) to quantify labeling in the 18:1, 18:2, and 18:3 acyl-chain fragments and compute the pair-summed weights M0/M6, M3/M3, ... M6/M0 for each chain combination. Apply natural-abundance correction to the acyl-chain MIDs table with NACorrectionFxnII(MIDs_table, label_scheme, output), which wraps IsoCorrectoR. Finally scale corrected fractional enrichments by measured lipid pool sizes to express 13C enrichment as natom equivalents, exploiting HRMS resolution that distinguishes isotopologues differing by <0.01 Da.

## Related tools

- **SIMPEL** (R package performing isotopologue distribution generation, natural abundance correction, and natom-equivalent conversion for the lipidomics dataset) — https://github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processes acquired HRMS data into the feature table that is imported into SIMPEL)
- **IsoCorrectoR** (Wrapped by NACorrectionFxnII to perform natural-abundance correction of the acyl-chain MIDs table)
- **LC-MS/MS** (Acquisition platform providing the high-resolution MS1 and MS2 spectra of PC(36:4) isotopologues)

## Examples

```
MIDs_table <- MIDsTableNoScale(XCMS_data, label_scheme); MIDs_table_NA <- NACorrectionFxnII(MIDs_table, label_scheme, output)
```

## Evaluation signals

- Conservation check: the summed pair weights (M0/M6, M3/M3, ..., M6/M0) must reproduce the precursor isotopologue M6 fraction of PC(36:4)
- Natural-abundance correction step (IsoCorrectoR via NACorrectionFxnII) must complete without errors on the acyl-chain MIDs table
- Final natom-equivalent pattern matches that shown in Fig. 4 and Supplementary Data 1 Tables 8-9
- Output file MIDs_table_NA_corrected.csv exists and contains fractional enrichments converted to natom equivalents scaled by lipid pool sizes
- Only isotopologues resolvable by HRMS (m/z differing by <0.01 Da) should be assigned to specific chain-combination states

## Limitations

- Requires high-resolution MS: at nominal m/z resolution dual-labeled tracers produce nominally identical mass shifts, limiting resolution of chain-combination states
- Depends on MS2 spectra being available for each resolved isotopologue; no software versions for SIMPEL, XCMS, or IsoCorrectoR are reported
- Chain assignments assume the PC(36:4) fatty acid proportions are known, since the isotopologue distribution is adjusted for pool size and the proportions of the constituent fatty acids

## Evidence

- [results] MS2 spectra for each of the isotopologues were used to determine the distribution of FA labeling within the acyl chains of PC(36:4): "MS2 spectra for each of the isotopologues were used to determine the distribution of FA labeling within the acyl chains of PC(36:4)"
- [methods] NACorrectionFxnII(MIDs_table, label_scheme, output): "NACorrectionFxnII(MIDs_table, label_scheme, output)"
- [intro] Acquired data were first pre-processed using XCMS, and post-processing data analysis was performed using SIMPEL.: "Acquired data were first pre-processed using XCMS, and post-processing data analysis was performed using SIMPEL."
- [intro] High resolution mass spectrometry (HRMS) can distinguish metabolites (and isotopologues of heteroatoms) through accurate measurement of m/z values that differ by <0.01 Da: "High resolution mass spectrometry (HRMS) can distinguish metabolites (and isotopologues of heteroatoms) through accurate measurement of m/z values that differ by <0.01 Da"
