---
name: recall-comparison-visualization
description: Use when a ground-truth list of reported lipids (e.g. 2704 lipids from
  an aging-associated lipidome atlas) exists and you must quantify and plot how recall
  differs between spectral library levels or annotation backends.
when_to_use_negative:
- There is no reference/ground-truth lipid list — recall is undefined without a set
  of reported lipids to recover; use coverage or FDR metrics instead
- The goal is de novo discovery of lipids not present in any reference atlas, where
  recall against a fixed lipid list would trivially cap at the reference content
- Input is already a processed feature/annotation table where only one library was
  ever used — there is no cross-library comparison to visualize
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: LipidIN
  role: Provides the EQ-module MS2 library search, the hierarchical 1st–5th-level
    library (R-native pos_ALL.rda/neg_ALL.rda), the WMYn level-5 fingerprints, and
    the LCI module's three relative retention time rules used to filter false positives
    before recall is scored
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: 'Baseline annotation backend: the MS-DIAL public database searched with the
    same EQ module provides the comparator configuration against which LipidIN hierarchical-library
    recall is plotted'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:d1bbc91c25f4d579f139509b09788f7b57f251080199842fd99146a283bcc715
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# recall-comparison-visualization

## Summary

Computes and visualizes annotation recall (fraction of reference lipids recovered) across competing spectral library configurations — e.g. LipidIN's hierarchical library truncated at level 4 vs the full 1st–5th-level library containing WMYn-regenerated fingerprints vs the MS-DIAL public database — broken down per lipid subclass.

## When to use

Use when a ground-truth list of reported lipids (e.g. 2704 lipids from an aging-associated lipidome atlas) exists and you must quantify and plot how recall differs between spectral library levels or annotation backends.

## When NOT to use

- There is no reference/ground-truth lipid list — recall is undefined without a set of reported lipids to recover; use coverage or FDR metrics instead
- The goal is de novo discovery of lipids not present in any reference atlas, where recall against a fixed lipid list would trivially cap at the reference content
- Input is already a processed feature/annotation table where only one library was ever used — there is no cross-library comparison to visualize

## Inputs

- Reference list of reported/expected lipids (e.g. the 2704 lipids from the published aging-associated lipidome atlas in mice) with subclass labels
- Acquired MS/MS spectra from the dataset (mzML or vendor raw files)
- LipidIN R-native hierarchical spectral library (pos_ALL.rda / neg_ALL.rda, Zenodo record 13350719), truncatable to levels 1–4 or used at levels 1–5
- MS-DIAL public database for the baseline configuration
- Annotation result tables from each library configuration (post-LCI filtering)

## Outputs

- recall_comparison.csv — overall and per-subclass recall for each library configuration (level 1–4, level 1–5, MS-DIAL public database)
- recall_fig9ab.png — Supplementary Fig. 9a–b-style per-subclass recall comparison plot
- Validation verdict: agreement of level-5 recall with the reported 93.64% and the expected ~20% WMYn-driven gain

## How to apply

Load the reference lipid list and run each library configuration through the same search backend: the LipidIN EQ-module MS2 library search with the hierarchical library truncated to levels 1–4 (chain compositions without WMYn-regenerated fingerprints), the full 1st-to-5th-level library where level 5 comprises the Wide-spectrum Modeling Yield network (WMYn) regenerated lipid fragment fingerprints, and a baseline EQ search against the MS-DIAL public database. Apply the LipidIN LCI module's three relative retention time rules to filter false-positive annotations identically in all configurations before scoring, so the comparison isolates library content rather than filtering stringency. Compute overall and per-lipid-subclass recall as the fraction of reference lipids recovered by each configuration, then export a recall_comparison.csv table plus a per-subclass grouped recall plot in the style of Supplementary Fig. 9a–b (recall_fig9ab.png). Judge success by whether the level-5 configuration reproduces the reported recall (93.64% on the 2704 aging-atlas lipids) and whether the level-5 vs level-4 difference per subclass reflects the expected ~20% WMYn-driven recall gain concentrated in subclasses requiring fingerprint regeneration.

## Related tools

- **LipidIN** (Provides the EQ-module MS2 library search, the hierarchical 1st–5th-level library (R-native pos_ALL.rda/neg_ALL.rda), the WMYn level-5 fingerprints, and the LCI module's three relative retention time rules used to filter false positives before recall is scored) — https://github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Baseline annotation backend: the MS-DIAL public database searched with the same EQ module provides the comparator configuration against which LipidIN hierarchical-library recall is plotted)

## Examples

```
import pandas as pd; df = pd.read_csv('annotations_levels1_5.csv'); df.groupby('subclass')['recalled'].mean().to_csv('recall_comparison.csv')
```

## Evaluation signals

- Level-5 (1st-to-5th-level) library recall on the 2704 aging-atlas lipids reproduces the reported 93.64%; deviations beyond the reported values indicate a failed reproduction
- The level-5 vs level-4 recall difference per subclass matches the expected ~20% WMYn-driven recall gain reported for fingerprint regeneration
- The per-subclass recall pattern of recall_fig9ab.png matches the published Supplementary Fig. 9a–b
- The LCI module's three relative retention time rules were applied to all configurations before recall computation (identical filtering across arms)
- recall_comparison.csv contains a recall fraction for every lipid subclass × library-configuration combination, with the MS-DIAL baseline consistently below the LipidIN level-5 configuration

## Limitations

- Recall is bounded by the reference atlas: it measures recovery of reported lipids, not true lipidome coverage, and the ~20% WMYn gain is an estimate
- The article states future work must pre-train on 'a larger and more diverse dataset to extend reverse lipidomics', so WMYn-regenerated fingerprints may transfer poorly to sample matrices, instruments, or analytical methods underrepresented in training
- Level-5 fingerprints depend on WMYn regeneration quality; in subclasses where fingerprints are unreliable, the level-4 vs level-5 recall comparison may not follow the expected pattern

## Evidence

- [abstract] LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting.: "LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting."
- [discussion] regeneration of lipid fingerprints as the 5th-level library: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
- [other] LipidIN achieved 93.64% recall of the 2704 reported lipids when using the 1st-to-5th-level hierarchical library, which outperformed annotation based on the MS-DIAL public database.: [evidence span withheld — anchor status: not_found]
- [other] Apply the LCI module's three relative retention time rules to filter false-positive annotations in all search results.: [evidence span withheld — anchor status: not_found]
- [readme] Annotation — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions).: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)."
