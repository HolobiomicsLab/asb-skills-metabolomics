---
name: annotation-consistency-statistics
description: Use when KGMN/MetDNA2-annotated unknown peaks (e.g., from NIST urine
  pos/neg mode) lack standards and spectral library entries, and you need in silico
  MS/MS tool agreement statistics to corroborate annotations.
when_to_use_negative:
- The metabolites are known compounds with entries in standard MS/ spectral libraries
  or available chemical standards — direct library search or standard-based validation
  is appropriate instead.
- Input is raw LC-MS data not yet processed through the KGMN/MetDNA2 annotation workflow
  — no annotation table or MS/MS peak entries exist to corroborate.
- Applying KGMN/this corroboration to non-model organisms or environmental/exposomics
  samples where organism-specific reaction databases have not been incorporated —
  the article advises caution in these cases.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA2Vis
  role: Loads KGMN annotation results and converts raw peak MS/MS entries (convertSpectraData)
    into a usable spectral table for corroboration analysis
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: MetFrag
  role: In silico fragmentation tool scoring candidate structures against measured
    MS/MS of each unknown peak
- name: CFM-ID
  role: Predicts in silico MS/MS spectra for candidate structures and computes spectral
    similarity against each unknown peak's MS/MS
- name: MS-FINDER
  role: Provides independent in silico fragmentation corroboration scores for the
    same candidate structure lists per unknown peak
- name: SpectraTools
  role: Companion ZhuMetLab R package installed alongside MetDNA2Vis for spectral
    data handling during reproduction
  repo: https://github.com/ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:a11c17a785e2aad6f45cbf81ccf3138b94d24c2ea8aa4427688431b9332297b4
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# annotation-consistency-statistics

## Summary

Corroborates KGMN-putatively annotated unknown metabolites by scoring candidate structures with in silico MS/MS fragmentation tools (MetFrag, CFM-ID, MS-FINDER) and computing per-peak agreement rates. It addresses the challenge that unknowns absent from spectral databases and without chemical standards cannot be validated conventionally.

## When to use

Use when KGMN/MetDNA2-annotated unknown peaks (e.g., from NIST urine pos/neg mode) lack standards and spectral library entries, and you need in silico MS/MS tool agreement statistics to corroborate annotations.

## When NOT to use

- The metabolites are known compounds with entries in standard MS/ spectral libraries or available chemical standards — direct library search or standard-based validation is appropriate instead.
- Input is raw LC-MS data not yet processed through the KGMN/MetDNA2 annotation workflow — no annotation table or MS/MS peak entries exist to corroborate.
- Applying KGMN/this corroboration to non-model organisms or environmental/exposomics samples where organism-specific reaction databases have not been incorporated — the article advises caution in these cases.

## Inputs

- MetDNA2/KGMN annotation results for a working directory (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/)
- Raw peak MS/MS entries convertible via convertSpectraData(ms2_data)
- Putative candidate structures retrieved from HMDB and PubChem

## Outputs

- in_silico_corroboration_results.csv with per-peak MetFrag/CFM-ID/MS-FINDER agreement fields
- Summary corroboration statistics per ionization mode (e.g., 237/293 = 80.9% pos; 547/631 = 86.7% neg)

## How to apply

Load MetDNA2/KGMN annotation results for the working directory (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/) with the MetDNA2Vis package and convert raw peak MS/MS entries with convertSpectraData(ms2_data) into a usable spectral table. For each annotated unknown peak, retrieve putative candidate structures from HMDB and PubChem, then run MetFrag in silico fragmentation to score candidates against the measured MS/MS, run CFM-ID to predict spectra and compute spectral similarity, and run MS-FINDER independently on the same candidate lists. Aggregate per peak, marking a peak as corroborated when at least one of the three tools supports the KGMN annotation, and write per-peak and summary results to in_silico_corroboration_results.csv. Judge success by comparing corroboration fractions to the reported 237/293 (80.9%) in positive mode and 547/631 (86.7%) in negative mode; the computed fraction should exceed 80% in each mode.

## Related tools

- **MetDNA2Vis** (Loads KGMN annotation results and converts raw peak MS/MS entries (convertSpectraData) into a usable spectral table for corroboration analysis) — https://github.com/ZhuMetLab/MetDNA2Vis
- **MetFrag** (In silico fragmentation tool scoring candidate structures against measured MS/MS of each unknown peak)
- **CFM-ID** (Predicts in silico MS/MS spectra for candidate structures and computes spectral similarity against each unknown peak's MS/MS)
- **MS-FINDER** (Provides independent in silico fragmentation corroboration scores for the same candidate structure lists per unknown peak)
- **SpectraTools** (Companion ZhuMetLab R package installed alongside MetDNA2Vis for spectral data handling during reproduction) — https://github.com/ZhuMetLab/SpectraTools

## Examples

```
library(MetDNA2Vis); setwd('Demo_MetDNA2_NIST_urine_pos/06_visualization/'); annotation_table <- reformatTable1(); ms2_table <- convertSpectraData(ms2_data)
```

## Evaluation signals

- in_silico_corroboration_results.csv exists, is valid CSV, and contains per-peak tool-agreement fields for MetFrag, CFM-ID, and MS-FINDER
- Computed corroboration fraction exceeds 80% in each ionization mode
- Positive-mode results approach the reported 237/293 unknown peaks (80.9%) corroborated
- Negative-mode results approach the reported 547/631 unknown peaks (86.7%) corroborated
- Every annotated unknown peak has a corroboration verdict derived from at least one of the three in silico MS/MS tools

## Limitations

- Since unknown metabolites are not included in spectral databases and have no chemical standards, validation remains indirect — in silico tool agreement is corroboration, not definitive confirmation
- Only five recurrent unknown metabolites were ultimately validated with synthetic chemical standards via repository mining
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), which limits annotation accuracy regardless of tool agreement
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; BioTransformer reaction sets mainly originate from human metabolic reactions

## Evidence

- [abstract] >80% unknown metabolites corroborated with in silico MS/MS tools: "Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools."
- [intro] bioinformatic tools predict in silico MS/MS for candidate structures: "bioinformatic tools, such as MetFrag15, CFM-ID16, MS-FINDER17, and SIRIUS18, have been developed to predict in silico MS/MS or molecular fingerprinting"
- [discussion] validation of unknown annotations is a grand challenge: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
- [readme] MetDNA2Vis reproduces KGMN annotations: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [methods] convertSpectraData converts raw peak MS/MS entries: "- `convertSpectraData(ms2_data)`"
