---
name: lipid-subclass-recall-benchmarking
description: Use when a published lipidomics dataset reports a known set of lipids
  (e.g. 2704 lipids in the mouse aging-associated lipidome atlas) and you must quantify
  how many are recovered by a spectral-library search.
when_to_use_negative:
- The goal is discovery of novel/unannotated lipids rather than quantitative benchmarking
  against a known reference lipid list.
- The dataset lacks MS/MS spectra or retention-time information, so neither the EQ
  MS2 library search nor the LCI relative-RT filtering can be applied.
- The task is Paternò–Büchi double-bond localization, which uses LipidIN's separate
  PB library module rather than the general hierarchical library recall workflow.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Provides the EQ-module MS2 library search over the R-native hierarchical library
    (pos_ALL.rda/neg_ALL.rda), the WMYn 5th-level regenerated fingerprints, and the
    LCI module's three relative retention time rules used to filter false positives
    before recall scoring.
  repo: github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Supplies the public spectral database used as the baseline annotation configuration
    against which LipidIN's hierarchical-library recall is compared.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:6d0558096c362f7b1d4b6ad7337003a799546ff4877fad6e3f114cd4b0b27442
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lipid-subclass-recall-benchmarking

## Summary

Benchmarks lipid annotation recall of a spectral library against a reference list of reported lipids from a published dataset, comparing library variants (e.g. hierarchical level 1–4 vs 1–5) and a public-database baseline at overall and per-subclass granularity.

## When to use

Use when a published lipidomics dataset reports a known set of lipids (e.g. 2704 lipids in the mouse aging-associated lipidome atlas) and you must quantify how many are recovered by a spectral-library search.

## When NOT to use

- The goal is discovery of novel/unannotated lipids rather than quantitative benchmarking against a known reference lipid list.
- The dataset lacks MS/MS spectra or retention-time information, so neither the EQ MS2 library search nor the LCI relative-RT filtering can be applied.
- The task is Paternò–Büchi double-bond localization, which uses LipidIN's separate PB library module rather than the general hierarchical library recall workflow.

## Inputs

- Published aging-associated lipidome atlas in mice MS/MS dataset (LC-MS/MS spectra with retention times)
- Reference list of the 2704 reported lipids with lipid subclass labels
- LipidIN R-native hierarchical spectral library (pos_ALL.rda, neg_ALL.rda; Zenodo record 13350719)
- MS-DIAL public database (baseline comparator library)

## Outputs

- recall_comparison.csv (overall and per-lipid-subclass recall for level-4, level-5, and MS-DIAL baseline configurations)
- Per-subclass recall comparison plot in Supplementary Fig. 9a–b style (recall_fig9ab.png)

## How to apply

Load the published atlas MS/MS dataset together with the candidate libraries — for LipidIN, the R-native pos_ALL.rda/neg_ALL.rda hierarchical library from Zenodo record 13350719 and the MS-DIAL public database — into the LipidIN for R framework. Run the EQ-module MS2 library search twice: once with the hierarchical library truncated to levels 1–4 (chain compositions without WMYn-regenerated fingerprints) and once with the full 1st-to-5th-level library, where level 5 comprises Wide-spectrum Modeling Yield (WMYn) regenerated fragment fingerprints; run a third baseline search against the MS-DIAL public database. Apply the LCI module's three relative retention time rules to filter false-positive annotations in every search result before scoring. Compute overall and per-subclass recall as the fraction of the 2704 reference lipids recovered, then compare level-4 vs level-5 per subclass to isolate the WMYn-driven gain (~20% estimated). The reproduction is validated only if the 5-level library reproduces the reported 93.64% overall recall and the per-subclass pattern of Supplementary Fig. 9a–b.

## Related tools

- **LipidIN** (Provides the EQ-module MS2 library search over the R-native hierarchical library (pos_ALL.rda/neg_ALL.rda), the WMYn 5th-level regenerated fingerprints, and the LCI module's three relative retention time rules used to filter false positives before recall scoring.) — github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Supplies the public spectral database used as the baseline annotation configuration against which LipidIN's hierarchical-library recall is compared.)

## Examples

```
load('pos_ALL.rda'); load('neg_ALL.rda')  # LipidIN R-native hierarchical libraries (Zenodo 13350719) for EQ-module MS2 recall benchmarking
```

## Evaluation signals

- Overall recall of the 1st-to-5th-level library must reproduce the reported 93.64% of the 2704 reference lipids; deviations beyond reported values indicate a failed reproduction.
- Level-5 recall must show the expected ~20% WMYn-driven recall gain over the level-4 (chain-composition-only) configuration.
- Per-subclass recall pattern must match Supplementary Fig. 9a–b.
- MS-DIAL public database baseline must underperform the 5-level hierarchical library on this atlas.
- All retained annotations must survive the LCI module's three relative retention time rules (5.7% estimated FDR regime).

## Limitations

- The LipidIN R implementation is kept for reproducibility only and will no longer be updated; new users are directed to the Python track, so R-based recall reproduction depends on archived code.
- Similarity-algorithm baselines such as MS-DIAL, LipidSearch, and MS entropy can cause feature redundancy, which affects fair comparison of recall figures.
- Recall depends on the archived library version (Zenodo 13350719); using other Zenodo records (e.g. 21421866 for Python .pkl libraries or 13735639 for oxidized-lipid spectra) changes the benchmark configuration.

## Evidence

- [abstract] LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting.: "LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting."
- [abstract] developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [methods] the original hierarchical / MS-DIAL libraries are at https://zenodo.org/records/13350719: "the original hierarchical / MS-DIAL libraries are at https://zenodo.org/records/13350719"
- [methods] R-native | `.rda` (`pos_ALL.rda`, `neg_ALL.rda`): "R-native | `.rda` (`pos_ALL.rda`, `neg_ALL.rda`)"
- [discussion] regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
- [discussion] Furthermore, the combination of EQ and MS-DIAL public database also showed superior performance to MS-DIAL program: "Furthermore, the combination of EQ and MS-DIAL public database also showed superior performance to MS-DIAL program"
- [readme] Theoretical library size | 168.6 million fragmentation entries (all chain compositions + C=C double-bond positions): "Theoretical library size | 168.6 million fragmentation entries (all chain compositions + C=C double-bond positions)"
