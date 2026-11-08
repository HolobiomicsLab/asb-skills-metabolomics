---
name: wmys-regenerated-fingerprint-recall-evaluation
description: Use when you have a lipidomics MS/MS dataset with a reference list of
  reported lipids and need to measure the recall gain contributed by WMYn-regenerated
  (5th-level) fingerprints versus a truncated 1st–4th-level library and the MS-DIAL
  public database.
when_to_use_negative:
- The input dataset has no MS/MS spectra (MS1-only features cannot be scored by the
  EQ-module MS2 library search).
- The goal is C=C double-bond localization — use the Paternò–Büchi (PB) derivatization
  workflow instead of this recall benchmark.
- The analytes are non-lipid metabolites outside LipidIN's 121-subclass lipid coverage;
  the 168.6-million-entry lipid fragmentation library does not apply.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: LipidIN
  role: Provides the EQ-module MS2 library search over the 1st–5th-level hierarchical
    library (pos_ALL.rda / neg_ALL.rda) and the LCI module's three relative retention
    time rules used to remove false positives before recall is computed.
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Supplies the public spectral database used as the baseline annotation configuration
    against which LipidIN's level-5 recall (93.64%) is compared.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:64a2f5584d4f47c06aa6ab025924d0609695b194bb19b8fad02741d8ad650443
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# wmys-regenerated-fingerprint-recall-evaluation

## Summary

Quantifies how much lipid recall is gained by adding the WMYn-regenerated 5th-level fingerprint library to LipidIN's hierarchical (1st–4th level) library, benchmarked against the MS-DIAL public database. The benchmark case is the mouse aging-associated lipidome atlas, where the 5-level library must reproduce 93.64% recall of 2704 reported lipids.

## When to use

Use when you have a lipidomics MS/MS dataset with a reference list of reported lipids and need to measure the recall gain contributed by WMYn-regenerated (5th-level) fingerprints versus a truncated 1st–4th-level library and the MS-DIAL public database.

## When NOT to use

- The input dataset has no MS/MS spectra (MS1-only features cannot be scored by the EQ-module MS2 library search).
- The goal is C=C double-bond localization — use the Paternò–Büchi (PB) derivatization workflow instead of this recall benchmark.
- The analytes are non-lipid metabolites outside LipidIN's 121-subclass lipid coverage; the 168.6-million-entry lipid fragmentation library does not apply.

## Inputs

- Published aging-associated lipidome atlas MS/MS dataset (mouse)
- LipidIN R-native hierarchical spectral library pos_ALL.rda / neg_ALL.rda (Zenodo record 13350719)
- MS-DIAL public database (baseline spectral library)
- Reference list of the 2704 reported lipids from the atlas

## Outputs

- recall_comparison.csv (overall and per-subclass recall for 1–4-level, 1–5-level, and MS-DIAL configurations)
- recall_fig9ab.png (Supplementary Fig. 9a–b-style per-subclass recall comparison plot)
- Filtered annotation tables after applying the LCI module's three relative retention time rules

## How to apply

Load the published MS/MS dataset and both libraries (MS-DIAL public database; the R-native pos_ALL.rda / neg_ALL.rda hierarchical library from Zenodo record 13350719) into the LipidIN R framework. Run the EQ-module MS2 library search twice over all acquired spectra: once with the hierarchical library truncated to levels 1–4 (chain compositions without WMYn-regenerated fingerprints) and once with the full 1st-to-5th-level library, where level 5 comprises the Wide-spectrum Modeling Yield network (WMYn) regenerated lipid fragment fingerprints. Run a baseline EQ search against the MS-DIAL public database to situate LipidIN performance, then apply the LCI module's three relative retention time rules to all search results to filter false-positive annotations. Compute overall and per-lipid-subclass recall as the fraction of reference lipids recovered for each library configuration, compare level-4 vs level-5 recall per subclass, and verify the overall level-5 recall reaches the reported ~93.64% with the expected ~20% WMYn-driven gain; export recall_comparison.csv and a Supplementary Fig. 9a–b-style per-subclass recall plot (recall_fig9ab.png).

## Related tools

- **LipidIN** (Provides the EQ-module MS2 library search over the 1st–5th-level hierarchical library (pos_ALL.rda / neg_ALL.rda) and the LCI module's three relative retention time rules used to remove false positives before recall is computed.) — https://github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Supplies the public spectral database used as the baseline annotation configuration against which LipidIN's level-5 recall (93.64%) is compared.)

## Examples

```
R> load("pos_ALL.rda"); load("neg_ALL.rda")  # R-native 1st–5th-level hierarchical libraries from Zenodo 13350719
```

## Evaluation signals

- The 1st-to-5th-level library configuration must reproduce 93.64% recall of the 2704 reported atlas lipids; deviations beyond the reported values indicate a failed reproduction.
- The level-5 recall must exceed the level-4 (chain-composition-only) configuration by the reported ~20% WMYn-driven recall gain.
- The per-subclass recall pattern must match Supplementary Fig. 9a–b, not just the overall recall number.
- The MS-DIAL public database baseline must be outperformed by the 5-level LipidIN configuration, as reported.
- recall_comparison.csv and recall_fig9ab.png must be exported with per-subclass recall fractions consistent with the overall aggregate.

## Limitations

- The ~20% WMYn recall gain is described as an estimated value, not an exact measured figure.
- False-positive removal depends on the LCI module's three relative retention time rules, whose 5.7% estimated FDR was calibrated across 8923 lipids; recall estimates are conditional on this filtering.
- The R implementation of LipidIN is maintained for reproducibility only and will no longer be updated; new work should use the Python LipidIN × Agent track.

## Evidence

- [abstract] LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting.: "LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting."
- [methods] the original hierarchical / MS-DIAL libraries are at https://zenodo.org/records/13350719: "the original hierarchical / MS-DIAL libraries are at https://zenodo.org/records/13350719"
- [readme] WMYn recall change | +20 % (estimated): "WMYn recall change | +20 % (estimated)"
- [discussion] regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
