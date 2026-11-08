---
name: derivatized-metabolite-annotation-maldi-msi
description: Use when MALDI-MSI m/z features must be assigned to metabolites that
  were derivatized with a reactive matrix (e.g., FMP-10) producing ions other than
  [M+H]+/[M-H]-, or when multiple derivatization sites on one molecule must be annotated.
when_to_use_negative:
- Input is underivatized conventional-matrix data where standard [M+H]+/[M-H]- annotation
  tools suffice (Met-ID still works, but derivatization-aware filtering adds no value)
- The target derivatizing matrix has no defined reactive functional groups or mass
  shift, since target subsetting and derivative m/z computation depend on these
- The query spectrum is not from MALDI-MSI or a compatible mass-spectrometry metabolomics
  experiment
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: Met-ID
  role: 'Primary software: matches m/z features to derivatization-aware target subsets
    from HMDB/LIPID MAPS SQLite databases, enumerates multiple derivatization sites,
    recalibrates masses, and compares MS2 spectra'
  repo: https://github.com/pbjarterot/Met-ID
- name: RDKit
  role: Computational-chemistry dependency used by Met-ID for structural/functional-group
    matching of derivatization sites
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:a387c8cd6a9dba66c893dcd98b107943889e48bf85051a46df007c4740df7998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  - RDKit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# derivatized-metabolite-annotation-maldi-msi

## Summary

Annotate chemically derivatized metabolites in MALDI-MSI data using Met-ID, which matches m/z features against HMDB/LIPID MAPS-derived targets filtered by derivatization-reactive functional groups and supports multiple derivatization sites per molecule (e.g., FMP-10 matrix).

## When to use

Use when MALDI-MSI m/z features must be assigned to metabolites that were derivatized with a reactive matrix (e.g., FMP-10) producing ions other than [M+H]+/[M-H]-, or when multiple derivatization sites on one molecule must be annotated.

## When NOT to use

- Input is underivatized conventional-matrix data where standard [M+H]+/[M-H]- annotation tools suffice (Met-ID still works, but derivatization-aware filtering adds no value)
- The target derivatizing matrix has no defined reactive functional groups or mass shift, since target subsetting and derivative m/z computation depend on these
- The query spectrum is not from MALDI-MSI or a compatible mass-spectrometry metabolomics experiment

## Inputs

- MALDI-MSI peak list of m/z values (optionally with CCS values)
- Selected derivatizing matrix (e.g., FMP-10) or custom derivatization reagent with its mass shift
- User-provided tissue MS2 spectra for similarity comparison
- Known m/z ratios for mass recalibration

## Outputs

- Annotated metabolite identifications with derivatization-site counts (possible_derivs) and derivative m/z values
- MS2 spectral similarity matches against the standards database
- Recalibrated m/z feature lists with boosted mass-match confidence

## How to apply

Start a local Met-ID server via the MetIDServer context manager, then use identify() to match one or more m/z values (with optional CCS values) against the SQLite database parsed from HMDB and LIPID MAPS. Met-ID uses structural information for the derivatizing matrix to generate a subset of targets containing only functional groups specific to the derivatization agent, so set the matrix (defaults select HMDB(All) and FMP-10) and let it enumerate multiple derivatization sites per molecule. Recalibrate data with known m/z ratios to boost confidence in mass-match results, and for confirmation compare user-provided tissue MS2 spectra against the built-in spectral database of standards (collected at 0–50 eV collision energies, 1 Da isolation window) for similarity assessment. For non-FMP-10 reagents like AMPP, count reactive amine sites per SMILES and apply one reagent mass shift per site to the protonated mass to compute derivative m/z values.

## Related tools

- **Met-ID** (Primary software: matches m/z features to derivatization-aware target subsets from HMDB/LIPID MAPS SQLite databases, enumerates multiple derivatization sites, recalibrates masses, and compares MS2 spectra) — https://github.com/pbjarterot/Met-ID
- **RDKit** (Computational-chemistry dependency used by Met-ID for structural/functional-group matching of derivatization sites)

## Examples

```
from metid import MetIDServer; with MetIDServer() as server: hits = server.identify(mz_values=[175.0957, 104.0586], matrix='FMP-10')
```

## Evaluation signals

- Each reported identification contains a derivatization-site count and derivative m/z consistent with the selected matrix's mass shift(s)
- Recalibration with known m/z ratios reduces mass-match residuals and increases match confidence
- MS2 similarity matches are supported by spectra acquired with 1 Da isolation at 0–50 eV collision energies comparable to the built-in standards database
- Verification workflow: all panel metabolites (e.g., dopamine, GABA, spermidine) present with at least one annotated derivatization site and finite derivative m/z, checked by reading the output CSV back with pandas

## Limitations

- Preconfigured finite number of MS2 spectra in Met-ID; no automated MS2 collection for MSI (noted as future work)
- FMP-10 was developed in-house and its distribution to other users is not stated; users must extend the software locally for other matrices
- Database changes (e.g., added functional groups or spectra) only affect the local install, not other users
- rdkit installation is nontrivial (recommended via anaconda); no version numbers pinned for Met-ID, rdkit, or Tauri dependencies
- Reinstalling newer Met-ID versions may not update bundled database files — the 'com.farmbio.metid' app-data folder must be removed manually before reinstalling; on MacOS there are known issues adding functional groups

## Evidence

- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [abstract] Met-ID also permits users to recalibrate data with known m/z ratios, boosting confidence in mass match results.: "Met-ID also permits users to recalibrate data with known m/z ratios, boosting confidence in mass match results."
- [methods] Import ``MetIDServer`` and use it as a context manager.: "Import ``MetIDServer`` and use it as a context manager."
- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [readme] Met-ID is extendable to use any derivatizing matrix with the tools to do local version changes right from inside the software.: [evidence span withheld — anchor status: external_doc]
- [discussion] The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV): "The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV)"
