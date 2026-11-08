---
name: chemical-taxonomy-assignment
description: Use when a merged Qemistree feature-data artifact already carries structures
  (ms2_smiles and/or csi_smiles) and you need ClassyFire chemical ontology labels
  per molecule to color tree subtrees, check class-level sample patterns, or compute
  class annotation coverage.
when_to_use_negative:
- The feature data artifact has no resolved structures because make-hierarchy has
  not yet been run or all features were filtered out as MS1-only without fingerprints
  — ClassyFire classification requires a SMILES structure.
- The goal is to assign taxonomy from raw spectra directly, without prior GNPS spectral
  library matching or CSI:FingerID fingerprint prediction — spectral matches alone
  typically cover only 1–20% of features and ClassyFire operates on annotated structures.
- Input is a feature table or MGF of raw MS/MS data rather than the merged feature-data
  artifact output by qemistree make-hierarchy.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: Qemistree
  role: Provides the get-classyfire-taxonomy QIIME 2 method that submits merged feature
    structures to ClassyFire and attaches the 5-level taxonomy columns
  repo: github.com/biocore/q2-qemistree
- name: ClassyFire
  role: Assigns the 5-level chemical taxonomy (kingdom, superclass, class, subclass,
    direct parent) to all molecules annotated via spectral library matching and in
    silico prediction
- name: GNPS
  role: Source of ms2_smiles spectral library match structures, which the tool prioritizes
    over CSI:FingerID predictions for taxonomy assignment
- name: CSI:FingerID
  role: Source of csi_smiles predicted structures used for taxonomy assignment when
    no spectral match exists
- name: QIIME 2
  role: Artifact framework (FeatureData[Molecules]) into which the classified feature
    data is written and from which downstream tree-based analyses consume it
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:220a7146812322ebbcc958e12cd06cfbb198518ddaeafad4ab67027d498511ad
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  tools:
  - CSI:FingerID
  - ClassyFire
  - GNPS
  - MZmine2
  - Python
  - QIIME 2
  - Qemistree
  - SIRIUS
  - iTOL
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chemical-taxonomy-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each MS1 feature a ClassyFire 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct_parent) using the structures obtained from GNPS MS2 spectral library matches and CSI:FingerID predictions, so that chemical classes (e.g., acyl-carnitines, flavonoids) can be traced as subtrees on the Qemistree chemical hierarchy.

## When to use

Use when a merged Qemistree feature-data artifact already carries structures (ms2_smiles and/or csi_smiles) and you need ClassyFire chemical ontology labels per molecule to color tree subtrees, check class-level sample patterns, or compute class annotation coverage.

## When NOT to use

- The feature data artifact has no resolved structures because make-hierarchy has not yet been run or all features were filtered out as MS1-only without fingerprints — ClassyFire classification requires a SMILES structure.
- The goal is to assign taxonomy from raw spectra directly, without prior GNPS spectral library matching or CSI:FingerID fingerprint prediction — spectral matches alone typically cover only 1–20% of features and ClassyFire operates on annotated structures.
- Input is a feature table or MGF of raw MS/MS data rather than the merged feature-data artifact output by qemistree make-hierarchy.

## Inputs

- merged-feature-data.qza — QIIME 2 FeatureData[Molecules] artifact from qiime qemistree make-hierarchy, containing SMILES structures (ms2_smiles / csi_smiles) and structure_source
- Optionally qemistree.qza (Phylogeny[Rooted]) for later visualization of the classified molecules on the chemical hierarchy

## Outputs

- classified-merged-feature-data.qza — FeatureData[Molecules] with kingdom, superclass, class, subclass, direct_parent, and structure_source columns
- Class-level annotation coverage metric (fraction of structures with ClassyFire assignments, ~92% in the food dataset)
- Per-class feature lists (e.g., acyl-carnitines, flavonoids) for tree visualization in iTOL or Empress

## How to apply

Run 'qiime qemistree get-classyfire-taxonomy' on the merged feature data produced by 'qiime qemistree make-hierarchy', which merges GNPS spectral-match structures with CSI:FingerID-predicted structures. The tool prioritizes MS/MS spectral library matches (ms2_smiles) over CSI:FingerID predictions (csi_smiles), and records which origin was used in a structure_source column. Each structure is classified into the ClassyFire 5-level chemical taxonomy (chemical kingdom, superclass, class, subclass, and direct parent ontology). Export the resulting FeatureData[Molecules] artifact (e.g., as classified-merged-feature-data.qza or TSV) and inspect the 'class' column to identify features belonging to classes of interest (e.g., flavonoids, alkaloids, phospholipids, acyl-carnitines, O-glycosyl compounds) before intersecting with sample metadata for tree/subtree interpretation. Because ClassyFire requires a resolved structure, this step should follow make-hierarchy's filtering of MS1 features without fingerprints.

## Related tools

- **Qemistree** (Provides the get-classyfire-taxonomy QIIME 2 method that submits merged feature structures to ClassyFire and attaches the 5-level taxonomy columns) — github.com/biocore/q2-qemistree
- **ClassyFire** (Assigns the 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent) to all molecules annotated via spectral library matching and in silico prediction)
- **GNPS** (Source of ms2_smiles spectral library match structures, which the tool prioritizes over CSI:FingerID predictions for taxonomy assignment)
- **CSI:FingerID** (Source of csi_smiles predicted structures used for taxonomy assignment when no spectral match exists)
- **QIIME 2** (Artifact framework (FeatureData[Molecules]) into which the classified feature data is written and from which downstream tree-based analyses consume it)

## Examples

```
qiime qemistree get-classyfire-taxonomy --i-feature-data merged-feature-data.qza --o-classified-feature-data classified-merged-feature-data.qza
```

## Evaluation signals

- The classified output contains all five expected columns: 'kingdom', 'superclass', 'class', 'subclass', and 'direct_parent'.
- A populated 'structure_source' column exists and records whether each assignment came from ms2_smiles (spectral match) or csi_smiles (CSI:FingerID prediction).
- Fraction of structures receiving ClassyFire assignments falls within a few percentage points of the reported ~92% on the complex foodomics dataset.
- Class-level subtree patterns are biologically coherent: acyl-carnitine branches restricted to animal products and a flavonoid clade shared by plant products and honey, matching Figure 3a of the paper.
- Output remains a valid QIIME 2 FeatureData[Molecules] artifact importable by downstream QIIME 2 tools.

## Limitations

- Only features that survived make-hierarchy filtering (MS1 features without fingerprints are removed) can be classified, so taxonomy coverage is bounded by fingerprint-prediction coverage.
- ClassyFire assignment quality depends on the structure being correct; CSI:FingerID predictions may be wrong, whereas spectral matches are more accurate but their coverage is too low to adequately summarize the chemical content of complex samples.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models.

## Evidence

- [results] methods — structure_source column records assignment origin.: [evidence span withheld — anchor status: not_found]
