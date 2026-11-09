---
name: permanova-group-significance-testing
description: Use when a sample metadata column (e.g., chromatographic batch) must
  be tested against beta diversity of a metabolomics feature table, especially to
  quantify how much a Qemistree chemical hierarchy attenuates batch-driven separation.
when_to_use_negative:
- No chemical hierarchy is available (features lack MS/MS data or SIRUS/CSI:FingerID
  fingerprints), so a tree-informed UniFrac metric cannot be computed and only tree-agnostic
  distances apply.
- The metadata column has fewer than two groups or highly unbalanced group sizes,
  making PERMANOVA pseudo-F interpretation unreliable.
- The goal is within-sample (alpha) diversity such as Faith's Phylogenetic Diversity
  rather than between-sample group separation.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QIIME 2
  role: Runs beta diversity, beta-phylogenetic (weighted UniFrac), PERMANOVA (beta-group-significance),
    and PCoA commands; Qemistree outputs plug into these workflows
  repo: https://github.com/biocore/q2-qemistree
- name: Qemistree
  role: Constructs the chemical hierarchy from SIRIUS-predicted molecular fingerprints
    that serves as the phylogeny for tree-informed UniFrac
  repo: https://github.com/biocore/q2-qemistree
- name: Empress
  role: Visualizes the Qemistree tree and integrates with PCoA ordination plots (community-plot)
    for exploring PERMANOVA-supported group separation
  repo: https://github.com/biocore/empress
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:22eafdfd0b8780dba14bf9cbc6404494225f9963ff828ce188b9c5a56dcc6042
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
  - Empress
  - Python
  - QIIME 2
  - Qemistree
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# permanova-group-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether sample grouping (e.g., chromatographic batch) explains significant variation in beta-diversity by running one-sided PERMANOVA (999 permutations) on distance matrices, comparing tree-agnostic (Bray-Curtis) vs tree-informed (weighted UniFrac on a Qemistree chemical hierarchy) metrics.

## When to use

Use when a sample metadata column (e.g., chromatographic batch) must be tested against beta diversity of a metabolomics feature table, especially to quantify how much a Qemistree chemical hierarchy attenuates batch-driven separation.

## When NOT to use

- No chemical hierarchy is available (features lack MS/MS data or SIRUS/CSI:FingerID fingerprints), so a tree-informed UniFrac metric cannot be computed and only tree-agnostic distances apply.
- The metadata column has fewer than two groups or highly unbalanced group sizes, making PERMANOVA pseudo-F interpretation unreliable.
- The goal is within-sample (alpha) diversity such as Faith's Phylogenetic Diversity rather than between-sample group separation.

## Inputs

- FeatureTable[Frequency] (.qza) — e.g., evaluation dataset quant table from MassIVE MSV000083306 (162 samples)
- Qemistree chemical hierarchy as rooted phylogeny (Phylogeny[Rooted] .qza)
- Sample metadata TSV containing a batch column assigning each sample to one of the two chromatographic batches

## Outputs

- Bray-Curtis distance matrix (.qza)
- Weighted UniFrac distance matrix computed on the chemical hierarchy (.qza)
- PERMANOVA group-significance visualizations (.qzv) with pseudo-F and p-values
- PCoA ordinations (.qza) and interactive Emperor/Empress ordination plots (.qzv)

## How to apply

Import the feature table (e.g., the 162-sample evaluation dataset quant table from MassIVE accession MSV000083306) as FeatureTable[Frequency] and the Qemistree chemical hierarchy as a rooted phylogeny. Compute a tree-agnostic Bray-Curtis matrix with 'qiime diversity beta' and a tree-informed weighted UniFrac matrix with 'qiime diversity beta-phylogenetic' using the Qemistree tree as phylogeny input. Group samples by the metadata batch column and run one-sided PERMANOVA with 999 permutations via 'qiime diversity beta-group-significance' (permanova method) on each matrix. Generate PCoA ordinations from each distance matrix with 'qiime diversity pcoa' and export interactive ordination plots as Emperor/Empress .qzv artifacts. The rationale is that fingerprint-based representation is largely independent of technical variation such as chromatography shifts, so the tree-informed pseudo-F should be substantially lower than the tree-agnostic pseudo-F if the hierarchy mitigates batch signal.

## Related tools

- **QIIME 2** (Runs beta diversity, beta-phylogenetic (weighted UniFrac), PERMANOVA (beta-group-significance), and PCoA commands; Qemistree outputs plug into these workflows) — https://github.com/biocore/q2-qemistree
- **Qemistree** (Constructs the chemical hierarchy from SIRIUS-predicted molecular fingerprints that serves as the phylogeny for tree-informed UniFrac) — https://github.com/biocore/q2-qemistree
- **Empress** (Visualizes the Qemistree tree and integrates with PCoA ordination plots (community-plot) for exploring PERMANOVA-supported group separation) — https://github.com/biocore/empress

## Examples

```
qiime diversity beta-group-significance --i-distance-matrix unifrac_distance_matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column batch --p-method permanova --o-visualization permanova-unifrac.qzv
```

## Evaluation signals

- Recover pseudo-F = 120.75 (p = 0.001) for the tree-agnostic Bray-Curtis metric and pseudo-F = 18.2239 (p = 0.001) for the tree-informed weighted UniFrac metric on the 162-sample evaluation dataset with 999 permutations.
- Confirm the tree-informed pseudo-F is substantially lower than the tree-agnostic pseudo-F, consistent with weighted UniFrac mitigating the chromatographic batch effect.
- Verify p = 0.001 is the minimum attainable value given 999 permutations, so the p-value granularity matches the permutation count.
- Check both distance matrices are square, symmetric, with zero diagonals and contain all 162 evaluation samples.
- Confirm the PCoA .qzv plots (Emperor/Empress) render batch as a colorable metadata category separating the two chromatographic batches in the Bray-Curtis ordination.

## Limitations

- Only 1–20% of features typically have spectral library matches and fingerprint prediction depends on the quality and coverage of MS/MS spectral databases, so the hierarchy (and hence UniFrac distances) may be incomplete.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees and weaker tree-informed distances.
- With 999 permutations, the smallest reportable p-value is 0.001; results depend on the tree-building approach, since different tree topologies capture complementary chemical information.

## Evidence
