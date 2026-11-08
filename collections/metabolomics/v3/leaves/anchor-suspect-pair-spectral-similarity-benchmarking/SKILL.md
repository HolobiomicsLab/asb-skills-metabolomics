---
name: anchor-suspect-pair-spectral-similarity-benchmarking
description: Use when you have anchor–suspect spectral pairs (e.g. from GNPS molecular
  networking) and need to quantify what fraction of suspect structures can be recovered
  by applying biotransformation rules and ranking derivatives.
when_to_use_negative:
- The queries already have candidate structures resolved by spectral library matching
  or by formula/structure annotation with SIRIUS rather than needing biotransformation-based
  annotation
- The data are not anchor–suspect spectral pairs from molecular networking (no annotated
  anchor with high spectral similarity to the query spectrum)
- Observed precursor mass differences cannot be matched at unit resolution to biotransformation
  rules, so no derivative will be generated
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: BAM
  role: 'Main pipeline: matches mass differences to rules, generates and ranks derivatives
    for anchor–suspect pairs via runBAM.sh'
  repo: https://github.com/HassounLab/BAM
- name: PROXIMAL2
  role: Extracts biotransformation rules into operators and applies them to anchor
    molecules to identify reaction centers and generate candidate derivatives
  repo: https://github.com/HassounLab/PROXIMAL2
- name: GNN-SOM
  role: Graph neural network that predicts site-of-metabolism likelihood to rank generated
    derivatives
  repo: https://github.com/HassounLab/GNN-SOM
- name: Python 3.9 / PubChemPy / BioServices / scikit-learn
  role: Analysis stack (Python 3.9, PubChemPy 1.0.4, BioServices 1.11.2, scikit-learn
    1.1.2) used to analyze PROXIMAL2 results and evaluate correctness
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:4de1c4283d1c771f031f7ec38fc2e517845258824bb246a57f90035da4fa2be8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/BAM
  tools:
  - BAM
  - BioServices
  - GNN-SOM
  - PROXIMAL2
  - PubChemPy
  - Python 3.9
  - Python 3.9 / PubChemPy / BioServices / scikit-learn
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# anchor-suspect-pair-spectral-similarity-benchmarking

## Summary

Benchmark how often BAM correctly annotates suspect structures across anchor–suspect pairs from a global molecular network, by generating and ranking biotransformation-rule derivatives and scoring exact InChIKey matches; the article reports a 24.2% correct-structure assignment rate.

## When to use

Use when you have anchor–suspect spectral pairs (e.g. from GNPS molecular networking) and need to quantify what fraction of suspect structures can be recovered by applying biotransformation rules and ranking derivatives.

## When NOT to use

- The queries already have candidate structures resolved by spectral library matching or by formula/structure annotation with SIRIUS rather than needing biotransformation-based annotation
- The data are not anchor–suspect spectral pairs from molecular networking (no annotated anchor with high spectral similarity to the query spectrum)
- Observed precursor mass differences cannot be matched at unit resolution to biotransformation rules, so no derivative will be generated

## Inputs

- molecules_of_interest CSV with identifier, suspect mass, anchor mass, and anchor SMILES (default: global molecular network anchor–suspect pairs, 60,368 pairs)
- RetroRules reaction dataset (351,704 rules / 84,614 reactant–product pairs), optionally KEGG
- reaction_list CSV with columns 'id', 'formula', 'EC'
- metabolites_list CSV of metabolites and their SMILES structures
- PROXIMAL2 and GNN-SOM repositories placed under BAM-main with conda envs 'proximal2' and 'som'

## Outputs

- Per-pair benchmark table recording correctness of exact first-14-character InChIKey matches
- benchmark_results.csv with the overall correct-structure assignment rate (target: 24.2%)

## How to apply

