---
name: metabolomics-feature-table-processing
description: Use when untargeted LC-MS/MS outputs (feature/quant tables plus MGF MS/MS
  spectra) must be turned into a chemical hierarchy for sample-level chemical comparison
  — e.g., weighted UniFrac on Qemistree trees vs Bray-Curtis on food metabolomics
  data.
when_to_use_negative:
- 'Features lack MS/MS spectra (MS1-only): fingerprints cannot be predicted and such
  features are explicitly excluded, so the hierarchy will not represent the full table.'
- Goal is high-confidence identification of individual metabolites by library matching
  — spectral match coverage (typically 1-20% of features) is too low to summarize
  the chemical content of complex samples.
- A chemical hierarchy already exists (e.g., GNPS cosine-score hierarchy imported
  as FeatureData[Molecules]) and only downstream UniFrac/ordination is needed — no
  table-to-tree fingerprint processing required.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0084
tools:
- name: Qemistree (q2-qemistree)
  role: QIIME 2 plugin that constructs the chemical hierarchy from predicted fingerprints
    (compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints,
    make-hierarchy, prune-hierarchy, get-classyfire-taxonomy)
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Determines molecular formulas and best fragmentation trees from MGF spectra
    inside compute-fragmentation-trees; formulas reranked with Zodiac (--p-zodiac-threshold
    0.95)
  repo: https://bio.informatik.uni-jena.de/sirius/
- name: CSI:FingerID
  role: Predicts the 2936 molecular properties per feature from fragmentation trees
    using kernel support vector machines (predict-fingerprints)
- name: QIIME 2
  role: Framework hosting q2-qemistree commands and tree-informed diversity computation
    (e.g., weighted UniFrac on the chemical hierarchy)
  repo: https://docs.qiime2.org/2019.7/install/
- name: GNPS
  role: Feature-based molecular networking library matches supply the MS/MS cosine-score
    hierarchy imported as FeatureData[Molecules]
  repo: https://gnps.ucsd.edu/
- name: MZmine2
  role: Upstream feature detection producing the feature quant table and MGF of fragmentation
    spectra from .mzXML/.mzML/.mzDATA files
  repo: http://mzmine.github.io
- name: ClassyFire
  role: Assigns the 5-level chemical taxonomy (chemical kingdom to direct parent)
    to molecules annotated via spectral matching and in silico prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:e1d55722d2dd9d2d504db69af5d015272baf54f15663f3ea9f0abade7d844a07
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
  - GNPS
  - MZmine2
  - Python
  - QIIME 2
  - Qemistree
  - Qemistree (q2-qemistree)
  - R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7741935483870968
n_claims: 31
---

# metabolomics-feature-table-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts untargeted LC-MS/MS feature tables (MZmine2 quant tables plus MGF fragmentation spectra) into predicted molecular fingerprints (SIRIUS/CSI:FingerID, 2936 properties) and Qemistree chemical hierarchies, enabling chemically informed sample dissimilarity (weighted UniFrac) benchmarked against tree-agnostic metrics (Bray-Curtis, CSCS). It is the data-processing backbone for tree-based metabolomics analysis in QIIME 2.

## When to use

Use when untargeted LC-MS/MS outputs (feature/quant tables plus MGF MS/MS spectra) must be turned into a chemical hierarchy for sample-level chemical comparison — e.g., weighted UniFrac on Qemistree trees vs Bray-Curtis on food metabolomics data.

## When NOT to use

- Features lack MS/MS spectra (MS1-only): fingerprints cannot be predicted and such features are explicitly excluded, so the hierarchy will not represent the full table.
- Goal is high-confidence identification of individual metabolites by library matching — spectral match coverage (typically 1-20% of features) is too low to summarize the chemical content of complex samples.
- A chemical hierarchy already exists (e.g., GNPS cosine-score hierarchy imported as FeatureData[Molecules]) and only downstream UniFrac/ordination is needed — no table-to-tree fingerprint processing required.

## Inputs

- MZmine2 feature quant table of ion peak areas (e.g., feature-table.biom or quant-table CSV)
- MGF file of MS/MS fragmentation spectra (e.g., sirius.mgf)
- Raw LC-MS/MS files (.mzXML, .mzML, .mzDATA) for upstream MZmine2 feature detection
- GNPS feature-based molecular networking matches imported as FeatureData[Molecules] (for the cosine-score hierarchy)
- Sample metadata with food ontology levels 1-4 categories

## Outputs

- Per-feature predicted fingerprint table with 2936 molecular properties (QIIME 2 artifact)
- Chemical hierarchy tree (Newick/QIIME 2 phylogeny) from make-hierarchy, prunable via prune-hierarchy
- ClassyFire 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent)
- Sample-by-sample dissimilarity matrices: weighted UniFrac (fingerprint hierarchy), Bray-Curtis (feature table), CSCS, weighted UniFrac (MS/MS cosine-score hierarchy)
- Per-metric, per-ontology-level comparison table (extended_data_table4_metric_comparison.csv)

