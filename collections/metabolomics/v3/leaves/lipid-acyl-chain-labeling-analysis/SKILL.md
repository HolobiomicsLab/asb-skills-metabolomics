---
name: lipid-acyl-chain-labeling-analysis
description: Use when HRMS lipidomics data from a 13C labeling time course contains
  a resolved lipid molecular species (e.g., PC(36:4)) whose precursor isotopologue
  distribution must be decomposed into per-acyl-chain (18:1, 18:2, 18:3) labeling
  using MS2 fragment spectra.
when_to_use_negative:
- Input is raw LC-MS vendor files that have not been pre-processed with XCMS into
  a feature table with isotopologue-level resolution
- MS2 spectra are nominal-resolution only and cannot distinguish resolved isotopologues
  (e.g., M0 vs M6) of the lipid species
- Goal is precursor-level (whole-lipid) isotopologue distribution without per-acyl-chain
  decomposition
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: SIMPEL
  role: R package providing MIDsTableNoScale and NACorrectionFxnII to build unscaled
    isotopologue distributions and perform natural-abundance correction of acyl-chain
    MIDs
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processing of acquired HRMS lipidomics data into the feature table consumed
    by SIMPEL
- name: IsoCorrectoR
  role: Wrapped by NACorrectionFxnII to correct acyl-chain MIDs for natural abundance
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:1b29c1e0c1b9b109e3868dc255bc53575fc04fd32d08688978edc18cb36091ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoCorrectoR
  - R
  - SIMPEL
  - XCMS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# lipid-acyl-chain-labeling-analysis

## Summary

A SIMPEL-based procedure for reconstructing how 13C labeling is distributed among the individual fatty acyl chains of a phospholipid molecular species, using high-resolution MS2 spectra of resolved isotopologues (e.g., unlabeled M0 vs labeled M6 of PC(36:4)) to attribute precursor enrichment to acyl-chain combinations such as 18:1/18:2/18:3 pair-summed weights.

## When to use

Use when HRMS lipidomics data from a 13C labeling time course contains a resolved lipid molecular species (e.g., PC(36:4)) whose precursor isotopologue distribution must be decomposed into per-acyl-chain (18:1, 18:2, 18:3) labeling using MS2 fragment spectra.

## When NOT to use

- Input is raw LC-MS vendor files that have not been pre-processed with XCMS into a feature table with isotopologue-level resolution
- MS2 spectra are nominal-resolution only and cannot distinguish resolved isotopologues (e.g., M0 vs M6) of the lipid species
- Goal is precursor-level (whole-lipid) isotopologue distribution without per-acyl-chain decomposition

## Inputs

- XCMS-pre-processed feature table from HRMS (LC-MS/MS) lipidomics of a 13C labeling time course
- Label scheme definition (labeled element/tracer, e.g., [U-13C6] glucose) for SIMPEL
- High-resolution MS2 spectra for each resolved PC(36:4) isotopologue (e.g., M0 and M6)
- Measured lipid pool sizes and fatty-acid proportions constituting the PC(36:4) species

## Outputs

- Acyl-chain mass isotopomer distribution (MIDs) table after natural-abundance correction (MIDs_table_NA_corrected.csv), expressed as natom equivalents of 13C per acyl chain (18:1, 18:2, 18:3)
- Pair-summed chain-combination weights (M0/M6, M3/M3, ... M6/M0) reconciling precursor and fragment labeling

## How to apply

Start from the XCMS-pre-processed feature table and build the unscaled isotopologue distribution with MIDsTableNoScale(XCMS_data, label_scheme). For each PC(36:4) isotopologue, inspect the high-resolution MS2 spectra to quantify labeling within the acyl-chain fragments and compute the pair-summed weights (M0/M6, M3/M3, ... M6/M0) for each 18:1/18:2/18:3 chain combination. Apply natural-abundance correction to the acyl-chain MIDs table with NACorrectionFxnII(MIDs_table, label_scheme, output), which wraps IsoCorrectoR. Convert corrected fractional enrichments to natom equivalents of 13C by scaling with the measured lipid pool sizes and the proportions of fatty acids constituting PC(36:4). Finally, verify that the summed pair weights reproduce the precursor isotopologue M6 fraction (conservation check) and export the corrected natom-equivalent table as MIDs_table_NA_corrected.csv, reproducing the pattern of Fig. 4 and Supplementary Data 1 Tables 8-9.

## Related tools

- **SIMPEL** (R package providing MIDsTableNoScale and NACorrectionFxnII to build unscaled isotopologue distributions and perform natural-abundance correction of acyl-chain MIDs) — github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processing of acquired HRMS lipidomics data into the feature table consumed by SIMPEL)
- **IsoCorrectoR** (Wrapped by NACorrectionFxnII to correct acyl-chain MIDs for natural abundance)

## Examples

```
MIDs_table <- MIDsTableNoScale(XCMS_data, label_scheme); corrected <- NACorrectionFxnII(MIDs_table, label_scheme, output)
```

## Evaluation signals

- Conservation check: pair-summed acyl-chain weights (M0/M6 through M6/M0) must reproduce the precursor isotopologue M6 fraction of PC(36:4)
- Corrected chain-level enrichments must be expressible as natom equivalents consistent with the measured lipid pool sizes and FA proportions
- Exported MIDs_table_NA_corrected.csv should reproduce the labeling patterns shown in Fig. 4 and Supplementary Data 1 Tables 8-9
- Natural-abundance correction via NACorrectionFxnII (IsoCorrectoR) should run without errors against the label_scheme, indicating the MIDs table schema is intact
- MS2 fragment labeling must be assignable for resolved isotopologues (e.g., distinct M0 and M6 spectra), not averaged across co-eluting species

## Limitations

- Requires high-resolution MS2 spectra of individually resolved isotopologues; nominal-resolution data cannot distinguish the needed labeling states
- The decomposition depends on accurate measured lipid pool sizes and fatty-acid proportions for the PC(36:4) species; errors propagate into natom-equivalent enrichments
- No software versions reported for SIMPEL, XCMS, or IsoCorrectoR in the context, so exact reproducibility of correction behavior may vary across versions

## Evidence

- [results] MS2-based procedure for PC(36:4) acyl-chain labeling: "MS2 spectra for each of the isotopologues were used to determine the distribution of FA labeling within the acyl chains of PC(36:4)"
- [methods] Natural-abundance correction function in SIMPEL methods: "NACorrectionFxnII(MIDs_table, label_scheme, output)"
- [methods] Unscaled MIDs table construction function: "MIDsTableNoScale(XCMS_data, label_scheme)"
- [intro] XCMS pre-processing before SIMPEL post-processing: "Acquired data were first pre-processed using XCMS, and post-processing data analysis was performed using SIMPEL."
- [intro] SIMPEL automates isotopologue generation and NA correction: "SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections"
