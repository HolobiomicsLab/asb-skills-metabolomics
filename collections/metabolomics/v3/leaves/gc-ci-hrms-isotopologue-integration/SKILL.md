---
name: gc-ci-hrms-isotopologue-integration
description: Use when the input is centroided GC-CI-MS data acquired with isobutane
  reagent gas (QqQ SIM/NS, GC-qTOF, or GC-Orbitrap mzML) from 13C stable-isotope labeling
  experiments and all isotopologues of derivatized metabolites need to be quantified.
when_to_use_negative:
- Input data are EI spectra (70 eV or low-energy EI) rather than CI-isobutane spectra,
  where [M+H]+ concentration is low and fragmentation is extensive
- Data are LC-MS or direct-infusion MS without chromatographic GC separation, since
  isoSCAN processes GC-CI-MS traces
- Files are not centroided mzML; the package vignette workflow assumes centroided
  GC-MS data
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: isoSCAN
  role: R package providing isoSCAN::autoQ, which targets and quantifies all isotopologues
    in GC-CI-MS LRMS/HRMS data, integrating isotope envelopes per file using enviPat-predicted
    distributions
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Execution environment in which the isoSCAN package is installed via devtools
    and autoQ is called on the mzML samples
  repo: https://github.com/jcapelladesto/isoSCAN
- name: enviPat
  role: Underlying isotope-pattern engine used by autoQ to calculate and merge theoretical
    isotopic distributions of formulas at a given mass resolution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:82eaf31287602d2b7d9272813bb62d41f25bd5945ceb74fcced339623b8b8c8c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# gc-ci-hrms-isotopologue-integration

## Summary

Applies the isoSCAN R package to automatically quantify isotopologues in GC-CI-MS stable-isotope labeling data from low-resolution (QqQ) and high-resolution (qTOF/Orbitrap) centroided files, integrating isotope envelopes using predicted formulas and reagent-gas-derived [M+H]+ ions.

## When to use

Use when the input is centroided GC-CI-MS data acquired with isobutane reagent gas (QqQ SIM/NS, GC-qTOF, or GC-Orbitrap mzML) from 13C stable-isotope labeling experiments and all isotopologues of derivatized metabolites need to be quantified.

## When NOT to use

- Input data are EI spectra (70 eV or low-energy EI) rather than CI-isobutane spectra, where [M+H]+ concentration is low and fragmentation is extensive
- Data are LC-MS or direct-infusion MS without chromatographic GC separation, since isoSCAN processes GC-CI-MS traces
- Files are not centroided mzML; the package vignette workflow assumes centroided GC-MS data

## Inputs

- GC-CI-MS centroided mzML files from LRMS (QqQ) or HRMS (GC-qTOF, GC-Orbitrap) acquisitions
- Formula table of derivatized metabolites (e.g. formulaTable_orbi for Orbitrap derivatives)
- enviPat isotope table shipped with the isoSCAN package
- Label atom specification (e.g. "13C")

## Outputs

- Data frame of isotopologue integrations with predicted isotopologue m/z and ppm error columns per sample
- Exported CSV table (e.g. autoQ_orbi_results.csv)

## How to apply

Load the isoSCAN package in R and run isoSCAN::autoQ on the centroided mzML files together with a formula table (e.g. formulaTable_orbi for derivatized Orbitrap analytes) and the enviPat isotope table shipped with the package. For high-resolution data autoQ uses enviPat to calculate and merge the theoretical isotopic distribution given an estimated mass resolution, which is needed because spectra contain less-intense isotopic peaks from Si, N or O; pass labelatom="13C" to define the isotope trace. Set targeted-quality parameters such as resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1 so that isotopologue peak groups are integrated per file independently. Rationale: CI with isobutane concentrates metabolite abundance in the [M+H]+ envelope, making those ions the correct targets for isotopologue integration. Confirm the returned data frame contains predicted-isotopologue m/z and ppm error columns before exporting the table.

## Related tools

- **isoSCAN** (R package providing isoSCAN::autoQ, which targets and quantifies all isotopologues in GC-CI-MS LRMS/HRMS data, integrating isotope envelopes per file using enviPat-predicted distributions) — https://github.com/jcapelladesto/isoSCAN
- **R** (Execution environment in which the isoSCAN package is installed via devtools and autoQ is called on the mzML samples) — https://github.com/jcapelladesto/isoSCAN
- **enviPat** (Underlying isotope-pattern engine used by autoQ to calculate and merge theoretical isotopic distributions of formulas at a given mass resolution)

## Examples

```
Rscript -e 'library(isoSCAN); res <- autoQ(files, formulaTable_orbi, isotopes, resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1, labelatom="13C"); write.csv(res, "autoQ_orbi_results.csv", row.names=FALSE)'
```

## Evaluation signals

- The autoQ run completes without errors and the returned data frame contains the expected isotopologue rows for the derivatized formulas
- Predicted-isotopologue m/z and ppm error columns are present and populated in the output table
- Reported ppm errors fall within the maxppm tolerance (5 ppm) set for HRMS integrations
- Each isotopologue group is integrated from at least minscans=6 data points with signal-to-noise ratio above SNR=5
- Exported CSV (e.g. autoQ_orbi_results.csv) contains one row per isotopologue per sample and can be re-loaded into R

## Limitations

- Isotopologue detection relies on the [M+H]+ envelope from isobutane CI; methane reagent gas induces unwanted fragmentation of the [M+H]+ ion and yields lower signal (~5-fold less intense)
- For high-resolution spectra, isotopic peaks from Si, N or O can be less intense, requiring enviPat-based prediction and merging rather than direct peak picking
- isoSCAN parameters such as resolution, minscans, SNR, maxppm, RTwin and min/max width must be tuned per instrument configuration; defaults are not universal

## Evidence

- [results] isoSCAN::autoQ targets and quantifies isotopologues in GC-MS stable-isotope labeling data, processing each file independently: [evidence span withheld — anchor status: not_found]
- [results] CI with isobutane concentrates metabolite abundance in the [M+H]+ envelope, making those ions the correct targets for isotopologue integration: [evidence span withheld — anchor status: not_found]
- [results] isoSCAN is an open-source R-based computational tool: [evidence span withheld — anchor status: not_found]
- [readme] The tool processes GC-CI-MS data for stable isotope labelling experiments: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
