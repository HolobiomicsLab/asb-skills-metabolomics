---
name: molecular-hierarchy-annotation
description: Use when a merged chemical hierarchy (e.g., qemistree.qza from make-hierarchy)
  and its merged feature data exist and you need ClassyFire taxonomy labels (e.g.,
  acyl-carnitines, flavonoids) to interpret chemical-class subtrees across sample
  types.
when_to_use_negative:
- Features are MS1-only without MS/MS data or resolved structures — they cannot be
  assigned ClassyFire taxonomy and are not the target of this annotation step
- The goal is to build or prune the chemical hierarchy itself (use make-hierarchy
  or prune-hierarchy) rather than label existing features with chemical ontology terms
- You need a complete chemical summary of a complex sample from spectral matches alone
  — spectral-match coverage is typically only 1–20% of features and too low to summarize
  chemical content
- Input is a raw feature table before fingerprint prediction/hierarchy construction
  — taxonomy annotation presumes structures already exist
tools:
- name: Qemistree
  role: Provides the get-classyfire-taxonomy method that attaches ClassyFire taxonomy
    to hierarchy feature data
  repo: https://github.com/biocore/q2-qemistree
- name: ClassyFire
  role: External chemical-ontology service assigning the 5-level taxonomy (kingdom,
    superclass, class, subclass, direct parent) to annotated molecules
- name: QIIME 2
  role: Plugin framework in which q2-qemistree methods (get-classyfire-taxonomy, prune-hierarchy)
    run and produce .qza artifacts
  repo: https://github.com/biocore/q2-qemistree
- name: iTOL
  role: Used to visualize the ClassyFire-labeled chemical tree to inspect class-specific
    subtrees
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:0b1472b7342095cfbc9481e9cc5fb0c4ed6c32fb2b78b808c7021b466932b155
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  repo_url: https://github.com/biocore/q2-qemistree
  tools:
  - ClassyFire
  - Python
  - QIIME 2
  - Qemistree
  - iTOL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 30
---

# molecular-hierarchy-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates each chemical feature in a Qemistree-built molecular hierarchy with the ClassyFire 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent), enabling class-level subtree interpretation of the chemical tree against sample metadata.

## When to use

Use when a merged chemical hierarchy (e.g., qemistree.qza from make-hierarchy) and its merged feature data exist and you need ClassyFire taxonomy labels (e.g., acyl-carnitines, flavonoids) to interpret chemical-class subtrees across sample types.

## When NOT to use

- Features are MS1-only without MS/MS data or resolved structures — they cannot be assigned ClassyFire taxonomy and are not the target of this annotation step
- The goal is to build or prune the chemical hierarchy itself (use make-hierarchy or prune-hierarchy) rather than label existing features with chemical ontology terms
- You need a complete chemical summary of a complex sample from spectral matches alone — spectral-match coverage is typically only 1–20% of features and too low to summarize chemical content
- Input is a raw feature table before fingerprint prediction/hierarchy construction — taxonomy annotation presumes structures already exist

## Inputs

- merged feature data (.qza) from the upstream make-hierarchy step, containing structures/SMILES for each chemical feature
- qemistree.qza merged chemical hierarchy tree
- sample metadata distinguishing sample types (e.g., animal products, plant products, honey)

## Outputs

- classified-merged-feature-data.qza with kingdom, superclass, class, subclass, and direct_parent columns
- structure_source column recording whether taxonomy came from ms2_smiles (spectral library match) or csi_smiles (CSI:FingerID prediction)

## How to apply

Run `qiime qemistree get-classyfire-taxonomy` on the merged feature data produced by the upstream make-hierarchy step of the complex foodomics dataset. The tool uses ClassyFire to assign the 5-level chemical taxonomy — chemical kingdom, superclass, class, subclass, and direct parent ontology — to molecules annotated via spectral library matching and in silico prediction. Verify that the tool prioritizes MS/MS spectral library matches (ms2_smiles) over CSI:FingerID predictions (csi_smiles), with the structure_source column recording the origin of each assignment; spectral matches are more accurate, so they take precedence when available. Export the classified feature data and inspect the 'class' column to identify features of interest (e.g., flavonoids, alkaloids, phospholipids, acyl-carnitines, O-glycosyl compounds), then cross-check membership against sample metadata (animal products, plant products, honey) to reproduce class-specific subtree patterns. Save the result as classified-merged-feature-data.qza for downstream visualization (e.g., iTOL or Empress) and validation.

## Related tools

- **Qemistree** (Provides the get-classyfire-taxonomy method that attaches ClassyFire taxonomy to hierarchy feature data) — https://github.com/biocore/q2-qemistree
- **ClassyFire** (External chemical-ontology service assigning the 5-level taxonomy (kingdom, superclass, class, subclass, direct parent) to annotated molecules)
- **QIIME 2** (Plugin framework in which q2-qemistree methods (get-classyfire-taxonomy, prune-hierarchy) run and produce .qza artifacts) — https://github.com/biocore/q2-qemistree
- **iTOL** (Used to visualize the ClassyFire-labeled chemical tree to inspect class-specific subtrees)

## Examples

```
qiime qemistree get-classyfire-taxonomy --i-feature-data merged-feature-data.qza --o-classified-feature-data classified-merged-feature-data.qza
```

## Evaluation signals

- The classified feature data contains all five expected columns: 'kingdom', 'superclass', 'class', 'subclass', and 'direct_parent'
- Every classified feature has a structure_source value of ms2_smiles or csi_smiles, confirming assignment provenance and spectral-match priority
- Class-level subtree pattern matches Figure 3a of the foodomics analysis: acyl-carnitine branches restricted to animal products, and a flavonoid clade shared by plant products and honey but absent from other animal foods
- Features of interest (flavonoids, alkaloids, phospholipids, acyl-carnitines, O-glycosyl compounds) can be retrieved by filtering on the 'class' column

## Limitations

- Typically only 1–20% of features have spectral library matches; most annotations come from in silico CSI:FingerID predictions whose accuracy depends on the quality and coverage of MS/MS spectral databases used for training
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees and sparser taxonomy coverage
- Qemistree depends on molecular fingerprints predicted by CSI:FingerID and shares its underlying limitations; different tree-building approaches produce different topologies capturing complementary chemical information

## Evidence
