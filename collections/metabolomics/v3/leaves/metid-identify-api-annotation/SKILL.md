---
name: metid-identify-api-annotation
description: Use when MALDI-MSI peaks must be annotated for a derivatizing matrix
  such as FMP-10, especially when multiple derivatization sites on one molecule (e.g.
  dopamine 1A/2A/2B forms) must be matched within a defined ppm tolerance.
when_to_use_negative:
- The dataset was acquired with a conventional, non-derivatizing matrix and only standard
  [M+H]+/[M-H]- adducts need annotation — general LC-MS annotation tools (Metlin,
  GNPS, Mass2adduct) may suffice
- Input is already a curated feature/annotation table rather than raw m/z features
  requiring database-driven identification
- MS2-level confirmation is required for a compound absent from Met-ID's preconfigured
  finite set of bundled MS2 spectra and no user-generated standard spectra are available
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Met-ID
  role: 'Primary identification engine: MetIDServer context manager, identify() mass
    matching, predict_ccs(), and search_by_name() lookups against bundled HMDB/LIPID
    MAPS SQLite databases'
  repo: https://github.com/pbjarterot/Met-ID/releases
- name: RDKit
  role: Computes neutral monoisotopic masses of retrieved metabolite structures to
    derive theoretical [M+H]+ m/z of single- and double-FMP-10-derivatized species
    for cross-checking identify() results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:819b41ddda317dd6e5ace92f2069dca2f058a969e70f302d1df38de1288bffee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  - RDKit
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metid-identify-api-annotation

## Summary

Annotate MALDI-MSI m/z features — including multi-site FMP-10-derivatized metabolites — by driving Met-ID's local server and identify() API against its bundled HMDB/LIPID MAPS SQLite databases, with RDKit-based computation of theoretical derivatized m/z values for validation.

## When to use

Use when MALDI-MSI peaks must be annotated for a derivatizing matrix such as FMP-10, especially when multiple derivatization sites on one molecule (e.g. dopamine 1A/2A/2B forms) must be matched within a defined ppm tolerance.

## When NOT to use

- The dataset was acquired with a conventional, non-derivatizing matrix and only standard [M+H]+/[M-H]- adducts need annotation — general LC-MS annotation tools (Metlin, GNPS, Mass2adduct) may suffice
- Input is already a curated feature/annotation table rather than raw m/z features requiring database-driven identification
- MS2-level confirmation is required for a compound absent from Met-ID's preconfigured finite set of bundled MS2 spectra and no user-generated standard spectra are available

## Inputs

- One or more observed m/z values from a MALDI-MSI experiment (optionally with CCS values)
- Optional user-provided tissue MS2 spectra for similarity comparison against the built-in FMP-10 standards library
- SMILES string of a metabolite for CCS prediction via predict_ccs()

## Outputs

- Annotated metabolite candidates per m/z, matched against HMDB(All)/LIPID MAPS entries for the selected matrix (FMP-10), adducts, and mass_error=5 ppm
- Predicted CCS values from predict_ccs()
- Computed-vs-reported comparison table of theoretical derivatized m/z values with dPPM deviations (e.g. derivatized_dopamine_mz_check.csv)

## How to apply

Start a local Met-ID server by importing MetIDServer and using it as a context manager, then call metid.identify() with an observed m/z (e.g. mz=421.1911, mzwindow=5) plus parameters such as metabolome='HMDB(All)', matrix='FMP-10', typ='All', adducts='All', mass_error=5, and ccs=-1. The software uses structural information for the derivatizing matrix to generate a subset of target molecules containing only the functional groups specific to that agent, so multi-derivatized species (e.g. FMP-10 double tags via demethylation or dehydrogenation) are enumerated automatically. Independently compute expected m/z values: retrieve the metabolite with search_by_name('dopamine'), compute its neutral monoisotopic mass with RDKit, and add one or two FMP-10 increments (with demethylation/dehydrogenation variants) plus a proton for [M+H]+ ions; report deviations as ppm (dPPM) against reported values (421.1911, 674.2802, 688.2959) and require agreement within the 5 ppm mass_error used by the identification workflow. Optionally predict collision cross sections with predict_ccs() from a SMILES string, and recalibrate data with known m/z ratios to boost confidence in mass match results.

## Related tools

- **Met-ID** (Primary identification engine: MetIDServer context manager, identify() mass matching, predict_ccs(), and search_by_name() lookups against bundled HMDB/LIPID MAPS SQLite databases) — https://github.com/pbjarterot/Met-ID/releases
- **RDKit** (Computes neutral monoisotopic masses of retrieved metabolite structures to derive theoretical [M+H]+ m/z of single- and double-FMP-10-derivatized species for cross-checking identify() results)

## Examples

```
from metid import MetIDServer; with MetIDServer(): metid.identify(mz=421.1911, metabolome='HMDB(All)', matrix='FMP-10', typ='All', adducts='All', mass_error=5, mzwindow=5, ccs=-1)
```

## Evaluation signals

- Computed theoretical m/z for single-derivatized dopamine (1A, 421.1911) and double-derivatized forms (2A, 674.2802 demethylated; 2B, 688.2959 dehydrogenated) agree with reported values within the 5 ppm mass_error used by Met-ID
- identify() called with mz=421.1911, mzwindow=5, mass_error=5, metabolome='HMDB(All)', matrix='FMP-10' returns dopamine-derived candidates
- PPM deviations (dPPM) between computed and reported values fall inside the ±5 ppm tolerance
- MetIDServer context manager runs without the bundled-database staleness issue (the 'com.farmbio.metid' app-data folder is cleared so the installer re-adds current database files)
- Comparison table (e.g. derivatized_dopamine_mz_check.csv) contains computed, reported, and dPPM columns for all three species 1A, 2A, 2B

## Limitations

- Met-ID ships a preconfigured finite number of MS2 spectra; automated MS2 collection for MSI is not yet available
- Database files bundled with the installer may not update on reinstall — the 'com.farmbio.metid' folder must be removed manually before reinstalling
- rdkit installation is described as nontrivial (often via anaconda) and no pinned dependency versions are specified
- FMP-10 was developed in-house and its distribution/availability to other users is not stated; MacOS has issues with adding functional groups

## Evidence

- [methods] identify accepts one or more m/z values and optional corresponding CCS values: "``identify`` accepts one or more m/z values and optional corresponding CCS values."
- [methods] defaults select HMDB(All) and FMP-10 matrix: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [abstract] structural subset by derivatization-agent functional groups: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [abstract] identify multiple derivatization sites, exemplified by FMP-10: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [discussion] distinguish dopamine derivatives retaining or losing methyl via MS2: "Met-ID can distinguish between dopamine derivatives that retain or lose a methyl group based on their MS2 fragmentation patterns"
- [methods] Met-IDServer used as context manager: "Import ``MetIDServer`` and use it as a context manager."
- [readme] README: database files must be removed manually on reinstall: [evidence span withheld — anchor status: external_doc]
