---
name: bgc-distance-metric-weight-optimization
description: Use when BiG-SCAPE distance weights for JI, AI, and DSS must be calibrated
  per BGC class against known compound structures (MIBiG) so that BGC similarity networks
  reflect chemical similarity.
when_to_use_negative:
- When the BGCs of interest lack compound reference structures in MIBiG — the weight
  calibration is anchored to BGC–compound distance correlation and cannot be optimized
  without them
- When the goal is GCF assignment/clustering itself (e.g. filtering the network at
  BiG-SCAPE's empirical cutoff of 0.3 or 0.75) rather than calibrating the underlying
  distance metric weights
- When analyzing a single unclassified BGC set, since the method produces class-specific
  weight sets, not a single global weighting
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SCAPE
  role: Supplies the pairwise distance components (Jaccard Index, Domain Sequence
    Similarity, Goodman-Kruskal, Adjacency Index) per BGC class that are re-weighted
    and correlated with compound distances
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: RDKit
  role: Computes all-pairs structural Tanimoto distances between MIBiG compounds from
    Morgan fingerprints (radius=4) as the chemical ground truth for weight optimization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:4c3d34cc651bf3e08a85309ee04d8dfd4f70819491137c325e616074889eebd2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# bgc-distance-metric-weight-optimization

## Summary

Reconstructs BiG-SCAPE's calibration of class-specific weights for its Jaccard Index (JI), Adjacency Index (AI), and Domain Sequence Similarity (DSS) distance components by maximizing the correlation between weighted BGC distances and MIBiG compound structural (Tanimoto) distances for each of the eight BiG-SCAPE classes.

## When to use

Use when BiG-SCAPE distance weights for JI, AI, and DSS must be calibrated per BGC class against known compound structures (MIBiG) so that BGC similarity networks best reflect chemical similarity.

## When NOT to use

- When the BGCs of interest lack compound reference structures in MIBiG — the weight calibration is anchored to BGC–compound distance correlation and cannot be optimized without them
- When the goal is GCF assignment/clustering itself (e.g. filtering the network at BiG-SCAPE's empirical cutoff of 0.3 or 0.75) rather than calibrating the underlying distance metric weights
- When analyzing a single unclassified BGC set, since the method produces class-specific weight sets, not a single global weighting

## Inputs

- MIBiG v1.3 compound SMILES strings (structural reference data)
- BiG-SCAPE pairwise distance components per BGC class: Jaccard Index of shared domain types, Domain Sequence Similarity, Goodman-Kruskal, and Adjacency Index values
- BiG-SCAPE class assignments for the eight BGC classes (e.g. from antiSMASH-predicted BGCs incl. MIBiG reference clusters)

## Outputs

- optimized_class_weights.tsv with one optimized weight row per BiG-SCAPE class
- Per-class correlation values between weighted BGC distances and compound Tanimoto distances
- Per-class comparison showing Adjacency Index correlation exceeds Goodman-Kruskal correlation under optimized weights

## How to apply

Parse the MIBiG v1.3 compound SMILES set and compute all-pairs structural distances with RDKit using Morgan fingerprints (radius=4) scored by Tanimoto coefficient. Load the BiG-SCAPE pairwise distance components for each of the eight BGC classes — the Jaccard Index of shared domain types, Domain Sequence Similarity, Goodman-Kruskal (GK), and Adjacency Index (AI) values. Brute-force enumerate weight combinations per class, stepping the JI/DSS/GK/AI weights in increments of 0.01 and the anchorboost weights 1–4 in steps of 0.5. For each candidate combination, compute the weighted BGC distance and correlate it with the MIBiG compound Tanimoto distance matrix, then select the weight set per class that maximizes agreement with compound structural distances. Finally, confirm that under the optimized weights the Adjacency Index correlates better with compound Tanimoto distances than the Goodman-Kruskal metric for every class.

## Related tools

- **BiG-SCAPE** (Supplies the pairwise distance components (Jaccard Index, Domain Sequence Similarity, Goodman-Kruskal, Adjacency Index) per BGC class that are re-weighted and correlated with compound distances) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **RDKit** (Computes all-pairs structural Tanimoto distances between MIBiG compounds from Morgan fingerprints (radius=4) as the chemical ground truth for weight optimization)

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fp = AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smiles), 4); sim = DataStructs.TanimotoSimilarity(fp1, fp2)
```

## Evaluation signals

- optimized_class_weights.tsv contains exactly one optimized weight row per BiG-SCAPE class (eight classes)
- For every class, the Adjacency Index correlation with compound Tanimoto distances exceeds the Goodman-Kruskal correlation under the optimized weights
- Selected weights correspond to the maximum BGC–compound distance correlation among the brute-forced combinations (JI/DSS/GK/AI stepped at 0.01; anchorboost weights 1–4 at 0.5 steps)
- Compound distance matrix matches RDKit Morgan radius=4 Tanimoto distances over the MIBiG v1.3 SMILES set (all-pairs)
- Weighted BGC distances are recomputed per class, not with a single shared weight vector

## Limitations

- Calibration is anchored to MIBiG reference data (release 1.3, August 2016); classes poorly represented in MIBiG cannot be reliably weight-optimized
- Brute-force enumeration at 0.01 step size over four weight dimensions is combinatorially expensive
- The optimization assumes the class partition (eight BiG-SCAPE classes) reflects genuinely different modes of evolution, so weights are not transferable across class assignments
- Optimized weights are validated by correlation with Tanimoto distance, which is a proxy — agreement between BGC distance and chemical structure does not guarantee biosynthetic correctness

## Evidence

- [discussion] Pairwise distance calculation is divided between three values that measure a) the percentage of shared domain types (Jaccard Index): "Pairwise distance calculation is divided between three values that measure a) the percentage of shared domain types (Jaccard Index)"
- [readme] calculating pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity.: [evidence span withheld — anchor status: external_doc]
- [intro] strengthened by comparative analysis of identified BGCs with biochemical reference data, such as those provided by the MIBiG community effort21: "strengthened by comparative analysis of identified BGCs with biochemical reference data, such as those provided by the MIBiG community effort21"
- [discussion] 1,393 more were added from the Minimum Information about a Biosynthetic Gene Cluster database (MIBiG21, release 1.3, August 2016: "1,393 more were added from the Minimum Information about a Biosynthetic Gene Cluster database (MIBiG21, release 1.3, August 2016"
