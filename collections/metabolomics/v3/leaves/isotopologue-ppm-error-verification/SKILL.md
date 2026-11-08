---
name: isotopologue-ppm-error-verification
description: Use when processing high-resolution (Orbitrap) centroided GC-MS mzML
  files with isoSCAN autoQ in stable-isotope labeling experiments and you must confirm
  that the integrated isotopologue peaks correspond to the predicted m/z values within
  the allowed mass error before reporting enrichment.
when_to_use_negative:
- Low-resolution GC-CI-MS data processed in SIM or NS modes, where isotopologue separation
  relies on unit-mass resolution rather than ppm error against an enviPat-predicted
  envelope
- Spectra produced by hard electron ionization (EI at 70 eV), which fragment compound
  structures and prevent assignment of a coherent [M+H]+ isotopic envelope
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: isoSCAN
  role: Runs autoQ on high-resolution GC-MS mzML files and reports predicted-isotopologue
    m/z and ppm error columns for verification
  repo: https://github.com/jcapelladesto/isoSCAN
- name: enviPat
  role: Calculates and merges the theoretical isotopic distribution of a formula at
    estimated mass resolution, providing the predicted m/z values against which ppm
    errors are checked
- name: R
  role: Language in which isoSCAN is implemented and invoked
- name: GC-Orbitrap
  role: High-resolution instrument platform whose centroided mzML data are processed
    and whose mass accuracy supports ppm-level isotopologue verification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:1be034a8662b3742a3fc1b30e29e634e07c0a5892a4958913d4590e1221545ce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - GC-Orbitrap
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# isotopologue-ppm-error-verification

## Summary

Verifies that autoQ-integrated isotopologues from high-resolution GC-CI-MS (Orbitrap/qTOF) data match the enviPat-predicted theoretical isotopic distribution by checking the predicted m/z and ppm error columns reported for each isotopologue.

## When to use

Use when processing high-resolution (Orbitrap) centroided GC-MS mzML files with isoSCAN autoQ in stable-isotope labeling experiments and you must confirm that the integrated isotopologue peaks correspond to the predicted m/z values within the allowed mass error before reporting enrichment.

## When NOT to use

- Low-resolution GC-CI-MS data processed in SIM or NS modes, where isotopologue separation relies on unit-mass resolution rather than ppm error against an enviPat-predicted envelope
- Spectra produced by hard electron ionization (EI at 70 eV), which fragment compound structures and prevent assignment of a coherent [M+H]+ isotopic envelope

## Inputs

- autoQ result data frame from high-resolution (Orbitrap or qTOF) centroided mzML files, containing per-isotopologue integrated intensities
- formulaTable_orbi rows for derivatized Orbitrap formulas from the isoSCAN package
- enviPat isotope table shipped with the isoSCAN package
- Acquisition parameter settings used in the autoQ call (resolution, maxppm, minscans, SNR, RTwin, maxwidth, minwidth)

## Outputs

- Verified isotopologue table with predicted-isotopologue m/z and ppm error columns populated for each derivatized formula
- Flagged or filtered isotopologue rows whose ppm error exceeds the configured maxppm tolerance
- Final CSV output (e.g. autoQ_orbi_results.csv) containing validated isotopologue integrations

## How to apply

After running isoSCAN::autoQ on high-resolution mzML files with formulaTable_orbi, the enviPat isotope table, and labelatom='13C', inspect the returned data frame for the predicted-isotopologue m/z and ppm error columns. The theoretical isotopic distribution is calculated and merged by enviPat using an estimated mass resolution (e.g. resolution=c(6e4,200)), because HR spectra contain less-intense isotopic peaks from Si, N or O. Confirm that the ppm error of each integrated isotopologue relative to the enviPat-predicted m/z falls within the acquisition tolerance used in the run (e.g. maxppm=5 in the Orbitrap vignette configuration). Rows with ppm errors exceeding this tolerance indicate mis-integrated or wrongly assigned isotopologues and should be flagged or excluded before computing isotopologue enrichment. The workflow is validated when the output table (e.g. autoQ_orbi_results.csv) contains the expected isotopologue rows with predicted m/z and ppm columns populated for the derivatized formulas.

## Related tools

- **isoSCAN** (Runs autoQ on high-resolution GC-MS mzML files and reports predicted-isotopologue m/z and ppm error columns for verification) — https://github.com/jcapelladesto/isoSCAN
- **enviPat** (Calculates and merges the theoretical isotopic distribution of a formula at estimated mass resolution, providing the predicted m/z values against which ppm errors are checked)
- **R** (Language in which isoSCAN is implemented and invoked)
- **GC-Orbitrap** (High-resolution instrument platform whose centroided mzML data are processed and whose mass accuracy supports ppm-level isotopologue verification)

## Examples

```
Rscript -e 'install_github("jcapelladesto/isoSCAN"); library(isoSCAN); res <- autoQ(files, formulaTable_orbi, isotopes, resolution=c(6e4,200), minscans=6, SNR=5, maxppm=5, RTwin=5, maxwidth=4, minwidth=1, labelatom="13C")'
```

## Evaluation signals

- Every isotopologue row in the output table has both predicted-isotopologue m/z and ppm error columns populated (no NULLs).
- Integrated isotopologue ppm errors fall within the configured tolerance (e.g. maxppm=5 used in the Orbitrap vignette run).
- The final CSV (e.g. autoQ_orbi_results.csv) contains the expected isotopologue rows for the derivatized Orbitrap formulas in formulaTable_orbi.
- Rows with ppm errors outside tolerance are flagged or excluded rather than silently retained in downstream enrichment calculations.
- The enviPat-predicted envelope matches the observed isotopic peak pattern including less-intense Si, N, O contributions.

## Limitations

- Verification depends on the resolution estimate passed to enviPat (e.g. resolution=c(6e4,200)); inaccurate resolution settings skew predicted m/z and inflate ppm errors.
- High-resolution spectra contain less-intense isotopic peaks from Si, N or O, so missing or weak isotopologues may not pass SNR thresholds (e.g. SNR=5, minscans=6) and cannot be verified.
- Isobutane CI concentrates abundance in the [M+H]+ envelope; isotopologues outside this envelope are not reliably verifiable.

## Evidence

- [other] For high-resolution data, autoQ uses enviPat functionalities to calculate and merge the theoretical isotopic distribution of a formula given an estimated mass resolution, which is needed because spectra contain less-intense isotopic peaks from Si, N or O.: [evidence span withheld — anchor status: not_found]
- [other] Validation of the autoQ output includes checking predicted-isotopologue m/z and ppm error columns.: [evidence span withheld — anchor status: not_found]
- [readme] The README describes how to install the isoSCAN package that implements autoQ.: "install_github("jcapelladesto/isoSCAN")"
- [results] Isobutane as reagent gas concentrates abundance in the [M+H]+ envelope, supporting isotopologue assignment against the predicted envelope.: [evidence span withheld — anchor status: not_found]
