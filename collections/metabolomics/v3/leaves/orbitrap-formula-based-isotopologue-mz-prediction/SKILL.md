---
name: orbitrap-formula-based-isotopologue-mz-prediction
description: Use when processing centroided high-resolution GC-CI-MS (GC-Orbitrap)
  mzML files for 13C stable-isotope labeling, where derivatized-metabolite formulas
  are known but the isotopologue peak series must be predicted and matched before
  quantification.
when_to_use_negative:
- Input is low-resolution GC-CI-MS data acquired in SIM or narrow-window scan (NS)
  mode; the skill's enviPat-based merging is specific to high-resolution spectra where
  Si/N/O isotopic satellites are resolved
- The derivatized formula table is unknown or unavailable, since the theoretical envelope
  cannot be computed without a formula
- Labeling is performed with an atom other than 13C, unless labelatom is changed accordingly
  and the theory still holds
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: isoSCAN
  role: R package whose autoQ() function invokes enviPat to calculate and merge theoretical
    isotopic distributions for each derivatized formula in Orbitrap mode, yielding
    predicted isotopologue m/z and ppm error columns
  repo: https://github.com/jcapelladesto/isoSCAN
- name: enviPat
  role: Underlying library used inside autoQ to compute the theoretical isotopic distribution
    of a formula given an estimated mass resolution and merge its peaks into target
    isotopologue clusters
- name: R
  role: Runtime environment in which isoSCAN and its bundled formulaTable_orbi / enviPat
    isotope tables are loaded and executed
- name: GC-Orbitrap
  role: High-resolution instrument platform whose spectra contain the resolved Si/N/O
    isotopic satellites that motivate formula-based prediction and merging
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:5ba618df8a0f050595e718681b073c1c916662206dec1dd0725dd24af2f53045
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - GC-Orbitrap
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# orbitrap-formula-based-isotopologue-mz-prediction

## Summary

Predicts theoretical isotopologue m/z clusters for derivatized metabolite formulas in high-resolution GC-CI-MS (Orbitrap) tracing data by merging enviPat-predicted isotopic envelopes at an instrument resolution (e.g. 200,000), so that individual 13C-labeled isotopologue peaks from Si, N or O can be located and quantified.

## When to use

Use when processing centroided high-resolution GC-CI-MS (GC-Orbitrap) mzML files for 13C stable-isotope labeling, where derivatized-metabolite formulas are known but the isotopologue peak series must be predicted and matched before quantification.

## When NOT to use

- Input is low-resolution GC-CI-MS data acquired in SIM or narrow-window scan (NS) mode; the skill's enviPat-based merging is specific to high-resolution spectra where Si/N/O isotopic satellites are resolved
- The derivatized formula table is unknown or unavailable, since the theoretical envelope cannot be computed without a formula
- Labeling is performed with an atom other than 13C, unless labelatom is changed accordingly and the theory still holds

## Inputs

- High-resolution (Orbitrap) centroided mzML files from GC-CI-MS with isobutane reagent gas
- formulaTable_orbi: table of derivatized metabolite formulas (Orbitrap-derivatized rows) bundled with isoSCAN
- enviPat isotope table shipped with the isoSCAN package
- Estimated mass resolution of the GC-Orbitrap instrument (e.g. resolution=c(6e4,200))
- Label atom specification (labelatom="13C")

## Outputs

- Data frame of predicted isotopologue m/z values and measured ppm error per formula/scan
- autoQ_orbi_results.csv: table of integrated isotopologue groups with predicted m/z and ppm columns

## How to apply

Load the derivatized-metabolite formula table (formulaTable_orbi) shipped with isoSCAN together with the enviPat isotope table bundled in the package, and pass the sample's estimated Orbitrap mass resolution to enviPat so it can calculate and merge the theoretical isotopic distribution for each formula into a single peak cluster. This merging is required because high-resolution spectra of derivatized GC-CI-MS metabolites contain additional less-intense isotopic peaks arising from Si, N or O atoms in the silylated/methoxyaminated derivatives (MSTFA + 1% TMCS derivatization), which would otherwise be missed if only the monoisotopic [M+H]+ series were targeted. Call isoSCAN::autoQ on the orbi mzML files with formulaTable_orbi, the isotopes table, resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1 and labelatom="13C"; autoQ uses enviPat internally to compute predicted isotopologue m/z values and merge them into target groups per chromatographic file. Verify the output data frame: each isotopologue row must carry the predicted m/z and the ppm error columns, populated for the derivatized Orbitrap formulas, confirming the prediction step succeeded before downstream isotopologue integration is trusted.

## Related tools

- **isoSCAN** (R package whose autoQ() function invokes enviPat to calculate and merge theoretical isotopic distributions for each derivatized formula in Orbitrap mode, yielding predicted isotopologue m/z and ppm error columns) — https://github.com/jcapelladesto/isoSCAN
- **enviPat** (Underlying library used inside autoQ to compute the theoretical isotopic distribution of a formula given an estimated mass resolution and merge its peaks into target isotopologue clusters)
- **R** (Runtime environment in which isoSCAN and its bundled formulaTable_orbi / enviPat isotope tables are loaded and executed)
- **GC-Orbitrap** (High-resolution instrument platform whose spectra contain the resolved Si/N/O isotopic satellites that motivate formula-based prediction and merging)

## Examples

```
library(isoSCAN); res <- isoSCAN::autoQ(orbi_mzml_files, formulaTable_orbi, isotopes, resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1, labelatom="13C")
```

## Evaluation signals

- The returned data frame from autoQ on orbi mzML files contains explicit predicted-isotopologue m/z and ppm error columns for each derivatized formula
- Running the function without errors on the bundled orbi sample mzML files with resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1 and labelatom="13C" produces a non-empty autoQ_orbi_results.csv
- Predicted m/z values for the derivatized Orbitrap formulas differ from the monoisotopic [M+H]+ by the expected +1 Da (or mass-defect-corrected) increments reflecting merged Si/N/O satellite peaks
- PPM error column values fall within the maxppm=5 tolerance used for matching, confirming the theoretical envelope aligns with observed peaks

## Limitations

- Prediction quality depends on an accurate estimated mass resolution; the article bundled resolution=c(6e4,200) for the GC-Orbitrap configuration and other instruments may require retuning this parameter
- Additional less-intense isotopic peaks arise from Si, N or O in derivatized metabolites; if the formula table does not reflect the full derivatized structure (e.g. missing silyl groups from MSTFA + 1% TMCS derivatization), predicted envelopes will be wrong
- The skill is specific to centroided high-resolution GC-CI-MS data; applying it to low-resolution SIM/NS data will not resolve the Si/N/O satellites the merging step is designed to handle

## Evidence

- [results] For high-resolution data, autoQ uses enviPat functionalities to calculate and merge the theoretical isotopic distribution of a formula given an estimated mass resolution: [evidence span withheld — anchor status: not_found]
- [results] which is needed because spectra contain less-intense isotopic peaks from Si, N or O.: [evidence span withheld — anchor status: not_found]
- [intro] we have developed an R-package called isoSCAN capable of processing GC- CI MS data from LRMS and HRMS stable-isotope labeling studies.: [evidence span withheld — anchor status: not_found]
- [readme] R Package to automatically process GC-CI-MS data for stable isotope labelling experiments: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
