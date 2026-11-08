---
name: biotransformation-rule-application
description: Use when an MS/MS query spectrum is unannotated but a spectral-similarity
  neighbor (anchor) with a known SMILES and a precursor mass difference is available,
  and candidate molecular structures for the suspect are needed.
when_to_use_negative:
- When the query spectrum already has a library-annotated molecular structure from
  standard spectral library searching against the default GNPS spectral libraries
- When no spectral-similarity anchor molecule with a known structure and similar precursor
  mass is available for the query
- When only molecular formula annotation is required (use formula/structure tools
  like SIRIUS instead of rule-based structure enumeration)
edam_operation: http://edamontology.org/operation_2950
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: PROXIMAL2
  role: Applies biotransformation rules to the anchor molecule, identifying suitable
    reaction centers for the proposed structural changes
  repo: https://github.com/HassounLab/PROXIMAL2
- name: GNN-SOM
  role: Graph neural network tool that predicts the likelihood of each atom in a derivative
    being the site of metabolism, used to rank putative derivatives
  repo: https://github.com/HassounLab/GNN-SOM
- name: BAM
  role: Orchestrates the rule extraction, rule application (via PROXIMAL2), derivative
    generation, and GNN-SOM ranking pipeline
  repo: https://github.com/HassounLab/BAM
- name: RDKit
  role: Used inside PROXIMAL2 to validate whether applied biotransformations yield
    chemically sensible products (generated SMILES)
- name: SIRIUS
  role: Baseline molecular formula and structure annotation tool used for comparison
    against BAM's rule-based annotation (version 5.6.3)
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:12140d31b006ea606b0c1a5ae1f7134070dc270837d35ee635e12b80b8addff9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/PROXIMAL2
  tools:
  - BAM
  - BioServices
  - GNN-SOM
  - KEGG
  - PROXIMAL2
  - PubChemPy
  - PyTorch
  - PyTorch Geometric
  - Python 3.9
  - RDKit
  - RetroRules
  - SIRIUS
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2950
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 24
---

# biotransformation-rule-application

## Summary

Apply biotransformation rules to a known 'anchor' molecule to enumerate and rank candidate derivative structures for a spectrally similar unknown 'suspect' molecule, using PROXIMAL2 for rule application and GNN-SOM for site-of-metabolism ranking.

## When to use

Use when an MS/MS query spectrum is unannotated but a spectral-similarity neighbor (anchor) with a known SMILES and a precursor mass difference is available, and candidate molecular structures for the suspect are needed.

## When NOT to use

- When the query spectrum already has a library-annotated molecular structure from standard spectral library searching against the default GNPS spectral libraries
- When no spectral-similarity anchor molecule with a known structure and similar precursor mass is available for the query
- When only molecular formula annotation is required (use formula/structure tools like SIRIUS instead of rule-based structure enumeration)

## Inputs

- molecules_of_interest CSV file with identifier and mass for suspect and anchor plus anchor SMILES
- reaction_list CSV file with columns 'id', 'formula', 'EC' (e.g. from RetroRules or KEGG)
- metabolites_list CSV file of metabolites and their structures (SMILES) included in the reaction dataset
- RetroRules database dump (biotransformation rules)
- PROXIMAL2 operators cache directory and GNN-SOM model state/config files

## Outputs

- Per-pair JSON product files containing 'GeneratedProduct' (smiles, mol), 'TemplateReaction' (ec, ID, Substrate, Product), and 'QueryInformation' (name, ID, smiles)
- Ranked list of candidate derivative structures per anchor-suspect pair
- Per-pair correctness outcome (correct structure assignment) by first-14-character InChIKey match

## How to apply

First, match the observed precursor mass difference between the anchor and suspect against biotransformation rules extracted from a reaction dataset (RetroRules, KEGG, or MetaCyc) to identify applicable rules. Then, apply PROXIMAL2 to the anchor's SMILES to apply those rules, identifying suitable reaction centers and generating candidate derivative structures as SMILES/mol text per anchor-suspect pair. Next, run GNN-SOM on each generated derivative to predict the likelihood of each atom being a site of metabolism, and rank the derivatives by the likelihood assigned to the atom to which the biotransformation rule was applied. Resolve rank ties using the average rank rule: sum all ranks including duplicates and divide by the number of tied candidates. Validate correctness per anchor-suspect pair by exact matching of the first 14 characters of the InChIKey between the top-ranked candidate and the true suspect.

## Related tools

- **PROXIMAL2** (Applies biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes) — https://github.com/HassounLab/PROXIMAL2
- **GNN-SOM** (Graph neural network tool that predicts the likelihood of each atom in a derivative being the site of metabolism, used to rank putative derivatives) — https://github.com/HassounLab/GNN-SOM
- **BAM** (Orchestrates the rule extraction, rule application (via PROXIMAL2), derivative generation, and GNN-SOM ranking pipeline) — https://github.com/HassounLab/BAM
- **RDKit** (Used inside PROXIMAL2 to validate whether applied biotransformations yield chemically sensible products (generated SMILES))
- **SIRIUS** (Baseline molecular formula and structure annotation tool used for comparison against BAM's rule-based annotation (version 5.6.3))

## Examples

```
sh runBAM.sh
```

## Evaluation signals

- Per anchor-suspect pair, verify that the top-ranked candidate's InChIKey prefix (first 14 characters) matches the known suspect's InChIKey prefix
- Confirm that the fraction of correct assignments across examined pairs reproduces the reported benchmark (24.2% correct-structure assignment when a derivative is generated)
- Check that a biotransformation rule was only applied when its rule mass difference matched the observed anchor-to-suspect precursor mass difference
- Verify per-pair product JSONs contain both 'GeneratedProduct' (with a non-empty 'smiles' field) and 'TemplateReaction' provenance fields (ec, ID, Substrate, Product)
- Confirm ranking ties are resolved by the average rank rule (sum of duplicate ranks divided by number of tied candidates)

## Limitations

- At unit mass resolution, only 17,254 of 42,307 unique RetroRules-derived biotransformation rules matched observed precursor mass differences, so many rules are not applicable for BAM
- RDKit may fail to find chemical sense about the prediction, in which case 'GeneratedProduct.smiles' may not be generated even though the modification was applied
- In benchmarking, only 4,171 of 17,271 queries with generated derivatives were correctly annotated (24.2%), so most candidate structures produced by this skill are not the true suspect

## Evidence

- [intro] BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes.: "BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers"
- [intro] the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives: "the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives"
- [methods] To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates.: "the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates"
- [results] observed precursor mass differences matched 17,254 biotransformation rules, corresponding to 41% of rules from the RetroRules database that are applicable for BAM: "observed precursor mass differences matched 17,254 biotransformation rules, corresponding to 41% of rules"
- [readme] Once the code and conda environments are structured as specified in the Installation and Requirements section  above, run the runBAM.sh file as it is to run the algorithm: "run the runBAM.sh file as it is to run the algorithm with the files as described here"
