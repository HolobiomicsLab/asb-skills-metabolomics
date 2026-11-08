---
name: molecular-fingerprint-tanimoto-similarity
description: Use when BGC- or pathway-based distance metrics must be validated or
  weight-optimized against a ground-truth chemical-structure distance matrix derived
  from reference compound SMILES (e.g., MIBiG).
when_to_use_negative:
- When similarity must be computed from MS/MS fragmentation spectra rather than structures
  — the article's molecular-family side uses fragmentation cosine similarity (> 0.75)
  and 4 ppm mass matching, not Tanimoto fingerprints
- When the goal is BGC clustering alone with no reference compound data, since the
  correlation-to-compound step requires MIBiG (or equivalent) SMILES input
edam_topics:
- http://edamontology.org/topic_2250
tools:
- name: RDKit
  role: Generates Morgan fingerprints (radius 4) from MIBiG compound SMILES and computes
    Tanimoto coefficients for the all-pairs compound distance matrix
- name: BiG-SCAPE
  role: Produces the pairwise BGC distance components (JI, DSS, GK, AI) whose per-class
    weights are optimized against the compound Tanimoto distances
  repo: https://github.com/medema-group/BiG-SCAPE
- name: MIBiG
  role: Supplies the reference biosynthetic gene clusters and compound structures
    (release 1.3) used as biochemical ground truth
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:717e59628b05617377bb52c3793cfc2dd45b55816e97beba750b084e4e70fb91
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/medema-group/BiG-SCAPE
  tools:
  - BiG-SCAPE
  - CFM-ID
  - FIORA
  - ICEBERG
  - MIBiG
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- molecular-fingerprint-tanimoto-similarity-computation
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# molecular-fingerprint-tanimoto-similarity

## Summary

Computes all-pairs structural similarity of small molecules (e.g., MIBiG v1.3 compound SMILES) using RDKit Morgan fingerprints at radius 4 scored by the Tanimoto coefficient, yielding a compound distance matrix used to calibrate BiG-SCAPE's per-class BGC distance weights (JI, AI, DSS).

## When to use

Use when BGC- or pathway-based distance metrics must be validated or weight-optimized against a ground-truth chemical-structure distance matrix derived from reference compound SMILES (e.g., MIBiG).

## When NOT to use

- When similarity must be computed from MS/MS fragmentation spectra rather than structures — the article's molecular-family side uses fragmentation cosine similarity (> 0.75) and 4 ppm mass matching, not Tanimoto fingerprints
- When the goal is BGC clustering alone with no reference compound data, since the correlation-to-compound step requires MIBiG (or equivalent) SMILES input

## Inputs

- MIBiG v1.3 compound SMILES set (reference biosynthetic compounds)
- BiG-SCAPE pairwise distance components per BGC class (Jaccard Index, Domain Sequence Similarity, Goodman-Kruskal, Adjacency Index)

## Outputs

- All-pairs compound Tanimoto distance matrix
- Per-class optimized weight table (optimized_class_weights.tsv) maximizing BGC-vs-compound distance correlation

## How to apply

Parse the reference compound SMILES set (here MIBiG v1.3) and compute all-pairs structural distances in RDKit by generating Morgan fingerprints at radius=4 and scoring pairs by Tanimoto coefficient. Use this compound distance matrix as the target when brute-force enumerating weight combinations for BiG-SCAPE's distance components (Jaccard Index of shared domain types, Domain Sequence Similarity, Goodman-Kruskal, Adjacency Index), stepping the JI/DSS/GK/AI weights in increments of 0.01 and the anchor weights in steps of 0.5. For each candidate weight vector, compute the weighted BGC distance and correlate it with the compound Tanimoto distance matrix, then select per BiG-SCAPE class the weight set that maximizes agreement. This grounds the per-class calibration in actual biosynthetic-product chemistry, since BGCs of the same class encode structurally related compounds.

## Related tools

- **RDKit** (Generates Morgan fingerprints (radius 4) from MIBiG compound SMILES and computes Tanimoto coefficients for the all-pairs compound distance matrix)
- **BiG-SCAPE** (Produces the pairwise BGC distance components (JI, DSS, GK, AI) whose per-class weights are optimized against the compound Tanimoto distances) — https://github.com/medema-group/BiG-SCAPE
- **MIBiG** (Supplies the reference biosynthetic gene clusters and compound structures (release 1.3) used as biochemical ground truth)

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fp1, fp2 = [AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(s), 4) for s in (smi1, smi2)]; d = 1 - DataStructs.TanimotoSimilarity(fp1, fp2)
```

## Evaluation signals

- optimized_class_weights.tsv contains exactly one optimized weight row per BiG-SCAPE class (eight classes)
- For every class, the Adjacency Index correlation with compound Tanimoto distances exceeds the Goodman-Kruskal metric correlation under the optimized weights
- Weight enumeration covers the full grid: JI/DSS/GK/AI in 0.01 increments, anchor weights 1-4 in 0.5 steps
- The winning per-class weight set is the one maximizing correlation between weighted BGC distance and the MIBiG compound Tanimoto distance matrix
- All-pairs Tanimoto matrix dimensions match the parsed MIBiG SMILES set (release 1.3, 1,393 added entries)

## Limitations

- Calibration is only as good as the MIBiG reference set; compound structures must be computable from SMILES via RDKit
- Correlation-based weight selection assumes compound Tanimoto distance is a meaningful proxy for BGC divergence within a class; classes lacking sufficient reference pairs cannot be calibrated reliably
- BiG-SCAPE's interactive reporting can silently drop genomes with duplicate ORGANISM identifiers (only 96 of 103 Streptomyces genomes reported in the article), affecting downstream pairwise inputs

## Evidence

- [readme] calculating pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity: [evidence span withheld — anchor status: external_doc]
