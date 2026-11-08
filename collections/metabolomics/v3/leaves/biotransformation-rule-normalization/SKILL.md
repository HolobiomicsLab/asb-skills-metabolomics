---
name: biotransformation-rule-normalization
description: Use when you need to convert a raw reaction dataset (e.g. RetroRules
  dump, KEGG reaction list) into a clean set of reaction-center substructure rules
  that can be applied to query molecules by PROXIMAL2.
when_to_use_negative:
- When the metabolites CSV is empty or lacks a metabolite present in the reaction
  list — the metabolite is excluded rather than retained (unless only the name and
  any RetroRules information exist).
- When you need mass-difference-matched rule filtering for BAM (e.g. unit-resolution
  precursor mass difference matching to 17,254 rules); that filtering is a separate
  downstream step, not part of rule normalization.
- When applying already-normalized operators to new queries — skip to the GenerateProducts/GenerateMolFiles
  steps of runPROX2.py instead of re-generating operators.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: PROXIMAL2
  role: 'Core tool: extracts and deduplicates reaction-center biotransformation rules
    (operators) from the supplied reaction database and applies them to anchor molecules'
  repo: https://github.com/HassounLab/PROXIMAL2
- name: Python 3.9
  role: Language environment used to run PROXIMAL2 (via conda env 'p2' or 'proximal2')
    and to analyze the generated rule outputs, e.g. producing generated_operators.tsv
- name: PubChemPy 1.0.4
  role: Dependency used in the PROXIMAL2 conda environment for annotating metabolites
    with chemical structures
- name: BioServices 1.11.2
  role: Dependency used in the PROXIMAL2 conda environment for database access during
    input generation
- name: RDKit
  role: RDKit (installed via conda) performs the substructure/SMARTS cheminformatics
    used to build and apply reaction-center operators; products whose SMILES RDKit
    cannot make chemical sense of are flagged
  repo: https://www.rdkit.org/
- name: BAM
  role: 'Downstream consumer of the normalized rules: BAM applies the generated operators
    to anchor molecules for structure discovery'
  repo: https://github.com/HassounLab/BAM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:7a6054b407f800c6cad8bfbf7bc17be8b6010e8ca1d732a2946798b08ae6fa3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/PROXIMAL2
  tools:
  - BAM
  - BioServices 1.11.2
  - PROXIMAL2
  - PubChemPy 1.0.4
  - Python 3.9
  - RDKit
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# biotransformation-rule-normalization

## Summary

Extract and deduplicate reaction-center biotransformation rules from biochemical reaction databases (e.g. RetroRules, KEGG, MetaCyc) via PROXIMAL2, producing a normalized, redundancy-free operator set that can be applied to anchor molecules for structure discovery.

## When to use

Use when you need to convert a raw reaction dataset (e.g. RetroRules dump, KEGG reaction list) into a clean set of reaction-center substructure rules that can be applied to query molecules by PROXIMAL2.

## When NOT to use

- When the metabolites CSV is empty or lacks a metabolite present in the reaction list — the metabolite is excluded rather than retained (unless only the name and any RetroRules information exist).
- When you need mass-difference-matched rule filtering for BAM (e.g. unit-resolution precursor mass difference matching to 17,254 rules); that filtering is a separate downstream step, not part of rule normalization.
- When applying already-normalized operators to new queries — skip to the GenerateProducts/GenerateMolFiles steps of runPROX2.py instead of re-generating operators.

## Inputs

