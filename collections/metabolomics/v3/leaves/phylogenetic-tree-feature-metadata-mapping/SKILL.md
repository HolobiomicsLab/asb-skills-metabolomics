---
name: phylogenetic-tree-feature-metadata-mapping
description: Use when a Qemistree chemical hierarchy (e.g., qemistree.qza from make-hierarchy)
  exists for an untargeted LC-MS/MS feature table and you need chemical-class labels
  on tree tips, or class-level subtree patterns interpreted against sample metadata
  such as food origin.
when_to_use_negative:
- The feature list contains only MS1 features without MS/MS data or fingerprints —
  these cannot receive ClassyFire taxonomy and are filtered from the feature table.
- You only need pairwise chemical distances or the bare tree topology without chemical-ontology
  labels (make-hierarchy alone suffices).
- Structural annotations (neither spectral library ms2_smiles nor CSI:FingerID csi_smiles
  structures) are absent, so no 5-level taxonomy can be assigned.
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0084
tools:
- name: Qemistree (q2-qemistree)
  role: Provides get-classyfire-taxonomy, which assigns the 5-level ClassyFire taxonomy
    to features on the chemical hierarchy
  repo: https://github.com/biocore/q2-qemistree
- name: ClassyFire
  role: Underlying chemical-ontology service assigning kingdom/superclass/class/subclass/direct_parent
    to annotated molecules
- name: QIIME 2
  role: Plugin framework in which the taxonomy-annotated feature data are generated
    and downstream analyses run
- name: iTOL
  role: Interactive visualization of the Qemistree tree with ClassyFire labels and
    sample metadata
- name: Empress (q2-empress)
  role: Tree viewer for Qemistree trees with feature-metadata coloring and sample-metadata
    integration
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:0c472abdfb4fca703c8a8dae33085aa1379413e4f6753b16d6b8a7594ad0f0f2
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
  - Empress (q2-empress)
  - Python
  - QIIME 2
  - Qemistree
  - Qemistree (q2-qemistree)
  - iTOL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# phylogenetic-tree-feature-metadata-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate the chemical features on a Qemistree hierarchy with ClassyFire's 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent) and map those labels against sample metadata so that class-specific subtrees (e.g., acyl-carnitines, flavonoids) can be read off the tree as in the FoodOmics Figure 3a analysis.

## When to use

Use when a Qemistree chemical hierarchy (e.g., qemistree.qza from make-hierarchy) exists for an untargeted LC-MS/MS feature table and you need chemical-class labels on tree tips, or class-level subtree patterns interpreted against sample metadata such as food origin.

## When NOT to use

- The feature list contains only MS1 features without MS/MS data or fingerprints — these cannot receive ClassyFire taxonomy and are filtered from the feature table.
- You only need pairwise chemical distances or the bare tree topology without chemical-ontology labels (make-hierarchy alone suffices).
- Structural annotations (neither spectral library ms2_smiles nor CSI:FingerID csi_smiles structures) are absent, so no 5-level taxonomy can be assigned.

## Inputs

- Qemistree chemical hierarchy tree (e.g., qemistree.qza / Newick) from make-hierarchy
- Merged feature data with structural annotations (merged-feature-data.qza) including ms2_smiles and csi_smiles structures
- Feature table (BIOM) with MS/MS-containing features
- Sample metadata TSV defining biological groups (e.g., animal-derived vs. plant-derived vs. honey products)

## Outputs

- Classified feature data (classified-merged-feature-data.qza) with kingdom, superclass, class, subclass, and direct_parent columns plus structure_source
- Taxonomy-colored chemical tree visualization (iTOL or Empress/q2-empress .qzv) showing class-level subtrees
- Class-by-sample-membership check (e.g., acyl-carnitines restricted to animal products; flavonoid clade shared by plant products and honey)

## How to apply

Take the merged chemical hierarchy tree and merged feature data produced by the upstream make-hierarchy step, then run QIIME 2's `qiime qemistree get-classyfire-taxonomy --i-feature-data merged-feature-data.qza` to assign each molecule its ClassyFire kingdom, superclass, class, subclass, and direct_parent. The tool prioritizes MS/MS spectral library matches (ms2_smiles) over CSI:FingerID predictions (csi_smiles) for taxonomy assignment, recording the origin in the structure_source column — check this column to gauge annotation provenance. Export the classified feature data and inspect the 'class' column for target classes (e.g., flavonoids, alkaloids, phospholipids, acyl-carnitines, O-glycosyl compounds), then cross-check each class's feature membership against sample metadata groups (animal products, plant products, honey) to recover class-specific subtrees. Visualize the labeled tree interactively with iTOL or q2-empress, coloring tips by the taxonomy levels and sample metadata. The rationale is that spectral-match annotations alone cover too little of the tree, so the ClassyFire layer provides the chemical ontology needed to read ecological patterns off the hierarchy.

## Related tools

- **Qemistree (q2-qemistree)** (Provides get-classyfire-taxonomy, which assigns the 5-level ClassyFire taxonomy to features on the chemical hierarchy) — https://github.com/biocore/q2-qemistree
- **ClassyFire** (Underlying chemical-ontology service assigning kingdom/superclass/class/subclass/direct_parent to annotated molecules)
- **QIIME 2** (Plugin framework in which the taxonomy-annotated feature data are generated and downstream analyses run)
- **iTOL** (Interactive visualization of the Qemistree tree with ClassyFire labels and sample metadata)
- **Empress (q2-empress)** (Tree viewer for Qemistree trees with feature-metadata coloring and sample-metadata integration) — https://github.com/biocore/empress

## Examples

```
qiime qemistree get-classyfire-taxonomy --i-feature-data merged-feature-data.qza --o-classified-feature-data classified-merged-feature-data.qza
```

## Evaluation signals

- The classified feature data contain all five taxonomy columns: 'kingdom', 'superclass', 'class', 'subclass', and 'direct_parent'.
- The 'structure_source' column is populated and records whether each assignment came from ms2_smiles (spectral match) or csi_smiles (CSI:FingerID prediction), with ms2_smiles prioritized.
- Inspecting the 'class' column recovers expected classes such as flavonoids, alkaloids, phospholipids, acyl-carnitines, and O-glycosyl compounds.
- Class-level subtree pattern reproduces Figure 3a: acyl-carnitine branches restricted to animal products, and a flavonoid clade shared by plant products and honey but no other animal foods.
- Features lacking MS/MS data (MS1-only, no fingerprints) have been filtered from the feature table before taxonomy assignment.

## Limitations

- Typically only 1–20% of features have spectral library matches, depending on how well the sample type has been investigated, so most tips rely on predicted structures.
- Spectral-match annotations are more accurate but too low-coverage to summarize the chemical content of complex samples on their own.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees and sparser taxonomy.

## Evidence

- [results] Acyl-carnitine subtrees discriminate animal products in FoodOmics Fig. 3a: [evidence span withheld — anchor status: not_found]
- [readme] Empress can view trees of metabolomics data generated using Qemistree: [evidence span withheld — anchor status: external_doc]
