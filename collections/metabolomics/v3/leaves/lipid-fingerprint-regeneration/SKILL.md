---
name: lipid-fingerprint-regeneration
description: Use when MS2 library search (EQ) plus LCI filtering leaves low-confidence
  or unannotated LC-MS/MS lipid features that need recall boosting via regenerated
  fingerprint spectra from a pre-trained deep-learning model.
when_to_use_negative:
- Features already annotated with high confidence by the EQ MS2 library search and
  LCI strict-criteria filtering — regeneration adds no value and risks false positives
- Non-lipid small-molecule metabolite identification where no lipid structural features
  (head group, chain length, unsaturation) map to retention/fingerprint models
- When a sample-specific experimental spectral library is already available — direct
  library search is preferable to model-regenerated fingerprints
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidIN
  role: Framework hosting the WMYn reverse-lipidomics fingerprint regeneration module
    and the theoretical spectral libraries
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Peak picking/feature-table generation feeding candidate MS/MS features into
    the annotation and regeneration workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:c58372d5ced681b0a60bdeafe54d62d8d1dd474eb4d534ecb784ee3beb9109dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - LipidIN EQ module
  - LipidIN LCI module
  - LipidIN WMYn module
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# lipid-fingerprint-regeneration

## Summary

Regenerates lipid fragment fingerprint spectra using LipidIN's Wide-spectrum Modeling Yield network (WMYn) as a 'reverse lipidomics' 5th-level library, boosting annotation recall (~20% estimated) and confirming low-confidence lipids independent of sample matrix, instrument, and analytical method.

## When to use

Use when MS2 library search (EQ) plus LCI filtering leaves low-confidence or unannotated LC-MS/MS lipid features that need recall boosting via regenerated fingerprint spectra from a pre-trained deep-learning model.

## When NOT to use

- Features already annotated with high confidence by the EQ MS2 library search and LCI strict-criteria filtering — regeneration adds no value and risks false positives
- Non-lipid small-molecule metabolite identification where no lipid structural features (head group, chain length, unsaturation) map to retention/fingerprint models
- When a sample-specific experimental spectral library is already available — direct library search is preferable to model-regenerated fingerprints

## Inputs

- Raw MS/MS feature tables from LC-MS/MS runs (Thermo .raw, SCIEX .wiff, Bruker/Agilent .d, .mzML)
- LipidIN EQ+LCI candidate annotation tables (low-confidence/unannotated features)
- LipidIN WMYn pre-trained fingerprint regeneration models from Zenodo (https://zenodo.org/records/21421866)
- LipidIN general theoretical spectral library (.rda: pos_ALL.rda/neg_ALL.rda; or .pkl: pos_common.pkl/neg_common.pkl, 24,631,702 spectra)

## Outputs

- Regenerated lipid fragment fingerprint spectra (5th-level library entries)
- Boosted/confirmed lipid annotation table with expanded coverage (~20% estimated recall increase)
- Annotation report (e.g. annotation_report.csv) with per-feature confidence and per-dataset FDR summary

## How to apply

After running the EQ (expeditious querying) MS2 library search against the LipidIN theoretical library (chain compositions + C=C double-bond positions, e.g. pos_ALL.rda/neg_ALL.rda or pos_common.pkl/neg_common.pkl) and applying LCI's three relative retention time rules with best-first search, invoke the WMYn module to regenerate accurate fingerprint spectrograms for candidate lipids. Use the regenerated fingerprints as the '5th-level library' to boost recall and confirm low-confidence annotations, following ECN-rule and feature-peak consistency criteria. Load pre-trained WMYn models (transferable, matrix-independent) from the Zenodo release (record 21421866) alongside the general library (24,631,702 spectra) and, if needed, the PB library (2,289,913 spectra) for double-bond localization. Verify regenerated fingerprints against experimental MS/MS feature tables before accepting boosted annotations.

## Related tools

- **LipidIN** (Framework hosting the WMYn reverse-lipidomics fingerprint regeneration module and the theoretical spectral libraries) — https://github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Peak picking/feature-table generation feeding candidate MS/MS features into the annotation and regeneration workflow)

## Examples

```
# Load WMYn-regenerated 5th-level library (from Zenodo record 21421866) and boost annotations
import LipidIN
report = LipidIN.annotate(feature_table='features.csv', library='pos_common.pkl', wmyn_5th_level=True, output='annotation_report.csv')
```

## Evaluation signals

- Estimated recall change should be approximately +20% after WMYn regeneration (as reported in the paper's figures)
- Regenerated fingerprint spectra must match experimental fragment m/z values of the candidate lipid (head group + fatty-acyl chain fragments) before confirming an annotation
- Regenerated fingerprints must be consistent with ECN-rule (equivalent carbon number) and feature-peak criteria enforced downstream
- Combined EQ+LCI+WMYn workflow on the RBL-2H3, mixture, human sera, and zebrafish datasets should reproduce ~5.7% average estimated FDR across 8,923 annotated lipids
- Annotation report should contain per-dataset FDR values and the 471 unique lipids in the mixture dataset consistent with Fig. 3e-h and Supplementary Data 6-7

## Limitations

- WMYn fingerprint regeneration accuracy depends on the pre-training corpus; the authors state that pre-training on a larger and more diverse dataset is a stated future need for extending reverse lipidomics
- FDR of ~5.7% is an estimated value, not a true error rate; requires target-decoy or annotated-ground-truth checks per dataset
- Minimum 16 GB RAM needed; regenerated libraries are model outputs and may not transfer perfectly to unusual sample matrices, instruments, or analytical methods

## Evidence

- [abstract] WMYn recall boost: "LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting."
- [discussion] regeneration as 5th-level library: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
- [intro] reverse lipidomics definition: "LipidIN incorporates a Wide-spectrum Modeling Yield network (WMYn) as “reverse lipidomics”, which regenerates highly accurate fingerprint spectrograms and easily transferable pre-trained models,"
- [methods] library download: "Download the spectral libraries from **[Zenodo → https://zenodo.org/records/21421866](https://zenodo.org/records/21421866)**"
- [discussion] future pre-training need: "Future research directions include pre-training on a larger and more diverse dataset to extend reverse lipidomics."
