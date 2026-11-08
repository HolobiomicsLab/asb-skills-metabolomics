---
name: molecule-structure-enumeration
description: Use when you have an annotated anchor molecule (SMILES) paired with an
  unannotated suspect via a precursor mass difference, and need candidate structures
  ranked by biotransformation likelihood.
when_to_use_negative:
- The query spectrum already has a spectral-library match — direct library annotation
  is more appropriate than rule-based enumeration
- No spectrally similar annotated anchor exists in the molecular network for the unknown
  spectrum
- Input is a set of independent spectra without anchor–suspect mass-difference pairs
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: PROXIMAL2
  role: Extracts biotransformation operators from reaction datasets and applies them
    to the anchor molecule to generate candidate derivatives
  repo: https://github.com/HassounLab/PROXIMAL2
- name: GNN-SOM
  role: Graph neural network that predicts the likelihood of each atom being the site
    of metabolism, used to rank candidate derivatives
  repo: https://github.com/HassounLab/GNN-SOM
- name: RDKit
  role: Parses SMILES and generates predicted product structures during operator application
  repo: https://www.rdkit.org/
- name: PyTorch Geometric
  role: Framework underlying the GNN-SOM site-of-metabolism model
  repo: https://pytorch-geometric.readthedocs.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:a0e11b024f07c1a1023100fcded10617cf8cc0ba0d3b5e9a41d28756c4638fef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/PROXIMAL2
  tools:
  - GNN-SOM
  - KEGG
  - PROXIMAL2
  - PyTorch
  - PyTorch Geometric
  - RDKit
  - RetroRules
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# molecule-structure-enumeration

## Summary

Enumerate and rank candidate molecular structures for an unknown 'suspect' metabolite by applying biotransformation rules to a spectrally similar known 'anchor' molecule, using PROXIMAL2 to extract operators and GNN-SOM to rank derivatives by predicted site-of-metabolism.

## When to use

Use when you have an annotated anchor molecule (SMILES) paired with an unannotated suspect via a precursor mass difference, and need candidate structures ranked by biotransformation likelihood.

## When NOT to use

- The query spectrum already has a spectral-library match — direct library annotation is more appropriate than rule-based enumeration
- No spectrally similar annotated anchor exists in the molecular network for the unknown spectrum
- Input is a set of independent spectra without anchor–suspect mass-difference pairs

## Inputs

- molecules_of_interest.csv: identifier, suspect mass, anchor mass, and anchor SMILES
- reaction_list.csv with columns 'id', 'formula', 'EC'
- metabolites_list.csv of metabolites and their SMILES in the reaction dataset
- Reaction dataset (RetroRules, KEGG, or MetaCyc)

## Outputs

- Ranked candidate derivative structures for the suspect molecule
- Per-candidate rank with true suspect rank reported for validation
- Predicted product SMILES / mol records in per-query folders

## How to apply

Load the metabolite list, reaction list (columns 'id', 'formula', 'EC'), and a molecules-of-interest CSV specifying the anchor SMILES and anchor–suspect mass difference. Run PROXIMAL2 to extract biotransformation operators from the reaction dataset (e.g. RetroRules or KEGG), matching each operator's mass difference to the observed precursor mass difference between anchor and suspect. Apply matched operators to the anchor molecule at identified reaction centers to generate candidate derivative structures. Then run GNN-SOM on each derivative to predict the likelihood of each atom being the site of metabolism, ranking derivatives by the likelihood assigned to the atom to which the rule was applied. Resolve ties using the average-rank rule (sum of ranks divided by the number of tied candidates).

## Related tools

- **PROXIMAL2** (Extracts biotransformation operators from reaction datasets and applies them to the anchor molecule to generate candidate derivatives) — https://github.com/HassounLab/PROXIMAL2
- **GNN-SOM** (Graph neural network that predicts the likelihood of each atom being the site of metabolism, used to rank candidate derivatives) — https://github.com/HassounLab/GNN-SOM
- **RDKit** (Parses SMILES and generates predicted product structures during operator application) — https://www.rdkit.org/
- **PyTorch Geometric** (Framework underlying the GNN-SOM site-of-metabolism model) — https://pytorch-geometric.readthedocs.io/

## Examples

```
sh runBAM.sh
```

## Evaluation signals

- Rank of the true suspect among candidate derivatives should fall in the top 5; the acceptance criterion is the proportion of true suspect identities ranked at positions 1 to 5
- Candidate derivative structures must be chemically valid (RDKit must find chemical sense; invalid predictions leave the 'smiles' field empty in the JSON output)
- Each generated product JSON should contain fields for GeneratedProduct, TemplateReaction, and QueryInformation
- A per-query output folder labelled 'MetX' should exist for each query, containing subfolders for each applied operator pair

## Limitations

- Only biotransformation rules whose mass difference matches the observed anchor–suspect precursor mass difference can be applied; unmatched transformations are excluded
- RDKit may fail to assign a chemically sensible SMILES even when the operator was applied
- Ranking depends on GNN-SOM site-of-metabolism likelihoods; ties are resolved by average rank, which can dilute true ranking signal

## Evidence

- [intro] PROXIMAL2 identifies reaction centers: "BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes."
- [intro] GNN-SOM ranks derivatives by SOM likelihood: "the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives based on the likelihood of each atom being a reaction center."
- [methods] Average-rank tie resolution: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [readme] Product JSON structure from PROXIMAL2: "Any product output within the folder of the related pair is defined in json format, containing the following fields:
- *GeneratedProduct*:"
- [readme] Run command and input specification: "All input information required is specified in runBAM.sh."
- [results] Validation criterion: top-5 rank: [evidence span withheld — anchor status: not_found]
