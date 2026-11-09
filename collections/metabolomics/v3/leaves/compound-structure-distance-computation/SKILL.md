---
name: compound-structure-distance-computation
description: Use when you need a compound-vs-compound structural distance matrix from
  a reference set (e.g. MIBiG release 1.3) to benchmark or weight-cluster BGC distances
  such as BiG-SCAPE's Jaccard, Adjacency Index and Domain Sequence Similarity components.
when_to_use_negative:
- Reference compounds lack parseable SMILES structures, so no fingerprint-based Tanimoto
  comparison is possible
- The goal is BGC-vs-BGC similarity (use BiG-SCAPE's Jaccard/Adjacency/Domain Sequence
  Similarity indices directly) rather than chemical ground truth for weight calibration
- No annotated reference compound set such as MIBiG is available for the BGCs under
  study
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RDKit
  role: Parses MIBiG compound SMILES, computes Morgan fingerprints (radius=4) and
    Tanimoto coefficients for the all-pairs compound distance matrix
- name: BiG-SCAPE
  role: Supplies the pairwise BGC distance components (JI, AI, DSS, GK) that are correlated
    against the compound Tanimoto distance matrix during per-class weight calibration
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:3529c2d130f030ecee4649c7530a4242911e822c3ff03d77093de508f14fff62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# compound-structure-distance-computation

## Summary

Computes all-pairs structural distance matrices for reference compounds (e.g. MIBiG v1.3 SMILES) using RDKit Morgan fingerprints (radius=4) scored by Tanimoto coefficient, providing the chemical ground truth needed to calibrate BiG-SCAPE's class-specific distance-index weights (JI, AI, DSS).

## When to use

Use when you need a compound-vs-compound structural distance matrix from a reference set (e.g. MIBiG release 1.3) to benchmark or weight-cluster BGC distances such as BiG-SCAPE's Jaccard, Adjacency Index and Domain Sequence Similarity components.

## When NOT to use

- Reference compounds lack parseable SMILES structures, so no fingerprint-based Tanimoto comparison is possible
- The goal is BGC-vs-BGC similarity (use BiG-SCAPE's Jaccard/Adjacency/Domain Sequence Similarity indices directly) rather than chemical ground truth for weight calibration
- No annotated reference compound set such as MIBiG is available for the BGCs under study

## Inputs

- MIBiG v1.3 compound SMILES strings (1,393 reference BGCs in release 1.3, August 2016)
- BiG-SCAPE pairwise distance components per BGC class (Jaccard Index of shared domain types, Domain Sequence Similarity, Goodman-Kruskal, Adjacency Index)

## Outputs

- All-pairs compound structural Tanimoto distance matrix (values in [0,1])
- optimized_class_weights.tsv with one optimized weight row per BiG-SCAPE class (used downstream to validate the weighting)

## How to apply

Parse the MIBiG v1.3 compound SMILES set and convert each entry to an RDKit molecule, then compute Morgan fingerprints with radius=4 for every compound. Build the all-pairs distance matrix as 1 minus the Tanimoto coefficient between fingerprint pairs, so distances lie in [0,1]. Use this compound Tanimoto distance matrix as the target when brute-force enumerating BiG-SCAPE weight combinations per BGC class (JI/DSS/GK/AI in 0.01 increments, anchor-boost weights in 0.5 steps), selecting per class the weight set that maximizes correlation between BGC distances and compound distances. The same matrix also serves as the validation reference: under the optimized per-class weights, the Adjacency Index must correlate better with compound Tanimoto distances than the Goodman-Kruskal metric.

## Related tools

- **RDKit** (Parses MIBiG compound SMILES, computes Morgan fingerprints (radius=4) and Tanimoto coefficients for the all-pairs compound distance matrix)
- **BiG-SCAPE** (Supplies the pairwise BGC distance components (JI, AI, DSS, GK) that are correlated against the compound Tanimoto distance matrix during per-class weight calibration) — https://git.wageningenur.nl/medema-group/BiG-SCAPE

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; mols=[Chem.MolFromSmiles(s) for s in smiles]; fps=[AllChem.GetMorganFingerprint(m,4) for m in mols]; D=[[1-DataStructs.TanimotoSimilarity(a,b) for b in fps] for a in fps]
```

## Evaluation signals

- Every entry of the compound distance matrix lies in [0,1] (Tanimoto-derived), the matrix is symmetric, and the diagonal is exactly zero
- The number of compounds processed equals the number of valid SMILES parsed from MIBiG v1.3 (no silent drops of unparseable entries)
- optimized_class_weights.tsv contains exactly one optimized weight row per BiG-SCAPE class (eight classes)
- For each class, the Adjacency Index's correlation with compound Tanimoto distances exceeds the Goodman-Kruskal metric's correlation under the optimized weights
- Weight enumeration respects the documented grid: 0.01 steps for JI/DSS/GK/AI weights and 0.5 steps for the anchor-boost weights 1-4

## Limitations

- Weight calibration is anchored to the MIBiG reference set; compound coverage of BGC0001840/BGC0001878/BGC0001841-style entries depends on the release used (release 1.3, August 2016)
- Brute-force enumeration over the 0.01/0.5 weight grids is combinatorially expensive and must be repeated separately for each of the eight BiG-SCAPE classes
- The approach requires paired BGC–compound annotations, so BGC classes without sufficient MIBiG chemical reference data cannot be calibrated this way

## Evidence

- [readme] BiG-SCAPE reference BGC and MIBiG input basis: [evidence span withheld — anchor status: not_found]
