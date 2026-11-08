---
name: chromatographic-batch-variation-assessment
description: Use when LC-MS/MS samples span two chromatographic batches (a metadata
  'batch' column) and you need to test whether chemical-hierarchy-based beta diversity
  mitigates the batch effect versus an unweighted feature-table distance.
when_to_use_negative:
- No chemical hierarchy is available for the features (weighted UniFrac requires a
  phylogeny; only the tree-agnostic comparison is possible)
- Sample metadata lacks a batch/experiment grouping, so no PERMANOVA hypothesis about
  chromatographic batches can be tested
- Features are only MS1 peaks without MS/MS spectra — such features cannot receive
  fingerprints and are excluded from the hierarchy
tools:
- name: QIIME 2
  role: Runs the beta diversity (Bray-Curtis), beta-phylogenetic (weighted UniFrac),
    PERMANOVA (beta-group-significance), and PCoA commands on the feature table and
    Qemistree hierarchy
  repo: https://github.com/biocore/q2-qemistree
- name: Qemistree
  role: Builds the chemical hierarchy from predicted molecular fingerprints that serves
    as the phylogeny input for weighted UniFrac; q2-qemistree plug-in methods include
    make-hierarchy and prune-hierarchy
  repo: https://github.com/biocore/q2-qemistree
- name: EMPress
  role: Visualizes the Qemistree chemical tree alongside PCoA ordinations for exploring
    batch patterns
  repo: https://github.com/biocore/empress
- name: Python
  role: Glue language for the QIIME 2 commands and system-wide metabolomic data analyses
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:ad766a9f69517aacdbcd7fc1a6fe6f5fef29bd07c5a3fd33c57756ffe6edc7ce
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
  - EMPress
  - Python
  - QIIME 2
  - Qemistree
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chromatographic-batch-variation-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies whether a tree-informed distance metric (weighted UniFrac on a Qemistree chemical hierarchy) reduces chromatography-driven batch signal relative to a tree-agnostic metric (Bray-Curtis), by comparing one-sided PERMANOVA pseudo-F statistics between batches in QIIME 2.

## When to use

Use when LC-MS/MS samples span two chromatographic batches (a metadata 'batch' column) and you need to test whether chemical-hierarchy-based beta diversity mitigates the batch effect versus an unweighted feature-table distance.

## When NOT to use

- No chemical hierarchy is available for the features (weighted UniFrac requires a phylogeny; only the tree-agnostic comparison is possible)
- Sample metadata lacks a batch/experiment grouping, so no PERMANOVA hypothesis about chromatographic batches can be tested
- Features are only MS1 peaks without MS/MS spectra — such features cannot receive fingerprints and are excluded from the hierarchy

## Inputs

- FeatureTable[Frequency] quant table (peak areas of chemical features, e.g., from MZmine2 processing of .mzXML/.mzML files)
- Qemistree chemical hierarchy artifact (Phylogeny[Rooted]) built from SIRIUS/CSI:FingerID-predicted molecular fingerprints
- Sample metadata TSV with a chromatographic 'batch' column

## Outputs

- Bray-Curtis distance matrix artifact (tree-agnostic)
- Weighted UniFrac distance matrix artifact (tree-informed)
- PERMANOVA beta-group-significance visualizations with pseudo-F and p-values per metric
- PCoA ordination artifacts and Emperor/EMPress .qzv interactive plots

## How to apply

Import the quant feature table (e.g., the 162-sample evaluation dataset quant table from MassIVE accession MSV000083306) as FeatureTable[Frequency] and the Qemistree chemical hierarchy (Phylogeny[Rooted]) into QIIME 2. Compute the tree-agnostic Bray-Curtis matrix with qiime diversity beta, and the tree-informed weighted UniFrac matrix with qiime diversity beta-phylogenetic using the Qemistree hierarchy as the phylogeny. Assign samples to chromatographic batches via the metadata batch column and run one-sided PERMANOVA with 999 permutations on both matrices using qiime diversity beta-group-significance. Generate PCoA ordinations from each distance matrix with qiime diversity pcoa and export interactive ordination plots (Emperor/EMPress qzv). The rationale: the fingerprint-based chemical representation is largely independent of chromatography shifts, so the tree-informed metric should yield a substantially lower pseudo-F for the batch grouping than the tree-agnostic metric.

## Related tools

- **QIIME 2** (Runs the beta diversity (Bray-Curtis), beta-phylogenetic (weighted UniFrac), PERMANOVA (beta-group-significance), and PCoA commands on the feature table and Qemistree hierarchy) — https://github.com/biocore/q2-qemistree
- **Qemistree** (Builds the chemical hierarchy from predicted molecular fingerprints that serves as the phylogeny input for weighted UniFrac; q2-qemistree plug-in methods include make-hierarchy and prune-hierarchy) — https://github.com/biocore/q2-qemistree
- **EMPress** (Visualizes the Qemistree chemical tree alongside PCoA ordinations for exploring batch patterns) — https://github.com/biocore/empress
- **Python** (Glue language for the QIIME 2 commands and system-wide metabolomic data analyses)

## Examples

```
qiime diversity beta-phylogenetic --i-table quant.qza --i-phylogeny qemistree-hierarchy.qza --p-metric weighted_unifrac --o-distance-matrix wunifrac.qza; qiime diversity beta-group-significance --i-distance-matrix wunifrac.qza --m-metadata-file metadata.tsv --m-metadata-column batch --p-method permanova --p-permutations 999 --o-visualization permanova-wunifrac.qzv
```

## Evaluation signals

- Recovered pseudo-F for the tree-agnostic Bray-Curtis PERMANOVA matches the reported 120.75 (p=0.001) on the 162-sample evaluation dataset
- Recovered pseudo-F for the tree-informed weighted UniFrac PERMANOVA matches the reported 18.2239 (p=0.001)
- Tree-informed pseudo-F is substantially lower than the tree-agnostic pseudo-F, indicating mitigation of the chromatographic batch effect
- Both PERMANOVA runs use one-sided testing with 999 permutations (p=0.001 is the minimum attainable)
- PCoA ordinations from both distance matrices export as valid interactive Emperor/EMPress .qzv visualizations

## Limitations

- Only chemical features with MS/MS data are considered; MS1-only features without fingerprints are filtered out of the feature table
- Typically only 1–20% of features have spectral library matches, and fingerprint prediction depends on the quality/coverage of MS/MS spectral databases
- Different tree-building approaches produce different tree topologies; a broader benchmarking effort is needed to know when each should be used

## Evidence

- [readme] Qemistree trees of metabolomics data viewable in Empress: [evidence span withheld — anchor status: external_doc]
