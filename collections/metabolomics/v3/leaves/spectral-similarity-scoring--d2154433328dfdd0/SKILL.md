---
name: spectral-similarity-scoring
description: Use when a peak has been putatively annotated as an unknown metabolite
  by KGMN/MetDNA2 (e.g. in NIST human urine data) and you need independent support
  for the annotation via in silico MS/MS fragmentation rather than library matching.
when_to_use_negative:
- The metabolite is a known compound already matched against standard spectral libraries
  — direct MS1 m/z and MS2 library search suffices.
- You need to distinguish structurally similar isomers of unknown metabolites (e.g.
  isocitrate glucuronidation product isomers), which KGMN and in silico tools cannot
  effectively discriminate.
- Applying to non-model organism or environmental/exposomics samples where organism-specific
  reaction databases are not incorporated.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetFrag
  role: In silico fragmentation tool used to score candidate structures against measured
    MS/MS of each unknown peak
- name: CFM-ID
  role: Predicts in silico MS/MS spectra for candidate structures and computes spectral
    similarity against the unknown peak's MS/MS
- name: MS-FINDER
  role: Provides an independent in silico fragmentation corroboration score per unknown
    peak
- name: MetDNA2Vis
  role: R package used to load KGMN annotation results and convert raw peak MS/MS
    entries with convertSpectraData(ms2_data)
  repo: ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: ZhuMetLab R package installed alongside MetDNA2Vis for spectra handling
  repo: ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:16fa325da563acf1bc20459082c6ddb8011e02e3e22f9dfc02ba89af7d4b9a5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - CFM-ID
  - MS-FINDER
  - MetDNA2Vis
  - MetFrag
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-similarity-scoring

## Summary

Corroborates KGMN-putatively annotated unknown metabolite peaks by scoring candidate structures' predicted in silico MS/MS spectra (via MetFrag, CFM-ID, MS-FINDER) against measured peak MS/MS, since unknowns are absent from spectral databases and lack chemical standards.

## When to use

Use when a peak has been putatively annotated as an unknown metabolite by KGMN/MetDNA2 (e.g. in NIST human urine data) and you need independent support for the annotation via in silico MS/MS fragmentation rather than library matching.

## When NOT to use

- The metabolite is a known compound already matched against standard spectral libraries — direct MS1 m/z and MS2 library search suffices.
- You need to distinguish structurally similar isomers of unknown metabolites (e.g. isocitrate glucuronidation product isomers), which KGMN and in silico tools cannot effectively discriminate.
- Applying to non-model organism or environmental/exposomics samples where organism-specific reaction databases are not incorporated.

## Inputs

- MetDNA2/KGMN annotation results for a working directory (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/)
- Raw peak MS/MS entries converted via convertSpectraData(ms2_data)
- Putative candidate structures retrieved from HMDB and PubChem

## Outputs

- Per-peak tool-agreement table (MetFrag, CFM-ID, MS-FINDER corroboration per unknown peak)
- in_silico_corroboration_results.csv with per-peak and summary corroboration fractions

## How to apply

Load the MetDNA2/KGMN annotation results for the working directory (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/) with the MetDNA2Vis package and run convertSpectraData(ms2_data) to convert raw peak MS/MS entries into a usable spectral table. For each annotated unknown peak, retrieve putative candidate structures from HMDB and PubChem, then run MetFrag to score candidates against the measured MS/MS. Run CFM-ID to predict in silico MS/MS spectra for the candidates and compute spectral similarity against each unknown peak's MS/MS, and run MS-FINDER independently for a third corroboration score. Aggregate per peak, marking a peak corroborated when at least one of the three tools supports the KGMN annotation, and write per-peak and summary tables to in_silico_corroboration_results.csv.

## Related tools

- **MetFrag** (In silico fragmentation tool used to score candidate structures against measured MS/MS of each unknown peak)
- **CFM-ID** (Predicts in silico MS/MS spectra for candidate structures and computes spectral similarity against the unknown peak's MS/MS)
- **MS-FINDER** (Provides an independent in silico fragmentation corroboration score per unknown peak)
- **MetDNA2Vis** (R package used to load KGMN annotation results and convert raw peak MS/MS entries with convertSpectraData(ms2_data)) — ZhuMetLab/MetDNA2Vis
- **SpectraTools** (ZhuMetLab R package installed alongside MetDNA2Vis for spectra handling) — ZhuMetLab/SpectraTools

## Examples

```
library(MetDNA2Vis); ms2_table <- convertSpectraData(ms2_data)
```

## Evaluation signals

- Reproduced corroboration rate approaches 237/293 unknown peaks (80.9%) in positive ionization mode and 547/631 (86.7%) in negative mode.
- Computed corroboration fraction exceeds 80% in each ionization mode.
- in_silico_corroboration_results.csv exists, is valid CSV, and contains per-peak tool-agreement fields for MetFrag, CFM-ID, and MS-FINDER.
- convertSpectraData(ms2_data) yields a usable spectral table with MS/MS entries for each unknown peak.
- Every unknown peak marked corroborated has support from at least one of the three in silico MS/MS tools.

## Limitations

- Unknown metabolites are not included in spectral databases and have no chemical standards, so validation of unknown annotations remains a grand challenge.
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g. isocitrate glucuronidation product isomers).
- In silico biotransformation-derived candidate structures depend on BioTransformer reaction sets, which mainly originate from human metabolic reactions; caution is needed for non-model organisms and environmental/exposomics samples.

## Evidence

- [abstract] Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools.: "Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools."
- [results] 237 unknown peaks (80.9%) in positive ionization mode and 547 (86.7%) in negative ionization mode were corroborated by at least one in silico MS/MS tool.: [evidence span withheld — anchor status: not_found]
- [intro] bioinformatic tools, such as MetFrag15, CFM-ID16, MS-FINDER17, and SIRIUS18, have been developed to predict in silico MS/MS or molecular fingerprinting: "bioinformatic tools, such as MetFrag15, CFM-ID16, MS-FINDER17, and SIRIUS18, have been developed to predict in silico MS/MS"
- [readme] The goal of MetDNA2Vis package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN.: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [discussion] Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
