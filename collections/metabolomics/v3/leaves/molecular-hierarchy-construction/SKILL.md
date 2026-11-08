---
name: molecular-hierarchy-construction
description: Use when an MGF file of MS2 spectra and a feature table of MS1 peak areas
  are available and samples must be compared by chemical similarity rather than by
  low-coverage spectral library matches alone.
when_to_use_negative:
- Features lack MS/MS data entirely (only MS1 features are not considered by Qemistree)
- The goal is metabolite identification from accurate spectral library matches only;
  spectral matches may be more accurate but their coverage (typically 1–20%) is too
  low to summarize complex samples — the hierarchy serves chemical-relationship analysis,
  not ID confirmation
- The dataset is in negative ionization mode with very few reference spectra available,
  where fewer fingerprints can be confidently predicted and the resulting trees are
  less extensive
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: Qemistree
  role: QIIME 2 plugin (q2-qemistree) providing compute-fragmentation-trees, rerank-molecular-formulas,
    predict-fingerprints, make-hierarchy, get-classyfire-taxonomy, and prune-hierarchy
    methods
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Determines the molecular formula of each feature using isotope and fragmentation
    patterns and estimates the best fragmentation tree; molecular substructures predicted
    by SIRIUS are used to build the hierarchy
- name: CSI:FingerID
  role: Operates on fragmentation trees using kernel support vector machines to predict
    molecular properties (2936 properties) used as the fingerprint vector for clustering
- name: GNPS
  role: Provides MS2 spectral library matches (via Feature-Based Molecular Networking)
    that are merged into the hierarchy through make-hierarchy --i-ms2-matches
- name: ClassyFire
  role: Assigns a 5-level chemical taxonomy to molecules annotated via spectral library
    matching and in silico prediction
- name: MZmine2
  role: Upstream feature detection producing the MGF and feature table inputs
- name: QIIME 2
  role: Framework into which Qemistree outputs (tree, feature table, feature data)
    plug for downstream tree-informed analyses
- name: Empress
  role: Visualization of Qemistree chemical trees via q2-empress or standalone tree-plot/community-plot
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:33b5ec3c45e9321bd0d177bc7e4ce39953a81140313444e85a5e2bd4b9a9a8d3
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
  - CSI:FingerID
  - ClassyFire
  - Empress
  - GNPS
  - MZmine2
  - QIIME 2
  - Qemistree
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# molecular-hierarchy-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a chemical hierarchy (tree) of LC-MS/MS features by predicting molecular fingerprints with SIRIUS/CSI:FingerID and hierarchically clustering the fingerprint vectors, enabling tree-informed, chemically aware comparison of metabolomics samples.

## When to use

Use when an MGF file of MS2 spectra and a feature table of MS1 peak areas are available and samples must be compared by chemical similarity rather than by low-coverage spectral library matches alone.

## When NOT to use

- Features lack MS/MS data entirely (only MS1 features are not considered by Qemistree)
- The goal is metabolite identification from accurate spectral library matches only; spectral matches may be more accurate but their coverage (typically 1–20%) is too low to summarize complex samples — the hierarchy serves chemical-relationship analysis, not ID confirmation
- The dataset is in negative ionization mode with very few reference spectra available, where fewer fingerprints can be confidently predicted and the resulting trees are less extensive

## Inputs

- MGF file containing MS1 and MS2 entries from MZmine2 feature detection (e.g. sirius.mgf)
- Feature table of MS1 peak areas per sample (e.g. feature-table.biom)
- QIIME 2 artifacts of type MassSpectrometryFeatures and FeatureTable[Frequency]
- Optional: GNPS FBMN 'clusterinfo_summary' tsv of MS2 spectral library matches as FeatureData[Molecules]

## Outputs

- Fragmentation trees (SiriusFolder artifact)
- Reranked molecular formulas (top-scoring formula per feature)
- CSI:FingerID substructure probabilities for 2936 molecular properties per feature
- Rooted chemical tree (Phylogeny[Rooted] artifact)
- Hashed/merged feature table with features lacking fingerprints filtered out
- FeatureData[Molecules] with featureID, parent_mass, retention_time, csi_smiles, ms2_smiles, table_number columns
- ClassyFire 5-level chemical taxonomy (kingdom/superclass/class/subclass/direct_parent)

