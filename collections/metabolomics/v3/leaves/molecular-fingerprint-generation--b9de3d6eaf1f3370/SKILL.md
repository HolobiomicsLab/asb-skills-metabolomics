---
name: molecular-fingerprint-generation
description: Use when a generated list of candidate structures must be ranked or evaluated
  by structural similarity to a known reference molecule (e.g., suspect vs. anchor)
  using Tanimoto similarity on molecular fingerprints.
when_to_use_negative:
- When the analysis goal is MS/MS spectral similarity scoring (e.g., cosine or modified
  cosine on GNPS spectral library matching) rather than molecular-graph structural
  similarity.
- When candidate ranking depends on site-of-metabolism likelihood from GNN-SOM rather
  than structural similarity to a reference structure.
- When molecules lack parseable structures (no SMILES or InChI available) so fingerprints
  cannot be computed.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: RDKit
  role: Parse structures from SMILES/InChI, compute Morgan circular fingerprints,
    and calculate Tanimoto similarity between candidate, suspect, and anchor molecules.
  repo: https://www.rdkit.org/
- name: Pandas
  role: Aggregate per-pair similarity results and write the candidate_similarities.csv
    output table.
- name: BAM
  role: Produces the candidate derivative structures whose similarity to suspect and
    anchor is evaluated by this skill.
  repo: https://github.com/HassounLab/BAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:8f717b4d923f0808c1bb0c5809a29876308bbc4aba7204f47c1e1e50f9977bf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BAM
  - Pandas
  - RDKit
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# molecular-fingerprint-generation

## Summary

Compute Tanimoto-similarity fingerprints for candidate derivative structures, suspects, and anchors to score and evaluate structure hypotheses in biotransformation-based metabolite annotation workflows such as BAM.

## When to use

Use when a generated list of candidate structures must be ranked or evaluated by structural similarity to a known reference molecule (e.g., suspect vs. anchor) using Tanimoto similarity on molecular fingerprints.

## When NOT to use

- When the analysis goal is MS/MS spectral similarity scoring (e.g., cosine or modified cosine on GNPS spectral library matching) rather than molecular-graph structural similarity.
- When candidate ranking depends on site-of-metabolism likelihood from GNN-SOM rather than structural similarity to a reference structure.
- When molecules lack parseable structures (no SMILES or InChI available) so fingerprints cannot be computed.

## Inputs

- Generated candidate derivative structures (SMILES/InChI) from BAM biotransformation rules
- Suspect molecular structures (SMILES/InChI)
- Anchor molecular structures (SMILES/InChI)
- Anchor–suspect pair metadata from the global molecular network evaluation dataset

## Outputs

- Tanimoto similarity values per candidate against suspect and anchor
- Boolean indicator per candidate of whether Tanimoto(candidate, suspect) >= Tanimoto(candidate, anchor)
- Aggregate enrichment fraction over all evaluated pairs
- Per-pair similarity results table (e.g., candidate_similarities.csv)

## How to apply

Parse molecular structures (from SMILES or InChI) into RDKit molecule objects for candidates, suspects, and anchors. Compute Morgan circular fingerprints for each molecule, then calculate the Tanimoto coefficient between each generated derivative candidate and both the suspect and the anchor structure for every anchor–suspect pair. For each candidate, compare Tanimoto(candidate, suspect) against Tanimoto(candidate, anchor) to determine whether the candidate is more (or equally) similar to the suspect than to the anchor. Aggregate over all evaluated pairs to compute the fraction of candidates enriched toward the suspect, writing per-pair results to a table such as candidate_similarities.csv alongside the aggregate enrichment value. The article used RDKit version 2022.03.5 for fingerprint computation and used this similarity comparison to demonstrate that BAM improved structural hypotheses in 74.0% of cases.

## Related tools

- **RDKit** (Parse structures from SMILES/InChI, compute Morgan circular fingerprints, and calculate Tanimoto similarity between candidate, suspect, and anchor molecules.) — https://www.rdkit.org/
- **Pandas** (Aggregate per-pair similarity results and write the candidate_similarities.csv output table.)
- **BAM** (Produces the candidate derivative structures whose similarity to suspect and anchor is evaluated by this skill.) — https://github.com/HassounLab/BAM

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fps = [AllChem.GetMorganFingerprintAsBitVect(m, 2) for m in mols]; sim = DataStructs.TanimotoSimilarity(fps[0], fps[1])
```

## Evaluation signals

- Per-pair table contains, for every candidate, both Tanimoto(candidate, suspect) and Tanimoto(candidate, anchor) values in [0, 1].
- Boolean enrichment flag per candidate matches the comparison Tanimoto(candidate, suspect) >= Tanimoto(candidate, anchor).
- Aggregate enrichment fraction is a value in [0, 1] summarizing the boolean flags across all evaluated pairs (article reports 74.0%).
- All molecular structures successfully parse into RDKit Mol objects with no SMILES/InChI failures.
- Fingerprints are consistent Morgan circular fingerprints computed with a fixed RDKit version (2022.03.5 in the article) for reproducibility.

## Limitations

- Tanimoto similarity on fingerprints is a structural proxy; it does not capture spectral or metabolic plausibility of the candidate.
- Ties (equal similarity to suspect and anchor) are counted as 'more (or equally) similar to the suspect', which slightly inflates the enrichment fraction.
- Requires successfully parseable structures; RDKit may fail on some SMILES/InChI inputs (the article noted that some products could not be generated because RDKit found no chemical sense in the prediction).

## Evidence

- [results] BAM-generated derivative candidates were more or equally Tanimoto-similar to the suspect than to the anchor in 74.0% of cases, indicating that BAM improves structural hypotheses for metabolite annotation.: [evidence span withheld — anchor status: not_found]
- [other] RDKit Morgan circular fingerprints and Tanimoto comparison procedure used to evaluate BAM candidates.: [evidence span withheld — anchor status: not_found]
- [readme] Structural evaluation of products is performed after RDKit parsing; unparseable predictions are excluded.: [evidence span withheld — anchor status: not_found]
