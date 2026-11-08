---
name: r-package-batch-data-processing
description: Use when you have GC-CI-MS mzML files from 13C stable-isotope-labeling
  experiments and need to quantify isotopologue peak areas for derivatized metabolite
  formulas across a batch of samples.
when_to_use_negative:
- Data were acquired with electron ionization (EI) rather than chemical ionization
  with isobutane — isoSCAN is built for the intact [M+H]+ isotopic envelopes produced
  by CI-isobutane, not fragmented EI spectra.
- The goal is de novo metabolite identification or peak picking without a predefined
  target formula list — autoQ requires a formula table of targeted compounds.
- Input files are not in mzML format (e.g. vendor raw or NetCDF) without prior conversion.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: isoSCAN
  role: R package whose autoQ function performs the batch isotopologue extraction
    and integration
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment in which isoSCAN is installed and invoked
- name: enviPat
  role: Isotope-distribution library used by autoQ to predict theoretical isotopologue
    m/z values for high-resolution data
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:0a79fb146c1885101fbc8e50a23c0d058b072811d32278951a033922ca3233c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# r-package-batch-data-processing

## Summary

Batch-process GC-CI-MS stable-isotope-labeling chromatograms in R using the isoSCAN package, which automatically integrates all isotopologues of targeted metabolites from both low-resolution (QqQ/SIM) and high-resolution (GC-Orbitrap/qTOF) mzML files.

## When to use

Use when you have GC-CI-MS mzML files from 13C stable-isotope-labeling experiments and need to quantify isotopologue peak areas for derivatized metabolite formulas across a batch of samples.

## When NOT to use

- Data were acquired with electron ionization (EI) rather than chemical ionization with isobutane — isoSCAN is built for the intact [M+H]+ isotopic envelopes produced by CI-isobutane, not fragmented EI spectra.
- The goal is de novo metabolite identification or peak picking without a predefined target formula list — autoQ requires a formula table of targeted compounds.
- Input files are not in mzML format (e.g. vendor raw or NetCDF) without prior conversion.

## Inputs

- List of GC-CI-MS data files in mzML format
- Formula table with targeted compounds and their derivatized chemical formulas (formulaTable_lowres for QqQ, formulaTable_orbi for Orbitrap)
- enviPat isotope table (shipped with isoSCAN) for high-resolution mode

## Outputs

- Data frame with integrated area and maxo intensity per isotopologue per compound (low-resolution mode)
- Data frame with predicted-isotopologue m/z, ppm error, area and maxo per isotopologue per compound (high-resolution mode)
- CSV export of the isotopologue quantification table

## How to apply

Install isoSCAN from GitHub (devtools::install_github("jcapelladesto/isoSCAN")) and supply a formula table listing targeted compounds with their derivatized formulas, plus the mzML file list. For low-resolution QqQ data, set resolution=1, minscans=6, SNR=3, mzerror=0.1, RTwin=5, maxwidth=4, minwidth=1, and massdiff=1.003355 so autoQ computes isotopologue masses by adding the 13C mass difference repeatedly up to the maximum atom count, returning area and maxo per isotopologue. For high-resolution Orbitrap centroided data, set resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1, labelatom="13C", and pass the enviPat isotope table so autoQ predicts and merges theoretical isotopic distributions — this is needed because HR spectra contain less-intense Si, N or O isotopic peaks. Each file is processed independently, and the result is a data frame of integrated isotopologue intensities per compound that can be written to CSV.

## Related tools

- **isoSCAN** (R package whose autoQ function performs the batch isotopologue extraction and integration) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment in which isoSCAN is installed and invoked)
- **enviPat** (Isotope-distribution library used by autoQ to predict theoretical isotopologue m/z values for high-resolution data)

## Examples

```
library(isoSCAN); res <- autoQ(mzml_files, formulaTable_orbi, isotopes, resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1, labelatom="13C")
```

## Evaluation signals

- Output table contains an area and a maxo column for each detected isotopologue per compound.
- High-resolution output additionally carries predicted-isotopologue m/z and ppm error columns populated for each derivatized Orbitrap formula.
- Run completes without error across all mzML files in the batch, confirming each file is processed independently.
- Predicted m/z values deviate from observed centroids by no more than the configured maxppm (5 ppm in Orbitrap mode).

## Limitations

- Requires the user to supply a formula table with correct derivatized formulas; metabolites absent from the table are not quantified.
- High-resolution mode depends on enviPat-predicted distributions and an estimated mass resolution (resolution=c(6e4,200)); mis-set resolution degrades isotopologue merging.
- Intended for GC-CI-MS stable-isotope labeling data; other ionization modes or LC-MS acquisition are outside its scope.

## Evidence

- [intro] R package autoQ isotopologue quantification: [evidence span withheld — anchor status: not_found]
- [results] open-source repository: [evidence span withheld — anchor status: not_found]
- [other] low-resolution parameter set: [evidence span withheld — anchor status: not_found]
- [other] enviPat rationale: [evidence span withheld — anchor status: not_found]
- [abstract] isobutane CI advantage: [evidence span withheld — anchor status: not_found]
