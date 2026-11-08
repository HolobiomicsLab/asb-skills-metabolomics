---
name: foodomics-class-subtree-interpretation
description: Use when a Qemistree chemical hierarchy (qemistree.qza) merged with feature
  data across heterogeneous samples needs class-level interpretation — e.g., checking
  whether a ClassyFire class forms a subtree restricted to particular metadata groups
  (animal vs plant products, honey).
when_to_use_negative:
- Input features are still raw MS1/MS2 peak lists without a merged Qemistree hierarchy
  or SIRIUS/CSI:FingerID fingerprints — run compute-fragmentation-trees through make-hierarchy
  first.
- Structural annotations come only from spectral matches with low coverage (typically
  1–20% of features), insufficient to summarize complex-sample chemistry; use fingerprint-based
  subtrees instead of match-only interpretation.
- The goal is abundance/differential testing between groups rather than taxonomic
  containment of chemical classes within the hierarchy.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: Qemistree
  role: Builds the merged chemical hierarchy whose ClassyFire-annotated subtrees are
    interpreted against sample metadata
  repo: https://github.com/biocore/q2-qemistree
- name: ClassyFire
  role: Assigns the 5-level chemical taxonomy (kingdom–superclass–class–subclass–direct
    parent) used to label class-level subtrees
- name: QIIME 2
  role: Hosts the q2-qemistree plugin commands (get-classyfire-taxonomy, prune-hierarchy)
    used to annotate and filter the hierarchy
  repo: https://github.com/biocore/q2-qemistree
- name: iTOL
  role: Interactive visualization of the annotated chemical tree to inspect class-subtree
    containment patterns
- name: Empress
  role: QIIME 2 tree viewer that can visualize Qemistree trees with metadata coloring
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:7c42a3761900d93d8f8d79abaf7f94e38e0843c6d1a8eaa7ac9a5c750da06364
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
  - Empress
  - Python
  - QIIME 2
  - Qemistree
  - iTOL
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 30
---

# foodomics-class-subtree-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpreting ClassyFire-annotated Qemistree chemical hierarchies by reading class-level subtrees (e.g., acyl-carnitines, flavonoids) against sample metadata to detect class-specific chemical signatures discriminating sample types such as animal-derived vs plant-derived food products.

## When to use

Use when a Qemistree chemical hierarchy (qemistree.qza) merged with feature data across heterogeneous samples needs class-level interpretation — e.g., checking whether a ClassyFire class forms a subtree restricted to particular metadata groups (animal vs plant products, honey).

## When NOT to use

- Input features are still raw MS1/MS2 peak lists without a merged Qemistree hierarchy or SIRIUS/CSI:FingerID fingerprints — run compute-fragmentation-trees through make-hierarchy first.
- Structural annotations come only from spectral matches with low coverage (typically 1–20% of features), insufficient to summarize complex-sample chemistry; use fingerprint-based subtrees instead of match-only interpretation.
- The goal is abundance/differential testing between groups rather than taxonomic containment of chemical classes within the hierarchy.

## Inputs

- qemistree.qza — merged chemical hierarchy tree from make-hierarchy on the complex foodomics dataset
- merged-feature-data.qza — merged feature table with peak areas and MS/MS-based annotations
- sample metadata mapping features/samples to categories such as animal products, plant products, honey

## Outputs

- classified-merged-feature-data.qza — feature data with kingdom, superclass, class, subclass, direct_parent and structure_source columns
- class-level subtree assignments per feature (e.g., flavonoids, acyl-carnitines, O-glycosyl compounds) linked to metadata groups

## How to apply

First run `qiime qemistree get-classyfire-taxonomy` on the merged feature data produced by the upstream make-hierarchy step, so each molecule is classified into the ClassyFire 5-level taxonomy (kingdom, superclass, class, subclass, direct_parent). The tool prioritizes MS/MS spectral library matches (ms2_smiles) over CSI:FingerID predictions (csi_smiles) for taxonomy assignment, and the structure_source column records which structure was used. Export the classified feature data and inspect the 'class' column to select features annotated as flavonoids, alkaloids, phospholipids, acyl-carnitines, or O-glycosyl compounds, then cross-reference their sample membership against metadata (animal products, plant products, honey). A class is interpreted as discriminating when its branches appear exclusively in one metadata group — e.g., in the Global FoodOmics complex foodomics dataset, acyl-carnitine branches were exclusively found in animal products, and a flavonoid clade was shared by plant products and honey but no other animal foods (Fig. 3a). Visualize the annotated tree interactively with iTOL or Empress to confirm subtree containment.

## Related tools

- **Qemistree** (Builds the merged chemical hierarchy whose ClassyFire-annotated subtrees are interpreted against sample metadata) — https://github.com/biocore/q2-qemistree
- **ClassyFire** (Assigns the 5-level chemical taxonomy (kingdom–superclass–class–subclass–direct parent) used to label class-level subtrees)
- **QIIME 2** (Hosts the q2-qemistree plugin commands (get-classyfire-taxonomy, prune-hierarchy) used to annotate and filter the hierarchy) — https://github.com/biocore/q2-qemistree
- **iTOL** (Interactive visualization of the annotated chemical tree to inspect class-subtree containment patterns)
- **Empress** (QIIME 2 tree viewer that can visualize Qemistree trees with metadata coloring) — https://github.com/biocore/empress

## Examples

```
qiime qemistree get-classyfire-taxonomy --i-feature-data merged-feature-data.qza --o-classified-feature-data classified-merged-feature-data.qza
```

## Evaluation signals

- classified-merged-feature-data.qza contains the columns 'kingdom', 'superclass', 'class', 'subclass', 'direct_parent' and 'structure_source'
- structure_source values correctly distinguish ms2_smiles (spectral library match) from csi_smiles (CSI:FingerID prediction) for each feature
- Class-level subtree pattern reproduces Figure 3a: acyl-carnitine branches restricted to animal products; flavonoid clade shared by plant products and honey but no other animal foods
- Features lacking ClassyFire taxonomy are pruned (prune-hierarchy) so remaining branches carry annotations
- Annotated tree visualized in iTOL/Empress shows the identified class clades nesting within expected metadata groups

## Limitations

- Fingerprint prediction depends on SIRIUS/CSI:FingerID model quality and the coverage of MS/MS spectral databases used for training — low-coverage classes may form incomplete or missing subtrees
- In negative ionization mode fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees
- Qemistree depends on representing chemical features as molecular fingerprints and shares limitations with the underlying fingerprint prediction tool CSI:FingerID
- Only chemical features with MS/MS data are included; MS1-only features without fingerprints are filtered out of the feature table
- Different tree-building approaches produce different tree topologies capturing complementary chemical information, so class-subtree conclusions depend on the chosen hierarchy construction

## Evidence

- [results] In the ClassyFire-labeled FoodOmics chemical tree, branches representing acyl-carnitines were exclusively found in animal products (Fig. 3a): [evidence span withheld — anchor status: not_found]
- [supplementary] The tool prioritizes MS/MS spectral library matches over CSI:FingerID predictions, recording structure_source per feature: [evidence span withheld — anchor status: not_found]
- [readme] iTOL or Empress visualize the annotated tree so class clades can be inspected interactively: [evidence span withheld — anchor status: external_doc]
