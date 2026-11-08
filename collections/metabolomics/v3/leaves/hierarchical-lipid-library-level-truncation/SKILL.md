---
name: hierarchical-lipid-library-level-truncation
description: Use when benchmarking how much recall a deep-learning-regenerated (5th-level)
  lipid fragment library adds over a truncated 1st–4th-level or MS-DIAL public library,
  using a dataset with a known reference lipid list.
when_to_use_negative:
- The analysis is single-lipid targeted identification rather than a recall benchmark
  against a known reference lipid list.
- The goal is Paternò–Büchi double-bond localization, which uses LipidIN's dedicated
  PB libraries (e.g. PB_PX_H.pkl) rather than hierarchical level truncation.
- The dataset lacks a ground-truth reference set of reported lipids, so recall cannot
  be computed.
edam_topics:
- http://edamontology.org/topic_3294
tools:
- name: LipidIN
  role: Runs the EQ-module MS2 library search against truncated (1–4) and full (1–5)
    levels of its hierarchical library and applies the LCI module relative-RT false-positive
    filters
  repo: github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Public spectral database searched with the EQ module to provide the baseline
    recall against which the hierarchical library levels are compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:6ef9d92381507403ebe1132fd3e7857bf1412718a3dfa137f607d9c56adc2d4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3294
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hierarchical-lipid-library-level-truncation

## Summary

Benchmark strategy from LipidIN that compares MS2 library search recall across truncation levels of its hierarchical lipid fragmentation library (1st–4th vs 1st–5th level, the 5th level being WMYn-regenerated fingerprints) against the MS-DIAL public database on a reference lipidome dataset.

## When to use

Use when benchmarking how much recall a deep-learning-regenerated (5th-level) lipid fragment library adds over a truncated 1st–4th-level or MS-DIAL public library, using a dataset with a known reference lipid list.

## When NOT to use

- The analysis is single-lipid targeted identification rather than a recall benchmark against a known reference lipid list.
- The goal is Paternò–Büchi double-bond localization, which uses LipidIN's dedicated PB libraries (e.g. PB_PX_H.pkl) rather than hierarchical level truncation.
- The dataset lacks a ground-truth reference set of reported lipids, so recall cannot be computed.

## Inputs

- Published aging-associated lipidome atlas in mice LC-MS/MS dataset (reference list of 2704 reported lipids)
- LipidIN R-native hierarchical spectral libraries pos_ALL.rda and neg_ALL.rda (Zenodo record 13350719)
- MS-DIAL public database spectral library (baseline comparator)

## Outputs

- recall_comparison.csv with overall and per-lipid-subclass recall for level-4, level-5, and MS-DIAL configurations
- Per-subclass recall comparison plot in the style of Supplementary Fig. 9a–b (recall_fig9ab.png)

## How to apply

Load the target MS/MS dataset together with the spectral libraries (MS-DIAL public database and the LipidIN R-native pos_ALL.rda / neg_ALL.rda hierarchical library from Zenodo record 13350719) into the LipidIN framework. Run the EQ-module MS2 library search with the hierarchical library truncated to levels 1–4 (chain compositions without WMYn-regenerated fingerprints), then rerun the same search with the full 1st-to-5th-level library whose level 5 comprises Wide-spectrum Modeling Yield network (WMYn) regenerated lipid fragment fingerprints. Run a baseline EQ search against the MS-DIAL public database to situate performance. In all searches, apply the LCI module's three relative retention time rules to filter false-positive annotations (estimated FDR 5.7% in the paper). Compute overall and per-lipid-subclass recall as the fraction of reference lipids recovered for each configuration, and compare level-4 vs level-5 recall per subclass to quantify the WMYn-driven gain (reported as ~20%).

## Related tools

- **LipidIN** (Runs the EQ-module MS2 library search against truncated (1–4) and full (1–5) levels of its hierarchical library and applies the LCI module relative-RT false-positive filters) — github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Public spectral database searched with the EQ module to provide the baseline recall against which the hierarchical library levels are compared)

## Evaluation signals

- Level-5 library recall must reproduce the reported 93.64% of the 2704 reported lipids on the aging-associated lipidome atlas dataset.
- Level-5 recall should exceed level-4 recall by approximately the reported ~20% WMYn-driven recall gain.
- Per-subclass recall pattern must match the expected Supplementary Fig. 9a–b comparison; deviations beyond reported values indicate a failed reproduction.
- EQ + MS-DIAL public database baseline recall must fall below the full hierarchical-library recall, as reported in the paper.
- Annotations surviving the LCI module's three relative retention time rules should be consistent with the paper's 5.7% estimated false discovery rate.

## Limitations

- The R implementation is kept for reproducibility only and will no longer be updated; new analyses are directed to the Python LipidIN × Agent track.
- The 1st–4th-level library omits WMYn-regenerated fingerprints, so its recall is capped below the full 1st–5th-level library.
- The ~20% WMYn recall boost and 5.7% FDR are estimates reported by the authors; deviations beyond reported values indicate a failed reproduction.

## Evidence

- [abstract] LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting.: "LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting."
- [discussion] Level-5 library role: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
- [methods] Library source: "the original hierarchical / MS-DIAL libraries are at https://zenodo.org/records/13350719"
- [readme] Search mechanism: "Annotation — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)"