## How to apply

Detect features with MZmine2 from raw .mzXML/.mzML/.mzDATA files to produce a quant table of ion peak areas and an MGF of MS/MS spectra; only features with MS/MS data enter the pipeline — MS1-only features are excluded. In QIIME 2, run qiime qemistree compute-fragmentation-trees (SIRIUS, --p-ppm-max 15, --p-profile orbitrap), rerank-molecular-formulas (--p-zodiac-threshold 0.95), then predict-fingerprints so CSI:FingerID assigns 2936 molecular properties per feature (SIRIUS uses the PubChem snapshot of 13 August 2017; the demo runs on SIRIUS 4.9.3). Build the tree with qiime qemistree make-hierarchy, merging feature tables and filtering MS1 features without predicted fingerprints, optionally retaining MS/MS library matches via --i-ms2-matches, then prune-hierarchy to keep annotated molecules; add ClassyFire taxonomy with get-classyfire-taxonomy (5 levels: kingdom to direct parent). For a complementary tree, import GNPS feature-based molecular networking matches as FeatureData[Molecules] to build an MS/MS cosine-score hierarchy; --p-qc-properties subsets fingerprints to 489 PubChem positions when building contingency tables. Compute pairwise sample dissimilarity for all samples (e.g., N=126 Global FoodOmics foods) under weighted UniFrac on the fingerprint hierarchy, Bray-Curtis on the feature table, CSCS, and weighted UniFrac on the cosine hierarchy, then assess within-food vs between-food similarity per food ontology level 1-4. The rationale is that the fingerprint-based representation is largely independent of technical variation such as chromatography shifts, enabling cross-dataset comparison without repeating feature detection and alignment.

## Related tools

- **Qemistree (q2-qemistree)** (QIIME 2 plugin that constructs the chemical hierarchy from predicted fingerprints (compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints, make-hierarchy, prune-hierarchy, get-classyfire-taxonomy)) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Determines molecular formulas and best fragmentation trees from MGF spectra inside compute-fragmentation-trees; formulas reranked with Zodiac (--p-zodiac-threshold 0.95)) — https://bio.informatik.uni-jena.de/sirius/
- **CSI:FingerID** (Predicts the 2936 molecular properties per feature from fragmentation trees using kernel support vector machines (predict-fingerprints))
- **QIIME 2** (Framework hosting q2-qemistree commands and tree-informed diversity computation (e.g., weighted UniFrac on the chemical hierarchy)) — https://docs.qiime2.org/2019.7/install/
- **GNPS** (Feature-based molecular networking library matches supply the MS/MS cosine-score hierarchy imported as FeatureData[Molecules]) — https://gnps.ucsd.edu/
- **MZmine2** (Upstream feature detection producing the feature quant table and MGF of fragmentation spectra from .mzXML/.mzML/.mzDATA files) — http://mzmine.github.io
- **ClassyFire** (Assigns the 5-level chemical taxonomy (chemical kingdom to direct parent) to molecules annotated via spectral matching and in silico prediction)

## Examples

```
qiime qemistree compute-fragmentation-trees --p-ppm-max 15 --p-profile orbitrap --i-mgf-specs sirius.mgf --o-fragmentation-trees frag-trees.qza
```

## Evaluation signals

- Every tree tip has a predicted fingerprint: after make-hierarchy, MS1-only features and features without fingerprints are absent from the hierarchy (they are explicitly filtered).
- Fingerprint predictions contain the expected 2936 molecular properties per feature; --p-qc-properties subsets these to 489 PubChem fingerprint positions in contingency tables.
- Command logs show SIRIUS run with --p-ppm-max 15 and --p-profile orbitrap, formula reranking at --p-zodiac-threshold 0.95, against the PubChem snapshot of 13 August 2017.
- Recomputed per-metric, per-ontology-level counts reproduce the qualitative pattern of Extended Data Table 4: fingerprint-based weighted UniFrac supports chemically meaningful clustering at least as well as tree-agnostic Bray-Curtis across food ontology levels 1-4.
- For each food ontology level, within-food sample pairs are more chemically similar than between-food pairs for expected chemical classes (e.g., acyl-carnitine branches exclusively in animal products).

## Limitations

- Typically only 1-20% of features have spectral library matches, so match-based annotation coverage is too low to adequately summarize the chemical content of complex samples; fingerprint prediction is used to compensate.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.
- Qemistree depends on representing chemical features as molecular fingerprints and shares limitations with CSI:FingerID, whose prediction depends on the quality and coverage of MS/MS spectral databases used for training; SIRIUS searches a fixed PubChem release (13 August 2017).
- Different tree-building approaches (fingerprint-based vs MS/MS cosine-score) produce different tree topologies capturing complementary chemical information; a broader benchmarking effort would be needed to know when each approach should be used.
- Version dependence: Qemistree was initially developed under SIRIUS 4.0.1 and was adapted to work with new SIRIUS versions (>4.4.29; demo uses 4.9.3).

## Evidence
