---
name: reaction-center-substructure-matching
description: Use when you have an anchor molecule SMILES, a precursor mass difference
  to a suspect molecule, and a reaction dataset (RetroRules/KEGG/MetaCyc) from which
  biotransformation operators should be extracted and matched.
when_to_use_negative:
- Reaction data is already expressed as SMIRKS/SMARTS operators ready to apply to
  the query - skip PROXIMAL2 operator generation and apply directly.
- The suspect molecule is unknown/unannotated with no anchor–suspect mass difference
  available - the operator matching step requires a numeric mass difference between
  anchor and suspect.
- The task is spectral library search or molecular formula annotation rather than
  structure prediction from biotransformation rules - use SIRIUS or library matching
  instead of this skill.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0199
tools:
- name: PROXIMAL2
  role: Extracts biotransformation operators from reaction datasets, identifies suitable
    reaction centers on the anchor molecule, and generates candidate derivative structures.
  repo: https://github.com/HassounLab/PROXIMAL2
- name: GNN-SOM
  role: Predicts likelihood of each atom being the site of metabolism for each candidate
    derivative, used to rank candidates.
  repo: https://github.com/HassounLab/GNN-SOM
- name: RDKit
  role: Underlying cheminformatics library used for operator application and product
    generation (installed via conda for PROXIMAL2 environment).
- name: BAM
  role: Orchestrating tool that invokes PROXIMAL2 and GNN-SOM within the BAM pipeline.
  repo: https://github.com/HassounLab/BAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:0ebe0b974d91e1a3cb41d41f43bb4a9f23cc9996534e851d06907ea38518c5e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/PROXIMAL2
  tools:
  - BAM
  - GNN-SOM
  - KEGG
  - PROXIMAL2
  - PyTorch
  - PyTorch Geometric
  - RDKit
  - RetroRules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# Reaction-center substructure matching

## Summary

Match biotransformation operators to observed anchor–suspect mass differences and apply them as SMARTS-like substructure transformations at candidate reaction centers using PROXIMAL2's operator extraction (extract reaction pairs, generate operators, check applicability to the query, and generate predicted products), producing candidate derivative SMILES ranked by GNN-SOM.

## When to use

Use when you have an anchor molecule SMILES, a precursor mass difference to a suspect molecule, and a reaction dataset (RetroRules/KEGG/MetaCyc) from which biotransformation operators should be extracted and matched.

## When NOT to use

- Reaction data is already expressed as SMIRKS/SMARTS operators ready to apply to the query - skip PROXIMAL2 operator generation and apply directly.
- The suspect molecule is unknown/unannotated with no anchor–suspect mass difference available - the operator matching step requires a numeric mass difference between anchor and suspect.
- The task is spectral library search or molecular formula annotation rather than structure prediction from biotransformation rules - use SIRIUS or library matching instead of this skill.

## Inputs

- reaction_list CSV with columns 'id', 'formula', 'EC' (e.g. from RetroRules, KEGG, or MetaCyc)
- metabolites_list CSV mapping metabolite names/IDs (hmdb, kegg, metanetx) to SMILES
- molecules_of_interest CSV with identifier and mass for the suspect, anchor mass, and the anchor SMILES
- cofactors CSV with names and InChI used to strip cofactors from reactions
- RetroRules dump extracted in the PROXIMAL2 input folder

## Outputs

- Operator cache folder (OP_CACHE_DIRECTORY) containing the generated biotransformation operators
- Product folders per query, labeled 'MetX', containing per-applied-pair JSON files with GeneratedProduct (smiles, mol), TemplateReaction (ec, ID, Substrate, Product), and QueryInformation (name, ID, smiles)
- path_finalReactions: final list of metabolite/compound pairs represented by the generated operators

## How to apply

Prepare the reaction dataset as a reaction_list CSV with columns 'id', 'formula', 'EC' and a metabolites_list CSV mapping metabolites to SMILES. Run PROXIMAL2's GenerateInput.py to remove cofactors from substrates/products and drop reactions that are transformations only between cofactors, saving reachableMolecules.csv and templateReactions.csv. Run runPROX2.py to (1) extract reaction pairs with redundancy removal, (2) generate operators, (3) check operator applicability to the query molecule, and (4) generate predicted product structures as JSON files (GeneratedProduct.smiles/m mol, TemplateReaction.ec/ID/Substrate/Product, QueryInformation) into per-query folders labeled 'MetX'. Match operators whose mass difference equals the observed anchor–suspect precursor mass difference, then use GNN-SOM to score likelihood of each atom being the site of metabolism and rank candidates using average-rank tie resolution. Finally validate by checking the rank of the true suspect among the candidates.

## Related tools

- **PROXIMAL2** (Extracts biotransformation operators from reaction datasets, identifies suitable reaction centers on the anchor molecule, and generates candidate derivative structures.) — https://github.com/HassounLab/PROXIMAL2
- **GNN-SOM** (Predicts likelihood of each atom being the site of metabolism for each candidate derivative, used to rank candidates.) — https://github.com/HassounLab/GNN-SOM
- **RDKit** (Underlying cheminformatics library used for operator application and product generation (installed via conda for PROXIMAL2 environment).)
- **BAM** (Orchestrating tool that invokes PROXIMAL2 and GNN-SOM within the BAM pipeline.) — https://github.com/HassounLab/BAM

## Examples

```
sh runBAM.sh
```

## Evaluation signals

- Check that a 'reachableMolecules.csv' and 'templateReactions.csv' are generated in the input folder after running GenerateInput.py.
- Verify that each query produces a 'MetX' output folder containing per-pair JSON product files with the expected keys (GeneratedProduct.smiles/mol, TemplateReaction.ec/ID/Substrate/Product, QueryInformation).
- Confirm that the operator applied to the query produces a product whose molecular mass matches the anchor–suspect precursor mass difference.
- Validate by checking the rank of the true suspect among the ranked candidate list (acceptance: proportion of true suspect identities ranked at positions 1–5).
- Verify cofactor removal: no reactions remain in templateReactions.csv that represent transformations only between cofactors.

## Limitations

- RDKit may fail to assign chemical sense to a predicted product even though the operator was successfully applied (GeneratedProduct.smiles can be absent).
- Only 41% of RetroRules rules were applicable at unit-resolution mass-difference matching in the article's evaluation, so many biotransformation rules will not match an observed mass difference.
- Operator generation can be slow for some reactions - 'Note that some reactions will take a couple of minutes to build the operators.'

## Evidence

- [results] PROXIMAL2 operator extraction and matching to observed mass difference: "observed precursor mass differences matched 17,254 biotransformation rules, corresponding to 41% of rules from the RetroRules database that are applicable for BAM"
- [intro] BAM applies biotransformation rules to the anchor molecule via PROXIMAL2 to identify suitable reaction centers: "BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes."
- [intro] GNN-SOM ranks candidate derivatives by site-of-metabolism likelihood: "the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives based on the likelihood of each atom being a reaction center."
- [readme] PROXIMAL2 input generation requires specific CSV columns for reactions and metabolites: [evidence span withheld — anchor status: not_found]
- [readme] PROXIMAL2 operator applicability check generates predicted products with explicit JSON schema: [evidence span withheld — anchor status: external_doc]
- [readme] PROXIMAL2 removes cofactors from reaction substrates/products before operator generation: [evidence span withheld — anchor status: external_doc]
- [methods] Average-rank tie resolution is used when ranking candidates: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
