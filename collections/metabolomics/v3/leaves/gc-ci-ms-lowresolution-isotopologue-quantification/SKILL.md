---
name: gc-ci-ms-lowresolution-isotopologue-quantification
description: Use when processing low-resolution GC-CI-MS (e.g., QqQ or SIM-mode quadrupole)
  mzML files for stable-isotope labeling experiments and you need per-isotopologue
  peak areas for a compound table, as described in the isoSCAN package.
when_to_use_negative:
- The input data are high-resolution (qTOF, Orbitrap) full-scan spectra — the HR branch
  of isoSCAN uses a different mass-difference/isotopologue extraction strategy
- The data are already isotopologue-corrected area tables and the task is downstream
  natural-isotope-abundance correction rather than peak integration
- Acquisition was in targeted SIM mode with pre-programmed transition lists rather
  than the profile/scan mode expected by autoQ's low-resolution integration
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: isoSCAN
  role: R package whose autoQ function performs low-resolution isotopologue extraction
    from GC-CI-MS mzML files
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime language in which isoSCAN is implemented and autoQ is invoked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:71ff6fc945246dc0dfa7d1ea6386fdde7bb5fb0af4cbed297469baeb3a13f576
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gc-ci-ms-lowresolution-isotopologue-quantification

## Summary

Quantify isotopologues of targeted metabolites from low-resolution (quadrupole) GC-CI-MS profile-format files (mzML) using the isoSCAN R package's autoQ function, integrating area and maxo per isotopologue. This is the LRMS branch of the isoSCAN workflow, where isotopologue masses are computed by adding the 13C mass difference (1.003355 Da) repeatedly up to the maximum number of atoms.

## When to use

Use when processing low-resolution GC-CI-MS (e.g., QqQ or SIM-mode quadrupole) mzML files for stable-isotope labeling experiments and you need per-isotopologue peak areas for a compound table, as described in the isoSCAN package.

## When NOT to use

- The input data are high-resolution (qTOF, Orbitrap) full-scan spectra — the HR branch of isoSCAN uses a different mass-difference/isotopologue extraction strategy
- The data are already isotopologue-corrected area tables and the task is downstream natural-isotope-abundance correction rather than peak integration
- Acquisition was in targeted SIM mode with pre-programmed transition lists rather than the profile/scan mode expected by autoQ's low-resolution integration

## Inputs

- Low-resolution GC-CI-MS chromatogram files in profile format, e.g., mzML
- A formula table such as `formulaTable_lowres` listing compounds, formulas, and retention times (filtered to the Quadrupole instrument rows)

## Outputs

- A CSV table of isotopologue quantification, with columns of integrated peak `area` and `maxo` intensity per isotopologue per compound

## How to apply

Load the low-resolution example mzML files from the isoSCAN package's extdata folder together with the bundled `formulaTable_lowres` table, filtering it to rows for a Quadrupole instrument (e.g., the Agilent 7890A GC / 7000 QqQ used in the paper). Call `isoSCAN::autoQ` with the mzML file list and formula table; autoQ computes each isotopologue's m/z by adding the enrichment isotope mass difference (1.003355 Da for 13C) as many times as the maximum number of atoms in the compound, then integrates the chromatographic peak for each isotopologue. Set the LR quantification parameters to resolution=1, minscans=6, SNR=3, mzerror=0.1, RTwin=5, maxwidth=4, minwidth=1, and massdiff=1.003355, which define the integration window and mass difference between carbon isotopes. The function returns a table of integrated area and maxo intensity values per isotopologue per compound, which is saved as a CSV for downstream correction/export.

## Related tools

- **isoSCAN** (R package whose autoQ function performs low-resolution isotopologue extraction from GC-CI-MS mzML files) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime language in which isoSCAN is implemented and autoQ is invoked)

## Examples

```
autoQ(files = files, formulaTable = formulaTable_lowres, resolution = 1, minscans = 6, SNR = 3, mzerror = 0.1, RTwin = 5, maxwidth = 4, minwidth = 1, massdiff = 1.003355)
```

## Evaluation signals

- Output table is generated with an `area` and a `maxo` column for each detected isotopologue per compound, consistent with the isoSCAN autoQ expected structure
- The number of isotopologue columns per compound matches the maximum number of atoms from the formula table (massdiff 1.003355 added successively)
- Integration succeeds on the bundled low-resolution extdata mzML files without m/z or RT errors exceeding the mzerror=0.1 and RTwin=5 settings
- Each isotopologue peak passes the SNR=3 and minscans=6 thresholds before being reported

## Limitations

- Limited to low-resolution (quadrupole) profile data; high-resolution instruments require a different configuration
- Parameters like massdiff=1.003355 assume a 13C enrichment isotope; other isotope labels may need a different massdiff
- The vignette specifies `resolution=1`, `minscans=6`, `SNR=3`, `mzerror=0.1`, `RTwin=5`, `maxwidth=4`, `minwidth=1` — changing them may need re-optimization for the specific instrument/compound set
- Validation of the resulting area/maxo table relies on the autoQ expected output structure; no independent isotopologue cross-check is performed by autoQ itself

## Evidence

- [results] autoQ computes isotopologue masses by adding the enrichment isotope mass difference (i.e., 1.003355 for 13C) as many times as the maximum number of atoms, and returns area and maxo per isotopologue.: [evidence span withheld — anchor status: not_found]
- [methods] LRMS isotopologue acquisition in SIM and NS modes: [evidence span withheld — anchor status: not_found]
- [results] isoSCAN, an open-source R-based computational tool: [evidence span withheld — anchor status: not_found]
- [readme] R Package to automatically process GC-CI-MS data for stable isotope labelling experiments: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
- [methods] a 7890A GC system coupled to a 7000 QqQ mass spectrometer (Agilent Technologies) was used for isotopologue determination: [evidence span withheld — anchor status: not_found]
