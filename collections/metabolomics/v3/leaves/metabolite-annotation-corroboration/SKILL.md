---
name: metabolite-annotation-corroboration
description: Use when a KGMN or MetDNA2 annotation run produced putatively annotated
  unknown peaks (e.g. from NIST urine) that cannot be validated against spectral libraries
  or standards, and independent in silico MS/MS support is needed.
when_to_use_negative:
- The metabolites of interest are knowns already present in standard spectral libraries
  — match MS1 m/z and MS2 spectra directly against libraries instead
- Structurally similar isomers must be distinguished (KGMN cannot effectively discriminate
  isomers such as isocitrate glucuronidation product isomers)
- Input is a feature table with no MS/MS spectra and no candidate structures available
  from HMDB or PubChem
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MetFrag
  role: In silico fragmentation to score candidate structures from HMDB/PubChem against
    each unknown peak's measured MS/MS
- name: CFM-ID
  role: Predicts in silico MS/MS spectra for candidate structures and computes spectral
    similarity against each unknown peak's MS/MS
- name: MS-FINDER
  role: Provides an independent in silico fragmentation corroboration score per unknown
    peak
- name: MetDNA2Vis
  role: R package to load, visualize and reproduce KGMN annotations, including convertSpectraData
    and network reconstruction/retrieval functions
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab R package installed alongside MetDNA2Vis for spectral
    handling
  repo: github.com/ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:7b3e07b7b9247fc73bf1b5e900e1759ca96d040d57efd6db1397e4f895a861d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MS-FINDER
  - MetDNA2Vis
  - MetFrag
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolite-annotation-corroboration

## Summary

Corroborates KGMN/MetDNA2-putatively annotated unknown metabolites (absent from spectral databases and lacking chemical standards) using three independent in silico MS/MS fragmentation tools — MetFrag, CFM-ID, and MS-FINDER — on NIST urine LC-MS/MS data.

## When to use

Use when a KGMN or MetDNA2 annotation run produced putatively annotated unknown peaks (e.g. from NIST urine) that cannot be validated against spectral libraries or standards, and independent in silico MS/MS support is needed.

## When NOT to use

- The metabolites of interest are knowns already present in standard spectral libraries — match MS1 m/z and MS2 spectra directly against libraries instead
- Structurally similar isomers must be distinguished (KGMN cannot effectively discriminate isomers such as isocitrate glucuronidation product isomers)
- Input is a feature table with no MS/MS spectra and no candidate structures available from HMDB or PubChem

## Inputs

- MetDNA2/KGMN annotation results for a working directory (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/)
- Raw peak MS/MS entries convertible via convertSpectraData(ms2_data)
- Candidate structures retrieved from HMDB and PubChem
- Annotated unknown peak lists per ionization mode (positive: 293 peaks; negative: 631 peaks)

## Outputs

- in_silico_corroboration_results.csv with per-peak MetFrag/CFM-ID/MS-FINDER agreement fields
- Summary corroboration rates per ionization mode (target: >80% per mode)

## How to apply

Load the MetDNA2/KGMN annotation results for the working directory (e.g. Demo_MetDNA2_NIST_urine_pos) using the MetDNA2Vis package and convert raw peak MS/MS entries with convertSpectraData(ms2_data) into a usable spectral table. For each annotated unknown peak, retrieve putative candidate structures from HMDB and PubChem, then run MetFrag in silico fragmentation to score candidates against the measured MS/MS. Independently run CFM-ID to predict spectra and compute spectral similarity, and MS-FINDER to obtain an independent fragmentation corroboration score for the same candidate lists. Aggregate per peak, marking a peak corroborated when at least one of the three tools supports the annotation, and write per-peak plus summary results to in_silico_corroboration_results.csv. The rationale is that since unknowns are absent from spectral databases and have no chemical standards, tool consensus across complementary fragmentation algorithms is the practical validation proxy.

## Related tools

- **MetFrag** (In silico fragmentation to score candidate structures from HMDB/PubChem against each unknown peak's measured MS/MS)
- **CFM-ID** (Predicts in silico MS/MS spectra for candidate structures and computes spectral similarity against each unknown peak's MS/MS)
- **MS-FINDER** (Provides an independent in silico fragmentation corroboration score per unknown peak)
- **MetDNA2Vis** (R package to load, visualize and reproduce KGMN annotations, including convertSpectraData and network reconstruction/retrieval functions) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab R package installed alongside MetDNA2Vis for spectral handling) — github.com/ZhuMetLab/SpectraTools

## Examples

```
library(MetDNA2Vis); reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1(); convertSpectraData(ms2_data)
```

## Evaluation signals

- in_silico_corroboration_results.csv exists and contains per-peak tool-agreement fields for MetFrag, CFM-ID, and MS-FINDER
- Computed corroboration fraction exceeds 80% in each ionization mode
- Positive mode approaches the reported 237/293 unknown peaks (80.9%) corroborated
- Negative mode approaches the reported 547/631 unknown peaks (86.7%) corroborated
- Per-peak corroboration is defined consistently: a peak counts only when at least one of the three tools supports the KGMN annotation

## Limitations

- Unknowns are absent from spectral databases and have no chemical standards, so validation of unknown annotations remains a grand challenge and relies on in silico proxy support
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g. isocitrate glucuronidation product isomers)
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- Applying KGMN-derived annotations to non-model organisms and environmental/exposomics samples needs caution; organism-specific databases are not yet incorporated

## Evidence

- [abstract] Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools.: "Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools."
- [abstract] we apply KGMN in an in vitro enzymatic reaction system and different biological samples, with ~100–300 putative unknowns annotated in each data set.: "we apply KGMN in an in vitro enzymatic reaction system and different biological samples, with ~100–300 putative unknowns annotated in each data set."
- [methods] Demo_MetDNA2_NIST_urine_pos/06_visualization/: "Demo_MetDNA2_NIST_urine_pos/06_visualization/'"
- [methods] devtools::install_github("ZhuMetLab/MetDNA2Vis"): "devtools::install_github("ZhuMetLab/MetDNA2Vis")"
- [discussion] Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
- [readme] The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN.: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