## How to apply

After feature detection (e.g. MZmine2) produces an MGF with MS1/MS2 entries and a feature table of MS1 peak areas, import them into QIIME 2 artifacts (MassSpectrometryFeatures and FeatureTable[Frequency]). Run SIRIUS via 'qiime qemistree compute-fragmentation-trees' with --p-ppm-max 15, --p-profile orbitrap, and --p-ions-considered '[M+H]+' (adjust ion mode per dataset), then rerank formulas with --p-zodiac-threshold 0.95 and predict fingerprints with --p-ppm-max 20 to obtain CSI:FingerID probabilities for 2936 molecular properties. Build the hierarchy with 'qiime qemistree make-hierarchy', optionally passing --i-ms2-matches from GNPS FBMN so spectral library matches are merged with in silico predictions; MS1 features without fingerprints are filtered out of the resulting feature table. Optionally annotate the merged FeatureData[Molecules] with 'qiime qemistree get-classyfire-taxonomy' to add a 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent). The fingerprint-based representation is largely independent of technical variation such as chromatography shifts, so independently processed datasets can be compared without repeating feature detection or alignment.

## Related tools

- **Qemistree** (QIIME 2 plugin (q2-qemistree) providing compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints, make-hierarchy, get-classyfire-taxonomy, and prune-hierarchy methods) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Determines the molecular formula of each feature using isotope and fragmentation patterns and estimates the best fragmentation tree; molecular substructures predicted by SIRIUS are used to build the hierarchy)
- **CSI:FingerID** (Operates on fragmentation trees using kernel support vector machines to predict molecular properties (2936 properties) used as the fingerprint vector for clustering)
- **GNPS** (Provides MS2 spectral library matches (via Feature-Based Molecular Networking) that are merged into the hierarchy through make-hierarchy --i-ms2-matches)
- **ClassyFire** (Assigns a 5-level chemical taxonomy to molecules annotated via spectral library matching and in silico prediction)
- **MZmine2** (Upstream feature detection producing the MGF and feature table inputs)
- **QIIME 2** (Framework into which Qemistree outputs (tree, feature table, feature data) plug for downstream tree-informed analyses)
- **Empress** (Visualization of Qemistree chemical trees via q2-empress or standalone tree-plot/community-plot) — https://github.com/biocore/empress

## Examples

```
qiime qemistree compute-fragmentation-trees --i-features features.qza --p-ppm-max 15 --p-profile orbitrap --p-ions-considered '[M+H]+' --o-trees siriusfolder.qza
```

## Evaluation signals

- make-hierarchy outputs a valid QIIME 2 Phylogeny[Rooted] artifact
- The merged FeatureData[Molecules] contains populated featureID, parent_mass, retention_time, csi_smiles, ms2_smiles, and table_number columns (structure_source populated when MS2 matches are merged)
- Features without fingerprints are filtered out of the merged feature table (feature count drops accordingly)
- In the Global FoodOmics reproducibility, ~91% of the 663 quality-filtered features have fingerprint-annotated structures and ~92% of structures receive ClassyFire assignments; annotation rates within a few percentage points of these values indicate correct execution
- Tree can be visualized (e.g. via Empress or iTOL) and supports tree-informed metrics such as Weighted UniFrac / Faith's Phylogenetic Diversity on the chemical hierarchy

## Limitations

- Qemistree depends on representing chemical features as molecular fingerprints and shares limitations with the underlying fingerprint prediction tool CSI:FingerID, including dependence on the quality and coverage of MS/MS spectral databases used to train the models
- Spectral match annotations alone have coverage too low (typically 1–20% of features) to summarize the chemical content of complex samples
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees
- Different tree-building approaches produce different tree topologies capturing complementary chemical information; broader benchmarking is needed to decide when each approach should be used
- SIRIUS uses a specific PubChem release (version downloaded 13 August 2017 in the paper's version), so formula/formula-candidate coverage is tied to that database snapshot

## Evidence
