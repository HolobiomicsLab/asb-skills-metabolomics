---
name: pcoa-ordination-of-metabolomics-distance-matrices
description: Use when you have a beta diversity or phylogenetic distance matrix computed
  from an LC-MS/MS metabolomics feature table and need 2D/3D ordination plots to compare
  clustering by sample metadata (e.g. chromatographic batch).
when_to_use_negative:
- When the input is already an ordination (e.g. an Emperor-compatible .qzv or scikit-bio
  OrdinationResults file) — skip to visualization rather than re-running pcoa.
- When the analysis goal is a statistical significance test rather than visual ordination
  — PERMANOVA (qiime diversity beta-group-significance) is the appropriate tool, though
  it can be run alongside PCoA.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QIIME 2
  role: Hosts the qiime diversity pcoa command that projects each distance matrix
    into principal coordinates, and the qiime diversity beta/beta-phylogenetic commands
    that produce the distance matrices being ordinated.
  repo: https://github.com/biocore/q2-qemistree
- name: Qemistree
  role: Supplies the chemical hierarchy (Phylogeny[Rooted]) that makes the distance
    matrix tree-informed (weighted UniFrac), reducing chromatography-driven batch
    signal before ordination.
  repo: https://github.com/biocore/q2-qemistree
- name: EMPress
  role: Visualizes the chemical hierarchy tree alongside the PCoA ordination so molecular
    relationships and sample clustering can be explored together.
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:ea889c6db4daad8e9e154ee480b27051f18da84832f6ef67fc39adee6f9ed38a
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pcoa-ordination-of-metabolomics-distance-matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate PCoA ordinations from metabolomics distance matrices (e.g. weighted UniFrac on a Qemistree chemical hierarchy or Bray-Curtis on the raw feature table) and export interactive Emperor/EMPress visualizations so batch effects and chemically informed structure can be judged visually.

## When to use

Use when you have a beta diversity or phylogenetic distance matrix computed from an LC-MS/MS metabolomics feature table and need 2D/3D ordination plots to compare clustering by sample metadata (e.g. chromatographic batch).

## When NOT to use

- When the input is already an ordination (e.g. an Emperor-compatible .qzv or scikit-bio OrdinationResults file) — skip to visualization rather than re-running pcoa.
- When the analysis goal is a statistical significance test rather than visual ordination — PERMANOVA (qiime diversity beta-group-significance) is the appropriate tool, though it can be run alongside PCoA.

## Inputs

- QIIME 2 distance matrix (DistanceMatrix artifact) from Bray-Curtis or weighted UniFrac on an LC-MS/MS metabolomics feature table
- FeatureTable[Frequency] artifact (quant table, e.g. MassIVE MSV000083306 evaluation dataset)
- Sample metadata TSV with a chromatographic batch column
- Qemistree chemical hierarchy (Phylogeny[Rooted]) for tree-informed metrics

## Outputs

- PCoA ordination artifact (PCoAResults) per distance metric
- Interactive Emperor/EMPress ordination .qzv visualization
- Optional paired EMPress tree+ordination plot showing chemical hierarchy alongside samples

## How to apply

Compute a distance matrix first — in this workflow the tree-agnostic Bray-Curtis matrix from the FeatureTable[Frequency] and the tree-informed weighted UniFrac matrix computed with the Qemistree chemical hierarchy (Phylogeny[Rooted]) as the phylogeny input. Run PCoA on each matrix with qiime diversity pcoa, then export interactive ordination plots as .qzv visualizations using Emperor (or EMPress, which synchronizes the tree with the ordination). Ordinations are generated per metric so the batch effect (pseudo-F 120.75 for Bray-Curtis vs 18.2239 for weighted UniFrac) can be seen as reduced separation between chromatographic batches. Validate by confirming the ordination of the tree-informed matrix shows substantially less separation of the two chromatographic batches than the tree-agnostic one, and that PERMANOVA pseudo-F values match the reported 120.75 and 18.2239 (p=0.001, 999 permutations).

## Related tools

- **QIIME 2** (Hosts the qiime diversity pcoa command that projects each distance matrix into principal coordinates, and the qiime diversity beta/beta-phylogenetic commands that produce the distance matrices being ordinated.) — https://github.com/biocore/q2-qemistree
- **Qemistree** (Supplies the chemical hierarchy (Phylogeny[Rooted]) that makes the distance matrix tree-informed (weighted UniFrac), reducing chromatography-driven batch signal before ordination.) — https://github.com/biocore/q2-qemistree
- **EMPress** (Visualizes the chemical hierarchy tree alongside the PCoA ordination so molecular relationships and sample clustering can be explored together.) — https://github.com/biocore/empress

## Examples

```
qiime diversity pcoa --i-distance-matrix weighted_unifrac_distance_matrix.qza --o-pcoa-results pcoa_results.qza && qiime emperor plot --i-pcoa pcoa_results.qza --m-metadata-file sample_metadata.tsv --o-visualization emperor_plot.qzv
```

## Evaluation signals

- The PCoA of the tree-informed (weighted UniFrac) matrix shows visibly reduced separation between the two chromatographic batches compared to the Bray-Curtis PCoA.
- One-sided PERMANOVA on the underlying matrices recovers pseudo-F=120.75 (tree-agnostic Bray-Curtis) and pseudo-F=18.2239 (tree-informed weighted UniFrac), both with p=0.001 at 999 permutations.
- The ordination .qzv is openable on view.qiime2.org and contains sample points colored by the metadata batch column.
- For EMPress output, an empress.html (or qzv) renders the Qemistree hierarchy with synchronized sample points from the ordination.

## Limitations

- PCoA captures only the variance structure of the distance matrix; the observed reduction in batch signal (pseudo-F 120.75 → 18.2239) depends on the tree-informed metric, not on the ordination itself.
- Fingerprint prediction underlying the Qemistree hierarchy depends on the quality and coverage of MS/MS spectral databases, so tree-informed ordinations are less extensive in negative ionization mode where fewer reference spectra are available.
- Different tree-building approaches produce different tree topologies, so ordinations built on alternative hierarchies will capture complementary chemical information and may differ in how strongly they suppress batch effects.

## Evidence

- [other] PCoA from distance matrices: [evidence span withheld — anchor status: not_found]
- [other] Tree-informed vs tree-agnostic comparison: [evidence span withheld — anchor status: not_found]
- [readme] README: Empress ordination integration: [evidence span withheld — anchor status: external_doc]
- [readme] README: trees of metabolomics data generated using Qemistree: [evidence span withheld — anchor status: external_doc]