- reactions .csv file (tab-separated) with columns 'id', 'formula', 'EC'
- metabolites .csv file (tab-separated) with columns 'name', 'hmdb', 'kegg', 'metanetx'
- cofactors .csv file with name and InChI of cofactors to remove from reactions
- RetroRules database dump (downloaded from https://retrorules.org/dl/retrorules_dump) or another reaction database (KEGG, MetaCyc)

## Outputs

- reachableMolecules.csv (structures of molecules included in the templates)
- templateReactions.csv (definition of the reaction templates)
- definitive list of reaction pairs saved in the input folder after ExtractPairs
- generated_operators.tsv (structured biotransformation rules)
- path_finalReactions directory listing metabolite pairs represented by the generated operators
- per-query folders containing JSON product records with GeneratedProduct.smiles/mol, TemplateReaction (ec, ID, Substrate, Product), and QueryInformation (name, ID, smiles)

## How to apply

Run PROXIMAL2's GenerateInput.py on the reaction database of interest (e.g. the RetroRules dump downloaded from retrorules.org, or KEGG reactions), supplying a metabolites CSV with 'name', 'hmdb', 'kegg', 'metanetx' columns and a reaction CSV with 'id', 'formula', 'EC' columns; cofactors listed in the cofactors.csv file are stripped from substrates and products, and reactions whose transformations involve only cofactors (or all-substrate/all-product cofactor cases) are removed. GenerateInput.py outputs reachableMolecules.csv and templateReactions.csv, which define the reaction templates. runPROX2.py then performs ExtractPairs (redundancy removal and pair association), GenerateOperators, GenerateProducts, and GenerateMolFiles, writing one operator output folder per query and JSON product records under OP_CACHE_DIRECTORY/OUTPUT_DIRECTORY. The resulting normalized rules are saved as generated_operators.tsv and the final metabolite-pair list as the path_finalReactions directory; in the BAM validation, 42,307 unique rules were obtained from RetroRules' 351,704 rules representing 84,614 reactant-product pairs.

## Related tools

- **PROXIMAL2** (Core tool: extracts and deduplicates reaction-center biotransformation rules (operators) from the supplied reaction database and applies them to anchor molecules) — https://github.com/HassounLab/PROXIMAL2
- **Python 3.9** (Language environment used to run PROXIMAL2 (via conda env 'p2' or 'proximal2') and to analyze the generated rule outputs, e.g. producing generated_operators.tsv)
- **PubChemPy 1.0.4** (Dependency used in the PROXIMAL2 conda environment for annotating metabolites with chemical structures)
- **BioServices 1.11.2** (Dependency used in the PROXIMAL2 conda environment for database access during input generation)
- **RDKit** (RDKit (installed via conda) performs the substructure/SMARTS cheminformatics used to build and apply reaction-center operators; products whose SMILES RDKit cannot make chemical sense of are flagged) — https://www.rdkit.org/
- **BAM** (Downstream consumer of the normalized rules: BAM applies the generated operators to anchor molecules for structure discovery) — https://github.com/HassounLab/BAM

## Examples

```
conda create -n p2 -c conda-forge -c bioconda rdkit pubchempy bioservices; pip install kcfconvoy; python runPROX2.py
```

## Evaluation signals

- Rule count: the normalization should collapse the raw database rules (e.g. RetroRules 351,704 rules / 84,614 reactant-product pairs) into a smaller unique set — BAM reported 42,307 unique rules
- Cofactor filtering: reactions that are transformations only between cofactors, or where all substrates or all products are cofactors, must be absent from templateReactions.csv
- Schema check: reachableMolecules.csv and templateReactions.csv must exist in the input folder after GenerateInput.py; templateReactions.csv must carry the EC, ID, Substrate, and Product fields per reaction pair
- Redundancy check: the ExtractPairs step writes a definitive (deduplicated) list of reaction pairs into the input folder; running it twice must yield the same pair list
- Operator application sanity: for a query, each applied operator must produce a JSON product file with either a valid smiles or a mol field; a missing smiles indicates RDKit could not find chemical sense in the prediction

## Limitations

- Some reactions take a couple of minutes to build operators, so the pipeline is not instant on large databases
- If a metabolite in the reaction list is missing from the metabolites CSV (and no name/RetroRules information exists), it will be excluded rather than retained
- GeneratedProduct.smiles may be absent because 'RDKit do not find chemical sense about the prediction' even though the operator was applied
- Rules sourced from RetroRules, KEGG, or MetaCyc differ in coverage — RetroRules was chosen for its sizable collection, but only rules whose mass differences match observed precursor differences (17,254 rules, 41% of RetroRules) are applicable in BAM's unit-resolution filtering

## Evidence

- [methods] Rule sourcing from RetroRules: [evidence span withheld — anchor status: not_found]
- [readme] GenerateInput.py cofactor filtering and template output: [evidence span withheld — anchor status: external_doc]
- [results] Unique rule count after deduplication: "yielding 42,307 unique rules—covering a wide range of possible biochemical transformations. These rules were extracted from the 351,704 biotransformation rules in RetroRules, which represent 84,614"
- [readme] ExtractPairs redundancy removal: [evidence span withheld — anchor status: not_found]
- [readme] Output JSON product schema: [evidence span withheld — anchor status: external_doc]
- [results] Downstream mass-difference filtering context: "observed precursor mass differences matched 17,254 biotransformation rules, corresponding to 41% of rules from the RetroRules database that are applicable for BAM"
- [readme] Missing metabolite exclusion caveat: [evidence span withheld — anchor status: external_doc]
