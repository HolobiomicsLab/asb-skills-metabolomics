---
name: ms2-fragment-isotopologue-quantification
description: Use when HRMS MS2 spectra are available for individual lipid isotopologues
  (e.g., M0 vs M6 of PC(36:4)) and acyl-chain-specific 13C labeling, not just precursor-level
  MID, is required.
when_to_use_negative:
- Data were acquired at nominal m/z resolution, where dual-labeled isotopologues produce
  nominally identical mass shifts and MS2 spectra of individual isotopologues cannot
  be resolved
- Only precursor-level (MS1) isotopologue distributions are needed and acyl-chain-resolved
  labeling is not the analysis goal
- The lipid species lacks distinguishable acyl-chain fragment ions in its MS2 spectra,
  so chain-specific weights cannot be computed
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SIMPEL
  role: R package whose functions (MIDsTableNoScale, NACorrectionFxnII, MIDplot) build
    the isotopologue distribution table, compute acyl-chain pair-summed weights, and
    perform natural-abundance correction
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processes the acquired HRMS lipidomics data into the feature table that
    is imported into SIMPEL
- name: IsoCorrectoR
  role: Performs natural-abundance correction of the acyl-chain MIDs table, wrapped
    by NACorrectionFxnII
- name: R
  role: Execution environment for the SIMPEL package and all post-processing steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:226dbafb74a2f189aa22fa0ff5cd23778356eda0c9f588107232b8b568a05eee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoCorrectoR
  - R
  - SIMPEL
  - XCMS
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ms2-fragment-isotopologue-quantification

## Summary

A SIMPEL-based procedure that uses high-resolution MS2 spectra of resolved lipid isotopologues (e.g., unlabeled M0 vs labeled M6 of PC(36:4)) to quantify the distribution of 13C labeling within individual fatty acyl chains (18:1, 18:2, 18:3), reporting enrichment as natom equivalents after natural-abundance correction.

## When to use

Use when HRMS MS2 spectra are available for individual lipid isotopologues (e.g., M0 vs M6 of PC(36:4)) and acyl-chain-specific 13C labeling, not just precursor-level MID, is required.

## When NOT to use

- Data were acquired at nominal m/z resolution, where dual-labeled isotopologues produce nominally identical mass shifts and MS2 spectra of individual isotopologues cannot be resolved
- Only precursor-level (MS1) isotopologue distributions are needed and acyl-chain-resolved labeling is not the analysis goal
- The lipid species lacks distinguishable acyl-chain fragment ions in its MS2 spectra, so chain-specific weights cannot be computed

## Inputs

- XCMS-pre-processed HRMS feature table from a stable-isotope labeling time course (e.g., [U-13C6] glucose labeling of Camelina sativa embryos, 0-32 h)
- High-resolution MS2 spectra acquired for each resolved isotopologue of the target lipid molecular species (e.g., PC(36:4) M0, M3, M6)
- Label scheme definition passed to SIMPEL functions (label_scheme)
- Measured lipid pool sizes for scaling fractional enrichments to natom equivalents

## Outputs

- Acyl-chain MIDs table with pair-summed weights (M0/M6, M3/M3, ..., M6/M0) per chain combination
- Natural-abundance-corrected fractional enrichments per acyl chain (IsoCorrectoR output via NACorrectionFxnII)
- 13C enrichment expressed as natom equivalents per fatty acyl chain
- Exported CSV file: MIDs_table_NA_corrected.csv

## How to apply

Load the XCMS-pre-processed feature table and build the unscaled isotopologue distribution with MIDsTableNoScale(XCMS_data, label_scheme). From the MS2 spectrum of each resolved PC(36:4) isotopologue, quantify labeling in the 18:1, 18:2, and 18:3 acyl-chain fragments and compute the pair-summed weights (M0/M6, M3/M3, ..., M6/M0) for each chain combination; high resolution is essential because dual-labeled isotopologues with nominally identical mass shifts cannot be resolved at nominal m/z. Apply natural-abundance correction to the acyl-chain MIDs table with NACorrectionFxnII(MIDs_table, label_scheme, output), which wraps IsoCorrectoR. Convert corrected fractional enrichments to natom equivalents of 13C by scaling with the measured lipid pool sizes, then check that the summed pair weights conserve the precursor isotopologue M6 fraction. Export the corrected table as MIDs_table_NA_corrected.csv, reproducing the pattern in Fig. 4 and Supplementary Data 1 Tables 8-9.

## Related tools

- **SIMPEL** (R package whose functions (MIDsTableNoScale, NACorrectionFxnII, MIDplot) build the isotopologue distribution table, compute acyl-chain pair-summed weights, and perform natural-abundance correction) — github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processes the acquired HRMS lipidomics data into the feature table that is imported into SIMPEL)
- **IsoCorrectoR** (Performs natural-abundance correction of the acyl-chain MIDs table, wrapped by NACorrectionFxnII)
- **R** (Execution environment for the SIMPEL package and all post-processing steps)

## Examples

```
MIDs_table <- MIDsTableNoScale(XCMS_data, label_scheme); MIDs_table_NA_corrected <- NACorrectionFxnII(MIDs_table, label_scheme, output)
```

## Evaluation signals

- Conservation check: the summed pair weights (M0/M6 + M3/M3 + ... + M6/M0) must reproduce the precursor isotopologue M6 fraction of PC(36:4)
- The exported MIDs_table_NA_corrected.csv reproduces the pattern shown in Fig. 4 and Supplementary Data 1 Tables 8-9
- Isotopologue fractions in the unscaled table sum to 1 per lipid species and time point before correction
- Natural-abundance correction via IsoCorrectoR completes without error and yields corrected fractional enrichments that can be scaled by lipid pool sizes
- Each acyl-chain (18:1, 18:2, 18:3) labeling distribution is derived from MS2 spectra of every resolved isotopologue, not from a single averaged spectrum

## Limitations

- Requires HRMS: dual-labeled isotopologues with nominally identical mass shifts cannot be resolved with nominal-resolution instruments, limiting the approach to high-resolution MS2 data
- No software versions are reported for SIMPEL, XCMS, IsoCorrectoR, or INCA, so reproducibility depends on package availability rather than pinned versions
- Converting fractional enrichments to natom equivalents depends on measured lipid pool sizes; absent pool-size data only fractional enrichments can be reported
- The procedure was demonstrated on PC(36:4) with 18:1/18:2/18:3 chains; other lipid classes may require adapted fragment-quantification logic

## Evidence

- [results] MS2 spectra determine acyl-chain labeling: "MS2 spectra for each of the isotopologues were used to determine the distribution of FA labeling within the acyl chains of PC(36:4)"
- [intro] SIMPEL automates isotopologue generation and NA correction: "SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections"
- [methods] MIDsTableNoScale function signature: "MIDsTableNoScale(XCMS_data, label_scheme)"
- [methods] NACorrectionFxnII function signature: "NACorrectionFxnII(MIDs_table, label_scheme, output)"
- [intro] HRMS resolution requirement: "High resolution mass spectrometry (HRMS) can distinguish metabolites (and isotopologues of heteroatoms) through accurate measurement of m/z values that differ by <0.01 Da"
- [abstract] SIMPEL paired with lipidomics: "SIMPEL was paired with HRMS-based lipidomics data to describe lipid metabolism based on a single labeling experiment"