Load the 60,368 anchor–suspect pairs (from 30,184 structurally unique annotated spectrum pairs) from the Zenodo deposit and set them as BAM's molecules_of_interest input in runBAM.sh. Stage PROXIMAL2 and GNN-SOM under the BAM-main directory and create the 'proximal2' and 'som' conda environments, then extract biotransformation rules from RetroRules (optionally KEGG) and apply them with PROXIMAL2 to generate candidate derivatives whose precursor mass differences match observed unit-resolution mass differences. Rank generated derivatives with GNN-SOM site-of-metabolism likelihoods, resolving rank ties by averaging (sum of ranks including duplicates divided by the number of tied candidates). Score each pair by exact first-14-character InChIKey matching between the top-ranked derivative and the known suspect, then compute the overall correct-structure assignment rate (correct on 4,171 of 17,271 derivative-generating queries = 24.2% recall) and write benchmark_results.csv.

## Related tools

- **BAM** (Main pipeline: matches mass differences to rules, generates and ranks derivatives for anchor–suspect pairs via runBAM.sh) — https://github.com/HassounLab/BAM
- **PROXIMAL2** (Extracts biotransformation rules into operators and applies them to anchor molecules to identify reaction centers and generate candidate derivatives) — https://github.com/HassounLab/PROXIMAL2
- **GNN-SOM** (Graph neural network that predicts site-of-metabolism likelihood to rank generated derivatives) — https://github.com/HassounLab/GNN-SOM
- **Python 3.9 / PubChemPy / BioServices / scikit-learn** (Analysis stack (Python 3.9, PubChemPy 1.0.4, BioServices 1.11.2, scikit-learn 1.1.2) used to analyze PROXIMAL2 results and evaluate correctness)

## Examples

```
sh runBAM.sh
```

## Evaluation signals

- Among queries where BAM generated derivatives (17,271 queries), the correct suspect is ranked in 4,171 cases, i.e. recall of 24.2% — the benchmark value to reproduce
- Correctness per pair verified by exact first-14-character InChIKey match between generated derivative and suspect structure
- Intermediate counts should match: 454,091 annotated nodes, 30,184 structurally unique pairs, 60,368 anchor–suspect pairs
- Rule filtering invariants: 42,307 unique rules extracted from RetroRules; 17,254 rules matched observed precursor mass differences at unit resolution (41% applicable)
- Tie resolution invariant: tied candidate ranks resolved as the average rank (sum of ranks including duplicates divided by the number of tied candidates)

## Limitations

- 24.2% is conditional recall 'when a derivative is generated'; pairs where no derivative is produced are not credited, so overall coverage depends on rule–mass-difference matching
- Only biotransformation rules matched at unit-resolution precursor mass differences apply; only 41% of RetroRules rules were applicable for BAM
- PROXIMAL2 predictions may have no chemically sensible RDKit SMILES (smiles field may be absent even when modifications apply)
- GNN-SOM predictions may produce duplicate ranks, requiring average-rank tie resolution; exact ranking affects top-1 correctness

## Evidence

- [other] Among the queries where BAM generated one or more derivatives (17,271 queries), it generated the correct suspect structure in 4171 cases, achieving a recall of 24.2%: [evidence span withheld — anchor status: not_found]
- [abstract] BAM assigned correct molecular structures to 24.2% of examined anchor-suspect cases: "BAM was able to assign correct molecular structures to 24.2% of examined anchor-suspect cases"
- [intro] BAM makes use of PROXIMAL2 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers: "BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes."
- [intro] the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives: "the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives"
- [methods] To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates.: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [readme] both conda environments need to be created as described in those repositories and named "proximal2" and "som": "To use BAM, these tools need to be downloaded and included under the "BAM-main" directory. Also, both conda environments need to be created as described in those repositories and named "proximal2""
- [readme] molecules_of_interest = csv file of list of queries with identifier and mass for suspect and anchor: "molecules_of_interest = csv file of list of queries. It must have an identifier and mass for the suspect as well as anchor and a SMILES that represents the anchor."
- [readme] Once the code and conda environments are structured as specified ... run the runBAM.sh file: [evidence span withheld — anchor status: not_found]
