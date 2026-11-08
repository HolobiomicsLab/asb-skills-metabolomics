---
name: tanimoto-similarity-computation
description: Decision support for ranking or validating derivative candidates by structural
  similarity, e.g. testing whether BAM-generated derivatives from anchor-suspect pairs
  resemble the suspect more than the anchor.
when_to_use_negative:
- When MS/MS spectral similarity (e.g. cosine or modified cosine in GNPS spectral
  library searching) is the only needed comparison — this skill computes structural-graph
  similarity, not fragmentation-similarity.
- When no anchor-suspect pairing or candidate derivative list exists yet (e.g. before
  BAM/PROXIMAL2 has generated products) — there are no structures to fingerprint.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Parses SMILES/InChI into molecular graphs, computes Morgan fingerprints, and
    calculates Tanimoto similarity between candidates, suspects, and anchors
- name: Pandas
  role: Aggregates per-pair Tanimoto comparisons into candidate_similarities.csv and
    computes the overall enrichment fraction
- name: BAM
  role: Produces the derivative candidates being scored; source of the candidate structures
    compared against suspects and anchors
  repo: https://github.com/HassounLab/BAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:7a36ed00c78fbaa04837143dfd652c79313eec880bfd322571e1ac257592b259
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/BAM
  tools:
  - BAM
  - Pandas
  - RDKit
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# tanimoto-similarity-computation

## Summary

Computes Tanimoto similarity between RDKit Morgan circular fingerprints to score structural agreement of molecular pairs; used in BAM's evaluation to test whether generated derivative candidates are structurally closer to the suspect than to the anchor.

## When to use

Decision support for ranking or validating derivative candidates by structural similarity, e.g. testing whether BAM-generated derivatives from anchor-suspect pairs resemble the suspect more than the anchor.

## When NOT to use

- When MS/MS spectral similarity (e.g. cosine or modified cosine in GNPS spectral library searching) is the only needed comparison — this skill computes structural-graph similarity, not fragmentation-similarity.
- When no anchor-suspect pairing or candidate derivative list exists yet (e.g. before BAM/PROXIMAL2 has generated products) — there are no structures to fingerprint.

## Inputs

- SMILES or InChI strings for candidate derivatives, suspect molecules, and anchor molecules from the BAM evaluation dataset
- Anchor-suspect pair list derived from the global molecular network (molecules_of_interest csv with suspect identifier, suspect mass, anchor identifier, and anchor SMILES)

## Outputs

- candidate_similarities.csv table with per-pair Tanimoto(candidate, suspect) and Tanimoto(candidate, anchor) values
- Aggregate fraction of candidates with Tanimoto(candidate, suspect) >= Tanimoto(candidate, anchor) (reported as 74.0%)

## How to apply

Parse each molecule's SMILES/InChI into an RDKit Mol object (RDKit 2022.03.5) and compute Morgan circular fingerprints. For every candidate derivative produced by PROXIMAL2 and ranked by GNN-SOM, calculate Tanimoto(candidate, suspect) and Tanimoto(candidate, anchor) with RDKit's fingerprint similarity function. Mark each candidate as improved when its similarity to the suspect equals or exceeds its similarity to the anchor; aggregate over all evaluated anchor-suspect pairs to yield the enrichment fraction (74.0% in the BAM evaluation). Write per-pair values to candidate_similarities.csv alongside the aggregate fraction. This structural-similarity metric is computed with Pandas over the similarity tables, providing an orthogonal check on GNN-SOM's site-of-metabolism ranking.

## Related tools

- **RDKit** (Parses SMILES/InChI into molecular graphs, computes Morgan fingerprints, and calculates Tanimoto similarity between candidates, suspects, and anchors)
- **Pandas** (Aggregates per-pair Tanimoto comparisons into candidate_similarities.csv and computes the overall enrichment fraction)
- **BAM** (Produces the derivative candidates being scored; source of the candidate structures compared against suspects and anchors) — https://github.com/HassounLab/BAM

## Examples

```
from rdkit import DataStructs; from rdkit.Chem import AllChem; sim = DataStructs.TanimotoSimilarity(AllChem.GetMorganFingerprintAsBitVect(cand, 2), AllChem.GetMorganFingerprintAsBitVect(suspect, 2))
```

## Evaluation signals

- Every candidate has both Tanimoto(candidate, suspect) and Tanimoto(candidate, anchor) values populated in candidate_similarities.csv
- All Tanimoto values fall in [0.0, 1.0]
- The reported enrichment fraction equals (count of candidates with sim(candidate, suspect) >= sim(candidate, anchor)) / total candidates (74.0% expected on the BAM global molecular network evaluation set)
- RDKit fingerprint computation does not throw for any input SMILES; malformed SMILES that fail parsing are logged and excluded rather than silently scored
- Per-pair rows in candidate_similarities.csv match the number of anchor-suspect pairs with generated derivatives

## Limitations

- The 74.0% enrichment is specific to the global molecular network evaluation set of 30,184 structurally unique, annotated spectrum pairs (60,368 anchor-suspect pairs); results depend on the quality of the anchor-suspect derivation.
- Only derivative candidates for which PROXIMAL2 generated a valid SMILES product can be scored; products where RDKit 'do not find chemical sense about the prediction' are excluded.
- Similarity to the suspect is a structural heuristic and does not verify spectral correctness of the candidate.

## Evidence

- [results] BAM-generated derivative candidates were more or equally Tanimoto-similar to the suspect than to the anchor in 74.0% of cases: [evidence span withheld — anchor status: not_found]
- [methods] Compute Morgan fingerprints with RDKit and Tanimoto between candidate and suspect and anchor: [evidence span withheld — anchor status: not_found]
- [readme] Candidates where RDKit cannot assign chemical sense have no SMILES: [evidence span withheld — anchor status: external_doc]
- [readme] Anchor-suspect pairs are specified as molecules_of_interest csv with suspect mass and anchor SMILES: "molecules_of_interest = csv file of list of queries. It must have an identifier and mass for the suspect as well as anchor and a SMILES that represents the anchor."
