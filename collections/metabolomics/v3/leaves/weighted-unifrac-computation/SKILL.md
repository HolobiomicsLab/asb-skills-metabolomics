---
name: weighted-unifrac-computation
description: Use when you have a Qemistree chemical tree plus a feature table of MS1
  peak areas and need tree-informed sample comparisons, e.g., to quantify how chemically
  similar foods or samples are.
when_to_use_negative:
- Input is raw LC-MS/MS spectra (.mzXML/.mzML) with no Qemistree hierarchy yet — build
  the tree first with q2-qemistree and MZmine2 preprocessing
- Only MS1 features without fingerprints are present in the feature table — these
  are filtered out and cannot be placed on the chemical hierarchy
- The analysis goal is molecular annotation (spectral library matching, ClassyFire
  taxonomy) rather than sample-to-sample comparison — use annotation workflows instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: QIIME 2
  role: Provides the diversity plugin used to compute Weighted UniFrac on the Qemistree
    chemical hierarchy; Qemistree outputs are explicitly designed to plug into QIIME
    2 workflows
- name: Qemistree
  role: Builds the chemical tree of LC-MS/MS features from predicted molecular fingerprints
    that serves as the phylogeny input for UniFrac computation
  repo: github.com/biocore/q2-qemistree
- name: Empress
  role: Visualizes the Qemistree tree with sample metadata coloring so tips/features
    traced by the UniFrac analysis can be inspected interactively
  repo: github.com/biocore/empress
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:fabeb051d0c4c001c41ea9654e02eace44aa35c5a67971d09ae2234f70cefb85
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  tools:
  - Empress
  - QIIME 2
  - Qemistree
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# weighted-unifrac-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Weighted UniFrac sample-to-sample distances using the QIIME 2 diversity plugin on a Qemistree chemical hierarchy, adapting a phylogenetic beta-diversity metric designed for DNA sequence data to untargeted metabolomics feature tables.

## When to use

Use when you have a Qemistree chemical tree plus a feature table of MS1 peak areas and need tree-informed sample comparisons, e.g., to quantify how chemically similar foods or samples are.

## When NOT to use

- Input is raw LC-MS/MS spectra (.mzXML/.mzML) with no Qemistree hierarchy yet — build the tree first with q2-qemistree and MZmine2 preprocessing
- Only MS1 features without fingerprints are present in the feature table — these are filtered out and cannot be placed on the chemical hierarchy
- The analysis goal is molecular annotation (spectral library matching, ClassyFire taxonomy) rather than sample-to-sample comparison — use annotation workflows instead

## Inputs

- Qemistree chemical hierarchy (Newick tree of MS1 features built from predicted molecular fingerprints)
- Feature table of MS1 peak areas (.biom), with MS1 features lacking fingerprints filtered out
- Sample metadata (TSV) describing sample source (e.g., food type)

## Outputs

- Weighted UniFrac sample-by-sample distance matrix (QIIME 2 .qza / distance matrix)
- Ordination of samples in chemical-composition space
- CSV table of tree-informed sample comparisons / presence-absence checks of ingredient signatures

## How to apply

Build the chemical hierarchy first with q2-qemistree (compute-fragmentation-trees, predict-fingerprints, make-hierarchy), then feed the resulting tree plus the feature table (e.g., a .biom quant table) into the QIIME 2 diversity plugin to compute Weighted UniFrac between samples. Weighted UniFrac accounts for both chemical relatedness (branch structure of the fingerprint-based tree) and feature abundance (peak areas), so it captures chemically informed relationships between samples rather than presence/absence alone. Rationale: Qemistree explicitly repurposes ecological tools built for DNA sequencing analysis, so tree-based metrics like UniFrac and Faith's Phylogenetic Diversity become applicable to mass spectrometry data. After computing the distance matrix, ordinate it and visualize tip-level feature distributions in Empress colored by sample metadata (e.g., food source) to trace ingredient-derived signatures such as phosphoethanolamine in an egg scramble. Save the resulting per-sample comparisons as a CSV table for downstream checks.

## Related tools

- **QIIME 2** (Provides the diversity plugin used to compute Weighted UniFrac on the Qemistree chemical hierarchy; Qemistree outputs are explicitly designed to plug into QIIME 2 workflows)
- **Qemistree** (Builds the chemical tree of LC-MS/MS features from predicted molecular fingerprints that serves as the phylogeny input for UniFrac computation) — github.com/biocore/q2-qemistree
- **Empress** (Visualizes the Qemistree tree with sample metadata coloring so tips/features traced by the UniFrac analysis can be inspected interactively) — github.com/biocore/empress

## Examples

```
qiime diversity beta --i-table feature-table.biom --i-phylogeny qemistree.nwk --p-metric weighted_unifrac --o-distance-matrix weighted_unifrac.qza
```

## Evaluation signals

- The distance matrix is square, symmetric, with zero diagonal — a sanity invariant of beta-diversity output
- UniFrac distances between biological replicates of the same food (e.g., egg scramble replicates) should be lower than between different foods
- Feature counts entering the calculation must equal the pruned hierarchy tips: features lacking fingerprints must have been removed beforehand
- Presence/absence of traced signatures (e.g., phosphoethanolamine in egg scramble, black-pepper signal absent in blueberry kefir) should be consistent with the distance-matrix grouping
- Ordination of the resulting matrix should reproduce the chemically informed sample grouping reported for the foodomics dataset (Fig. 2 context)

## Limitations

- Typically only 1–20% of features have spectral library matches, and fingerprint prediction depends on the coverage of MS/MS spectral databases, so the tree underpinning UniFrac is partial
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted, resulting in less-extensive trees and thus less-informed distance calculations
- The tree hierarchy is a representation, not ground truth: different tree-building approaches produce different topologies, which can change UniFrac distances

## Evidence

- [readme] we can apply ecological tools, designed around the relatedness of DNA sequences, to study chemical composition: [evidence span withheld — anchor status: external_doc]
