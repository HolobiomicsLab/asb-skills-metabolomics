---
name: relative-retention-time-classification
description: Use when LC-MS/MS lipidomics annotations from MS2 library search contain
  false positives, and chromatographic retention-order information (relative RT across
  lipid subclasses) is available to filter or extend them.
when_to_use_negative:
- Direct-infusion or non-chromatographic MS/MS data — relative retention time rules
  require aligned LC retention orders and cannot be applied
- Raw spectra not yet searched against a library — RT classification is a downstream
  filter/predictor, not an MS2 search step (run EQ/MS2 library search first)
- Tasks needing exact C=C positional isomer localization by orthogonal chemistry —
  that requires Paternò–Büchi (PB) derivatization and the PB spectral library, not
  LCI RT rules
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: LipidIN LCI module
  role: Implements the three relative retention time rules with a best-first search
    heuristic to predict lipid subclass and remove false positive annotations
  repo: github.com/LinShuhaiLAB/LipidIN
- name: LipidIN EQ module
  role: Upstream MS2 library search against the LipidIN theoretical library whose
    candidate annotations LCI filters by retention time
  repo: github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Produces peak/feature tables (m/z, RT, MS/MS) that feed the RT-based classification
    workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:afafa1c281778b91b99aa99453fc164647d054ca988607f4c8a22e5ce5ece270
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - LipidIN EQ module
  - LipidIN LCI module
  - LipidIN WMYn module
  - MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# relative-retention-time-classification

## Summary

Applies LipidIN's LCI (lipid categories intelligence) module — three relative retention time rules driven by a best-first search heuristic over predicted lipid subclasses — to remove false positive MS2 annotations and predict unannotated lipids, achieving an estimated ~5.7% false discovery rate across 8,923 lipids.

## When to use

Use when LC-MS/MS lipidomics annotations from MS2 library search contain false positives, and chromatographic retention-order information (relative RT across lipid subclasses) is available to filter or extend them.

## When NOT to use

- Direct-infusion or non-chromatographic MS/MS data — relative retention time rules require aligned LC retention orders and cannot be applied
- Raw spectra not yet searched against a library — RT classification is a downstream filter/predictor, not an MS2 search step (run EQ/MS2 library search first)
- Tasks needing exact C=C positional isomer localization by orthogonal chemistry — that requires Paternò–Büchi (PB) derivatization and the PB spectral library, not LCI RT rules

## Inputs

- Candidate lipid annotation table output from LipidIN EQ MS2 library search (feature m/z, RT, fragment spectra, chain-composition and C=C-position candidates)
- Retention-time-aligned LC-MS/MS feature tables (e.g., Thermo .raw / SCIEX .wiff / .mzML-derived peak tables processed via MS-DIAL)
- LipidIN theoretical spectral libraries: pos_ALL.rda / neg_ALL.rda or pos_common.pkl / neg_common.pkl (24,631,702 spectra)
- WMYn-regenerated 5th-level fingerprint library (for low-confidence lipid confirmation)

## Outputs

- Subclass-assigned, false-positive-filtered lipid annotation table with predicted subclasses for previously unannotated features
- Per-dataset estimated FDR values and an aggregated average FDR summary (reference: 5.69% at cutoff 2.4; 8,923 lipids; 471 unique lipids in the mixture dataset)
- annotation_report.csv containing per-dataset and aggregated annotation tables

## How to apply

After running LipidIN EQ's MS2 library search against the theoretical library (chain compositions + C=C double-bond positions, ~168.6 million fragmentation entries; .rda files pos_ALL.rda/neg_ALL.rda or Python .pkl libraries from Zenodo record 21421866), feed candidate annotations into the LCI module. LCI encodes three relative retention time rules mapping lipid structural features (head group, chain length, unsaturation) to retention order, and uses a best-first search (BFS) heuristic to sort node expansions and predict each feature's lipid subclass; inconsistent RT-order candidates are removed as false positives. Then enforce the strict ECN-rule (equivalent carbon number) and feature-peak criteria to discard remaining inconsistent annotations, optionally using the WMYn-regenerated 5th-level fingerprint library to confirm low-confidence lipids. Compute the estimated FDR per dataset (e.g., the reported 5.69% average FDR at a cutoff threshold of 2.4 across RBL-2H3 cells, mixture, human sera, and zebrafish tissues) and export per-dataset and aggregated annotation tables (annotation_report.csv).

## Related tools

- **LipidIN LCI module** (Implements the three relative retention time rules with a best-first search heuristic to predict lipid subclass and remove false positive annotations) — github.com/LinShuhaiLAB/LipidIN
- **LipidIN EQ module** (Upstream MS2 library search against the LipidIN theoretical library whose candidate annotations LCI filters by retention time) — github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Produces peak/feature tables (m/z, RT, MS/MS) that feed the RT-based classification workflow)

## Evaluation signals

- Estimated FDR at the strict cutoff (threshold 2.4) falls near the reported ~5.69% average across the four datasets (RBL-2H3, mixture, human sera, zebrafish)
- Total annotated lipids approaches the reported 8,923 lipids, with ~471 unique lipids in the mixture dataset (matching Fig. 3e–h and Supplementary Data 6–7)
- Predicted subclasses are consistent with equivalent carbon number (ECN-rule) and feature-peak criteria after strict filtering
- Retained annotations show retention order consistent with the three RRT rules (head group, chain length, unsaturation dependence)

## Limitations

- The deeper relationship between metabolites and relative RT remains uninvestigated and is deferred to future work in the paper's discussion
- FDR is estimated, not measured, and performance depends on RT alignment quality across LC runs and instruments
- The R implementation is kept for reproducibility only and will no longer be updated; new work should use the Python LipidIN × Agent track (min. 16 GB RAM)

## Evidence

- [abstract] developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate, covering 8923 lipids cross: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate, covering 8923 lipids cross"
- [discussion] BFS heuristic: "We utilized BFS in this work to sort the sequence of node expansions according to a heuristic function"
- [intro] combining MS/MS and RT scores: "and that combining MS/MS- and RT-based scores could significantly improve annotation performance in LC-MS/MS-based lipidomics analyses"
- [readme] LipidIN theoretical library search: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)."
