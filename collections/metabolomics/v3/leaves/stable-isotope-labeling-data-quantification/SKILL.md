---
name: stable-isotope-labeling-data-quantification
description: Use when GC-CI-MS raw centroided files from a stable-isotope labelling
  study (e.g. 13C-glucose or 13C-arginine tracing) need automatic isotopologue quantification
  for derivatized central-carbon or polyamine metabolites.
when_to_use_negative:
- 'When the acquisition is EI rather than chemical ionization: EI at 70 eV fragments
  compounds so the [M+H]+ envelope needed for isotopologue integration is not concentrated
  in a single ion.'
- When the input is already a processed isotopologue/enrichment table rather than
  raw centroided mzML files.
- When the isotope label is not the atom specified in the formula table (e.g. labelatom
  must match the tracer; 15N tracing requires labelatom="15N").
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: isoSCAN
  role: R package providing the autoQ function that automatically quantifies all isotopologues
    from GC-CI-MS data.
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment in which isoSCAN is installed and the autoQ call is executed.
- name: enviPat
  role: Used by autoQ in high-resolution mode to calculate and merge the theoretical
    isotopic distribution of a formula at a given mass resolution.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:6271cfd2b62a501529d51a8af4789513d25cd76785054e620ee38e68fe32d7e2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# stable-isotope-labeling-data-quantification

## Summary

Automatically quantifies all isotopologues of derivatized metabolites in GC-CI-MS stable-isotope labelling experiments using the R package isoSCAN, handling both low-resolution (LRMS) and high-resolution (HRMS) centroided data.

## When to use

Use when GC-CI-MS raw centroided files from a stable-isotope labelling study (e.g. 13C-glucose or 13C-arginine tracing) need automatic isotopologue quantification for derivatized central-carbon or polyamine metabolites.

## When NOT to use

- When the acquisition is EI rather than chemical ionization: EI at 70 eV fragments compounds so the [M+H]+ envelope needed for isotopologue integration is not concentrated in a single ion.
- When the input is already a processed isotopologue/enrichment table rather than raw centroided mzML files.
- When the isotope label is not the atom specified in the formula table (e.g. labelatom must match the tracer; 15N tracing requires labelatom="15N").

## Inputs

- centroided mzML raw files (one per GC-CI-MS acquisition, LRMS or HRMS)
- formulaTable with derivatized metabolite formulas (e.g. formulaTable_orbi rows for Orbitrap data)
- enviPat isotope table shipped with the isoSCAN package
- label atom identifier (e.g. labelatom="13C")

## Outputs

- data frame of integrated isotopologue abundances per sample with predicted m/z and ppm-error columns
- CSV export of the isotopologue integration table (e.g. autoQ_orbi_results.csv)

## How to apply

Install isoSCAN from GitHub (jcapelladesto/isoSCAN) with devtools::install_github. Feed each centroided mzML file, along with a formula table of the derivatized metabolites and the enviPat isotope table, to isoSCAN::autoQ, which processes each file independently. For low-resolution (QqQ or qTOF) data autoQ targets the [M+H]+ isotopic envelope in SIM or narrow-window scan mode; for high-resolution Orbitrap data it predicts theoretical isotopic distributions with enviPat at the estimated mass resolution (e.g. resolution=c(6e4,200)) and integrates isotopologue peak groups with parameters such as minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1, and labelatom="13C". The returned data frame contains integrated isotopologue abundances together with predicted m/z and ppm-error columns, which should be written to a CSV (e.g. autoQ_orbi_results.csv) for downstream enrichment analysis.

## Related tools

- **isoSCAN** (R package providing the autoQ function that automatically quantifies all isotopologues from GC-CI-MS data.) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment in which isoSCAN is installed and the autoQ call is executed.)
- **enviPat** (Used by autoQ in high-resolution mode to calculate and merge the theoretical isotopic distribution of a formula at a given mass resolution.)

## Examples

```
Rscript -e 'library(isoSCAN); res <- autoQ(mzml_files, formulaTable_orbi, isotopes, resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1, labelatom="13C"); write.csv(res, "autoQ_orbi_results.csv")'
```

## Evaluation signals

- The output table contains the expected isotopologue rows for the derivatized formulas, with predicted m/z and ppm columns populated.
- ppm error of each integrated isotopologue is within the maxppm tolerance (e.g. 5 ppm for Orbitrap runs).
- Each integrated isotopologue peak group spans at least minscans (e.g. 6) scans and meets the SNR threshold (e.g. 5).
- Retention-time window of each integration falls within RTwin (e.g. 5 s) and peak width within minwidth–maxwidth bounds (e.g. 1–4).
- Running autoQ independently on replicate files yields consistent isotopologue counts for the same formula set.

## Limitations

- High-resolution spectra contain less-intense isotopic peaks from Si, N or O, which is why enviPat-predicted envelopes are required; low-abundance isotopologues may still fall below the SNR threshold.
- autoQ assumes chemical ionization data where the isotopologue signal is concentrated in the [M+H]+ envelope; EI-mode data are not suitable.
- The R package README does not document parameter defaults or a version; users must rely on the article vignette for settings.

## Evidence

- [results] isobutane CI concentrates signal in [M+H]+ envelope: [evidence span withheld — anchor status: not_found]
- [methods] high-resolution autoQ uses enviPat to predict isotopologue m/z: [evidence span withheld — anchor status: not_found]
- [results] isoSCAN repository URL: [evidence span withheld — anchor status: not_found]
- [results] EI causes extensive fragmentation, motivating CI: [evidence span withheld — anchor status: not_found]
