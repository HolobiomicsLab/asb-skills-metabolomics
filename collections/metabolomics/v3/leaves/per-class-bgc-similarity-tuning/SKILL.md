---
name: per-class-bgc-similarity-tuning
description: Use when tuning BiG-SCAPE pairwise distance weights per BGC class, e.g.
  when JI/AI/DSS combinations must be optimized against MIBiG compound structural
  distances for classes like RiPPs or PKS/NRP hybrids.
when_to_use_negative:
- When the goal is simply assigning GCFs at the empirical network cutoff distance
  of 0.3 with default weights — this skill calibrates weights, it does not cluster
  the network
- When no compound structural reference (e.g. MIBiG SMILES) is available for the BGC
  classes of interest, since weight selection is driven by correlation with Tanimoto
  distances
tools:
- name: BiG-SCAPE
  role: Source of the per-class pairwise distance components (Jaccard Index, Domain
    Sequence Similarity, Adjacency Index) whose weights are optimized
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: RDKit
  role: Computes Morgan fingerprints (radius=4) and Tanimoto-based compound structural
    distances from MIBiG SMILES to serve as the correlation target
- name: antiSMASH
  role: Produces the GenBank BGC predictions that feed BiG-SCAPE's domain-based distance
    calculation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:e0916acb4349147aedf10e31a51b5d8636af450bc97e6188a04600c6a8892a0b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - RDKit
  - antiSMASH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# per-class-bgc-similarity-tuning

## Summary

Calibrates class-specific weight combinations for BiG-SCAPE's pairwise distance indices (Jaccard Index, Domain Sequence Similarity, and Adjacency Index) by maximizing the correlation between weighted BGC distances and MIBiG compound structural (Tanimoto) distances. This grounds the similarity metric in biosynthetic chemistry rather than arbitrary defaults.

## When to use

Use when tuning BiG-SCAPE pairwise distance weights per BGC class, e.g. when JI/AI/DSS combinations must be optimized against MIBiG compound structural distances for classes like RiPPs or PKS/NRP hybrids.

## When NOT to use

- When the goal is simply assigning GCFs at the empirical network cutoff distance of 0.3 with default weights — this skill calibrates weights, it does not cluster the network
- When no compound structural reference (e.g. MIBiG SMILES) is available for the BGC classes of interest, since weight selection is driven by correlation with Tanimoto distances

## Inputs

- MIBiG v1.3 compound SMILES set (1,393 reference BGC entries, August 2016 release)
- BiG-SCAPE pairwise distance components per BGC class (JI, DSS, Goodman-Kruskal, Adjacency Index values)
- antiSMASH v4 (--minimal) GenBank outputs and MIBiG reference GenBank files processed with Pfam version 31 via HMMER 3.1b2

## Outputs

- optimized_class_weights.tsv containing one optimized weight row per BiG-SCAPE class
- Per-class correlation values of weighted BGC distance vs. MIBiG compound Tanimoto distance (AI vs. GK comparison)

## How to apply

First compute all-pairs compound structural distances on the MIBiG v1.3 SMILES set with RDKit Morgan fingerprints (radius=4) scored by Tanimoto coefficient. Then load the BiG-SCAPE pairwise distance components per class — Jaccard Index of shared domain types, Domain Sequence Similarity (DSS), Goodman-Kruskal (GK), and Adjacency Index (AI) values. Brute-force enumerate weight combinations per BiG-SCAPE class, stepping the JI/DSS/GK/AI weights in increments of 0.01. For each candidate combination, compute the weighted BGC distance and correlate it with the MIBiG compound Tanimoto distance matrix, then retain the per-class weight set that maximizes agreement. Validate that the Adjacency Index weighting outperforms the Goodman-Kruskal metric under the optimized weights; the rationale is that BiG-SCAPE explicitly 'accounts for differences in modes of evolution between BGC classes'.

## Related tools

- **BiG-SCAPE** (Source of the per-class pairwise distance components (Jaccard Index, Domain Sequence Similarity, Adjacency Index) whose weights are optimized) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **RDKit** (Computes Morgan fingerprints (radius=4) and Tanimoto-based compound structural distances from MIBiG SMILES to serve as the correlation target)
- **antiSMASH** (Produces the GenBank BGC predictions that feed BiG-SCAPE's domain-based distance calculation)

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fp1, fp2 = AllChem.GetMorganFingerprint(Chem.MolFromSmiles(s1), 4), AllChem.GetMorganFingerprint(Chem.MolFromSmiles(s2), 4); sim = DataStructs.TanimotoSimilarity(fp1, fp2)
```

## Evaluation signals

- optimized_class_weights.tsv contains exactly one optimized weight row per BiG-SCAPE class (eight classes expected)
- For every class, the correlation of the Adjacency Index with compound Tanimoto distances exceeds that of the Goodman-Kruskal metric under the optimized weights
- Correlations are computed against an RDKit Morgan (radius=4) Tanimoto distance matrix derived from MIBiG v1.3 SMILES, not arbitrary similarity targets
- Enumerated weights cover the JI/DSS/GK/AI grid at 0.01 increments, so no selected weight deviates from this grid resolution

## Limitations

- Weights are calibrated on MIBiG release 1.3 (1,393 clusters, August 2016); they may not transfer to newer MIBiG releases or BGC classes absent from the reference set
- Brute-force enumeration over per-class weight grids is computationally expensive as class size grows
- BiG-SCAPE 2.0 introduced several updates to input validation and reference usage, so the version-1.x calibrated weights and distance semantics may not apply directly

## Evidence

- [readme] calculating pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity.: [evidence span withheld — anchor status: external_doc]
