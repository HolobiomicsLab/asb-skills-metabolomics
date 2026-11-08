---
name: candidate-structure-fragmentation-scoring
description: Use when a KGMN/MetDNA2 annotation run has labeled unknown peaks in untargeted
  LC-MS/MS data (e.g., NIST urine pos/neg mode) that need reliability corroboration
  because no spectral-library match or chemical standard exists.
when_to_use_negative:
- The unknown annotations can be validated directly by spectral-library matching or
  by synthetic chemical standards (as done for the 5 validated metabolites in the
  paper) — in silico corroboration is a weaker surrogate
- The KGMN/MetDNA2 output working directory (e.g., 06_visualization/) has not been
  produced, so no annotated unknown peaks or ms2_data exist to convert
- The analysis concerns non-model organisms or exposomics profiles, where the authors
  caution the BioTransformer-derived reaction sets (mainly human metabolic reactions)
  limit unknown annotation
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MetFrag
  role: In silico fragmentation of candidate structures to score candidates against
    the measured MS/MS of each unknown peak
- name: CFM-ID
  role: Predicts in silico MS/MS spectra for candidate structures and computes spectral
    similarity against each unknown peak's MS/MS
- name: MS-FINDER
  role: Independent in silico fragmentation corroboration score per unknown peak from
    the same candidate structure lists
- name: MetDNA2Vis
  role: Loads KGMN annotation results and converts raw peak MS/MS entries (convertSpectraData)
    for the unknown peaks being corroborated
  repo: github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:cfd0608a153372b6331e0a43e937355b56fecd7b7f9752fe35b3c3c614deea80
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MS-FINDER
  - MetDNA2Vis
  - MetFrag
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# candidate-structure-fragmentation-scoring

## Summary

Corroborates KGMN-putatively annotated unknown metabolites (absent from spectral databases and without chemical standards) by scoring putative candidate structures from HMDB/PubChem against measured MS/MS using in silico fragmentation tools MetFrag, CFM-ID, and MS-FINDER, then aggregating per-peak tool agreement.

## When to use

Use when a KGMN/MetDNA2 annotation run has labeled unknown peaks in untargeted LC-MS/MS data (e.g., NIST urine pos/neg mode) that need reliability corroboration because no spectral-library match or chemical standard exists.

## When NOT to use

- The unknown annotations can be validated directly by spectral-library matching or by synthetic chemical standards (as done for the 5 validated metabolites in the paper) — in silico corroboration is a weaker surrogate
- The KGMN/MetDNA2 output working directory (e.g., 06_visualization/) has not been produced, so no annotated unknown peaks or ms2_data exist to convert
- The analysis concerns non-model organisms or exposomics profiles, where the authors caution the BioTransformer-derived reaction sets (mainly human metabolic reactions) limit unknown annotation

## Inputs

- MetDNA2/KGMN annotation results for the working directory (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/)
- Raw peak MS/MS entries (ms2_data) converted via convertSpectraData into a spectral table
- Putative candidate structures retrieved from HMDB and PubChem for each annotated unknown peak

## Outputs

- Per-peak tool-agreement table (MetFrag, CFM-ID, MS-FINDER corroboration per unknown peak)
- Summary corroboration table: counts/fractions of corroborated unknowns per ionization mode
- in_silico_corroboration_results.csv (CSV format with per-peak tool-agreement fields)

## How to apply

Load the MetDNA2/KGMN annotation results for the working directory (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/) via the MetDNA2Vis package and convert raw peak MS/MS entries into a spectral table with convertSpectraData(ms2_data). For each annotated unknown peak, retrieve putative candidate structures from HMDB and PubChem and run MetFrag in silico fragmentation to score candidates against the measured MS/MS spectrum. Run CFM-ID to predict in silico MS/MS spectra for the same candidates and compute spectral similarity to the unknown peak's MS/MS, and run MS-FINDER independently to obtain its fragmentation corroboration score. Mark a peak as corroborated when at least one of MetFrag, CFM-ID, or MS-FINDER supports the KGMN annotation, and write per-peak and summary agreement tables to in_silico_corroboration_results.csv. The rationale is that >80% of unknowns should be corroborated this way, providing reproducible evidence for annotations that cannot be validated with standards.

## Related tools

- **MetFrag** (In silico fragmentation of candidate structures to score candidates against the measured MS/MS of each unknown peak)
- **CFM-ID** (Predicts in silico MS/MS spectra for candidate structures and computes spectral similarity against each unknown peak's MS/MS)
- **MS-FINDER** (Independent in silico fragmentation corroboration score per unknown peak from the same candidate structure lists)
- **MetDNA2Vis** (Loads KGMN annotation results and converts raw peak MS/MS entries (convertSpectraData) for the unknown peaks being corroborated) — github.com/ZhuMetLab/MetDNA2Vis

## Examples

```
library(MetDNA2Vis); setwd('Demo_MetDNA2_NIST_urine_pos/06_visualization/'); reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1()
```

## Evaluation signals

- in_silico_corroboration_results.csv exists, is valid CSV, and contains per-peak tool-agreement fields for MetFrag, CFM-ID, and MS-FINDER
- Computed corroboration fraction exceeds 80% in each ionization mode, matching the reported >80% unknown metabolites corroborated with in silico MS/MS tools
- Positive-mode reproduction approaches 237/293 unknown peaks (80.9%) corroborated by at least one tool
- Negative-mode reproduction approaches 547/631 unknown peaks (86.7%) corroborated by at least one tool
- Per-peak rows cover all annotated unknown peaks from the MetDNA2Vis-converted spectral table (293 pos-mode and 631 neg-mode unknowns in the NIST urine demo)

## Limitations

- Unknowns are absent from spectral databases and have no chemical standards, so in silico corroboration is only putative support, not definitive validation
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), so corroborated annotations may still be isomerically ambiguous
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; BioTransformer reaction sets mainly originate from human metabolic reactions, limiting generalization to non-model organisms and environmental/exposomics samples
- Only five recurrent unknown metabolites were validated with synthetic standards via repository mining; most corroborations rely solely on tool agreement

## Evidence

- [abstract] >80% corroboration with in silico tools: "Among them, >80% unknown metabolites are corroborated with in silico MS/MS tools."
- [intro] in silico MS/MS prediction tools: "bioinformatic tools, such as MetFrag15, CFM-ID16, MS-FINDER17, and SIRIUS18, have been developed to predict in silico MS/MS or molecular fingerprinting"
- [discussion] unknown annotation validation challenge: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
- [readme] MetDNA2Vis purpose: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [methods] network reconstruction for unknown annotation: "reconstructNetwork1(is_unknown_annotation = TRUE)"
