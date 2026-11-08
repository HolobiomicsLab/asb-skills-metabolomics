---
name: in-silico-msms-spectral-prediction
description: Use when validating putative unknown metabolite annotations from LC-MS
  untargeted metabolomics (e.g., KGMN/MetDNA2 unknown peaks in NIST urine data) where
  no spectral library match or chemical standard exists and computational fragmentation
  support is needed.
when_to_use_negative:
- The metabolites are known/knowns already matched against standard spectral libraries
  by MS1 m/z and MS2 spectrum search, so no in silico corroboration is required
- Authentic chemical standards are available, in which case direct experimental MS/MS
  validation is preferred over in silico prediction
- The goal is discriminating structurally similar isomers of unknowns (e.g., isocitrate
  glucuronidation product isomers), which KGMN and this corroboration workflow cannot
  effectively resolve
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3520
tools:
- name: MetFrag
  role: In silico fragmentation tool that scores candidate structures against measured
    MS/MS for each unknown peak
- name: CFM-ID
  role: Predicts in silico MS/MS spectra for candidate structures and computes spectral
    similarity against each unknown peak's MS/MS
- name: MS-FINDER
  role: Provides an independent in silico fragmentation corroboration score per unknown
    peak on the same candidate structure lists
- name: MetDNA2Vis
  role: R package used to load KGMN annotation results and convert raw peak MS/MS
    entries into a usable spectral table via convertSpectraData
  repo: github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:8d7da7d5126eeac5b1b9dff4975fb45be61792ad22897a7da5bcfb91d7a10b27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MS-FINDER
  - MetDNA2Vis
  - MetFrag
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# in-silico-msms-spectral-prediction

## Summary

Corroborates KGMN-putatively annotated unknown metabolites (which are absent from spectral databases and lack chemical standards) by running candidate structures through in silico MS/MS prediction tools — MetFrag, CFM-ID, and MS-FINDER — and scoring candidate-vs-measured spectra agreement per unknown peak.

## When to use

Use when validating putative unknown metabolite annotations from LC-MS untargeted metabolomics (e.g., KGMN/MetDNA2 unknown peaks in NIST urine data) where no spectral library match or chemical standard exists and computational fragmentation support is needed.

## When NOT to use

- The metabolites are known/knowns already matched against standard spectral libraries by MS1 m/z and MS2 spectrum search, so no in silico corroboration is required
- Authentic chemical standards are available, in which case direct experimental MS/MS validation is preferred over in silico prediction
- The goal is discriminating structurally similar isomers of unknowns (e.g., isocitrate glucuronidation product isomers), which KGMN and this corroboration workflow cannot effectively resolve

## Inputs

- MetDNA2/KGMN annotation results for a working directory (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/)
- Raw peak MS/MS spectral data converted with convertSpectraData(ms2_data)
- Putative candidate structures retrieved from HMDB and PubChem for each annotated unknown peak

## Outputs

- in_silico_corroboration_results.csv with per-peak MetFrag, CFM-ID, and MS-FINDER agreement fields and a per-peak corroborated flag
- Summary corroboration counts and fractions per ionization mode (e.g., 237/293 = 80.9% positive; 547/631 = 86.7% negative)

## How to apply

Load the MetDNA2/KGMN annotation results for the working directory (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/) with the MetDNA2Vis package and convert raw peak MS/MS entries into a usable spectral table via convertSpectraData(ms2_data). For each annotated unknown peak, retrieve putative candidate structures from HMDB and PubChem, run MetFrag in silico fragmentation to score candidates against the measured MS/MS, run CFM-ID to predict spectra and compute spectral similarity, and run MS-FINDER on the same candidate lists for independent fragmentation corroboration scores. Aggregate per unknown peak, marking a peak corroborated when at least one of MetFrag, CFM-ID, or MS-FINDER supports the KGMN annotation, and write per-peak plus summary results to in_silico_corroboration_results.csv. Judge success by reproduction of the reported corroboration fractions: 237/293 unknown peaks (80.9%) in positive mode and 547/631 (86.7%) in negative mode — computed corroboration should exceed 80% per mode.

## Related tools

- **MetFrag** (In silico fragmentation tool that scores candidate structures against measured MS/MS for each unknown peak)
- **CFM-ID** (Predicts in silico MS/MS spectra for candidate structures and computes spectral similarity against each unknown peak's MS/MS)
- **MS-FINDER** (Provides an independent in silico fragmentation corroboration score per unknown peak on the same candidate structure lists)
- **MetDNA2Vis** (R package used to load KGMN annotation results and convert raw peak MS/MS entries into a usable spectral table via convertSpectraData) — github.com/ZhuMetLab/MetDNA2Vis

## Examples

```
library(MetDNA2Vis); setwd('Demo_MetDNA2_NIST_urine_pos/06_visualization/'); ms2_spectra_table <- convertSpectraData(ms2_data)
```

## Evaluation signals

- in_silico_corroboration_results.csv exists in CSV format and contains per-peak tool-agreement fields for MetFrag, CFM-ID, and MS-FINDER
- Computed corroboration fraction exceeds 80% in each ionization mode (positive and negative)
- Reproduced counts approach the reported values: 237/293 unknown peaks (80.9%) in positive mode and 547/631 (86.7%) in negative mode
- A peak is corroborated only under the defined 'at least one of MetFrag, CFM-ID, or MS-FINDER supports' criterion, applied per unknown peak
- All annotated unknown peaks in the KGMN result set are represented (none dropped) before aggregation

## Limitations

- KGMN and this corroboration cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; BioTransformer reaction sets mainly originate from human metabolic reactions
- Caution needed when applying to non-model organisms and environmental/exposomics samples, since organism-specific databases are not incorporated; validating unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards

## Evidence

- [abstract] Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools.: "Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools."
- [intro] bioinformatic tools, such as MetFrag15, CFM-ID16, MS-FINDER17, and SIRIUS18, have been developed to predict in silico MS/MS or molecular fingerprinting: "bioinformatic tools, such as MetFrag15, CFM-ID16, MS-FINDER17, and SIRIUS18, have been developed to predict in silico MS/MS or molecular fingerprinting"
- [methods] convertSpectraData(ms2_data): "`convertSpectraData(ms2_data)`"
- [readme] The goal of MetDNA2Vis package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN.: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
