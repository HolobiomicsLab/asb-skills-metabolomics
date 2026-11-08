---
name: metabolomics-annotation-table-preparation
description: Use when a KGMN/MetDNA2Vis unknown-annotation table is available and
  you must prepare MS/MS spectra of unknown metabolites for GNPS MASST searches to
  test recurrence across public metabolomics data.
when_to_use_negative:
- The metabolites are known compounds already present in common MS/MS libraries —
  repository mining is intended for unknowns absent from spectral databases
- You only need the three-layer KGMN networks (KMRN, MS2 similarity network, peak
  correlation network) themselves rather than spectra prepared for external repository
  search
- The input is raw metabolomics data not yet processed through KGMN/MetDNA2 annotation
  — no unknown annotation table exists to prepare
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA2Vis
  role: R package used to load and reproduce the KGMN annotation results and convert
    MS/MS spectra data (convertSpectraData) of unknown metabolites for downstream
    preparation
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: ZhuMetLab R package installed alongside MetDNA2Vis for handling the MS/MS
    spectra of annotated unknowns
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: GNPS
  role: MASST searches each prepared unknown MS/MS spectrum against all public data
    sets in MassIVE to identify recurrence across data sets, files, species, and sample
    types
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:6b4f8e8e17b3faa35ba4a3e199a95c9d7b611015452c2faca585ba1e4cfc10fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - GNPS
  - MetDNA2Vis
  - SpectraTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# metabolomics-annotation-table-preparation

## Summary

Extracts the putatively annotated unknown metabolite list and their MS/MS spectra from KGMN/MetDNA2Vis annotation results (e.g., Demo_MetDNA2_NIST_urine_pos) and reformats each spectrum (precursor m/z plus fragment peaks) for submission to GNPS MASST repository mining against MassIVE.

## When to use

Use when a KGMN/MetDNA2Vis unknown-annotation table is available and you must prepare MS/MS spectra of unknown metabolites for GNPS MASST searches to test recurrence across public metabolomics data.

## When NOT to use

- The metabolites are known compounds already present in common MS/MS libraries — repository mining is intended for unknowns absent from spectral databases
- You only need the three-layer KGMN networks (KMRN, MS2 similarity network, peak correlation network) themselves rather than spectra prepared for external repository search
- The input is raw metabolomics data not yet processed through KGMN/MetDNA2 annotation — no unknown annotation table exists to prepare

## Inputs

- KGMN/MetDNA2Vis annotation results directory for the unknown-annotation run (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/)
- Annotated unknown metabolite list with peak identifiers (e.g., M182T541, M262T526)
- MS/MS spectra (precursor m/z and fragment peaks) of the annotated unknown metabolites

## Outputs

- MASST-format MS/MS spectra for each unknown metabolite
- masst_recurrence_stats.csv with per-unknown data set count, data file count, species, and sample type

## How to apply

Load the annotation results from the KGMN run (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/) using the MetDNA2Vis R package together with SpectraTools, since the MetDNA2Vis package exists to 'visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN'. Use convertSpectraData() on the MS/MS data to place each unknown's precursor m/z and fragment peaks into the spectrum format required by GNPS MASST. Submit each prepared unknown spectrum to GNPS MASST so it is searched against all public data sets deposited in MassIVE. Aggregate matches per unknown by distinct data sets, data files, species, and sample types, and write the recurrence statistics to masst_recurrence_stats.csv; in the NIST urine case this yielded 187 recurrent unknowns in 351 data sets and 13,418 data files across 10 species and 12 sample types.

## Related tools

- **MetDNA2Vis** (R package used to load and reproduce the KGMN annotation results and convert MS/MS spectra data (convertSpectraData) of unknown metabolites for downstream preparation) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (ZhuMetLab R package installed alongside MetDNA2Vis for handling the MS/MS spectra of annotated unknowns) — https://github.com/ZhuMetLab/SpectraTools
- **GNPS** (MASST searches each prepared unknown MS/MS spectrum against all public data sets in MassIVE to identify recurrence across data sets, files, species, and sample types)

## Examples

```
library(MetDNA2Vis); library(SpectraTools); spectra_4_masst <- convertSpectraData(ms2_data)
```

## Evaluation signals

- masst_recurrence_stats.csv exists and contains fields for unknown identifier, data set count, data file count, species, and sample type
- Recurrence counts match the reported benchmark: 187 unknowns recurrent, 351 data sets, 13,418 data files, 10 species, 12 sample types
- Each prepared spectrum includes a precursor m/z and its fragment peaks in the format required by GNPS MASST
- The set of unknowns submitted corresponds exactly to the putatively annotated unknowns from the KGMN annotation results (e.g., peaks such as M182T541 and M262T526)

## Limitations

- Validation of unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards
- Only five recurrent unknown metabolites were validated with synthetic standards via repository-mining, so MASST recurrence alone does not confirm a structure
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites, which limits what repository mining of those spectra can resolve

## Evidence

- [abstract] we validate 5 metabolites that are absent in common MS/MS libraries through repository mining and synthesis of chemical standards.: "we validate 5 metabolites that are absent in common MS/MS libraries through repository mining and synthesis of chemical standards."
- [readme] The goal of MetDNA2Vis package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN.: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [methods] - `convertSpectraData(ms2_data)`: "- `convertSpectraData(ms2_data)`"
- [discussion] Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
