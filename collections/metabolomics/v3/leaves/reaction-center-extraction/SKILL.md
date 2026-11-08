---
name: reaction-center-extraction
description: Use when you need a library of reaction-center-based biotransformation
  rules derived from a curated reaction dataset (e.g. RetroRules or KEGG reactions
  with SMILES-annotated metabolites) to drive rule-based derivative generation.
when_to_use_negative:
- You only need spectral library searching or molecular formula annotation, not rule-based
  derivative generation.
- Your reactions lack resolvable chemical structures for substrates/products — metabolites
  with no structure in the provided file or RetroRules are excluded by the pipeline.
- The transformation set must come from a source other than reaction-template databases;
  rules here are sourced only from RetroRules, KEGG, or MetaCyc.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PROXIMAL2
  role: Extracts biotransformation rules (operators) from KEGG/RetroRules reaction
    datasets and applies them by locating suitable reaction centers on query molecules
  repo: https://github.com/HassounLab/PROXIMAL2
- name: BAM
  role: Downstream consumer of the extracted rules; applies them to anchor molecules
    to generate and rank derivative structures
  repo: https://github.com/HassounLab/BAM
- name: Python 3.9
  role: Analyze PROXIMAL2 results and structure the rules file (with PubChemPy, BioServices,
    scikit-learn, NumPy)
- name: RetroRules
  role: Source reaction database; its 351,704 rules (84,614 reactant–product pairs)
    were mined for the 42,307 unique biotransformation rules used here
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:aa1e1ef7eaa6dc4d8c6cbee9b1622dd9609928826c35ad0817b7bf264dda8731
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/PROXIMAL2
  tools:
  - BAM
  - PROXIMAL2
  - Python 3.9
  - RetroRules
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# reaction-center-extraction

## Summary

Extract biotransformation rules—each encoding a reaction-center substructure plus the structural change around it—from reaction databases (RetroRules, KEGG, MetaCyc) using PROXIMAL2, so they can later be applied to anchor molecules to enumerate biotransformed derivatives in BAM.

## When to use

Use when you need a library of reaction-center-based biotransformation rules derived from a curated reaction dataset (e.g. RetroRules or KEGG reactions with SMILES-annotated metabolites) to drive rule-based derivative generation.

## When NOT to use

- You only need spectral library searching or molecular formula annotation, not rule-based derivative generation.
- Your reactions lack resolvable chemical structures for substrates/products — metabolites with no structure in the provided file or RetroRules are excluded by the pipeline.
- The transformation set must come from a source other than reaction-template databases; rules here are sourced only from RetroRules, KEGG, or MetaCyc.

## Inputs

- reactions CSV with columns 'id', 'formula', 'EC' (e.g. from RetroRules or KEGG)
- metabolites CSV with columns 'name', 'hmdb', 'kegg', 'metanetx' (SMILES structures)
- cofactors CSV with name and InChI, used to strip cofactors from reactions
- RetroRules database dump downloaded from https://retrorules.org/dl/retrorules_dump

## Outputs

- reachableMolecules.csv (structures of molecules included in the templates)
- templateReactions.csv (definition of the reaction templates)
- generated biotransformation operators/rules (OP_CACHE_DIRECTORY), e.g. generated_operators.tsv
- final list of metabolite pairs per generated operator (path_finalReactions)

## How to apply

Assemble a reaction dataset as two tab-separated files: a reactions CSV with columns 'id', 'formula', 'EC', and a metabolites CSV with columns 'name', 'hmdb', 'kegg', 'metanetx' (SMILES-resolved via PubChem, HMDB, KEGG, MetaNetX, and RetroRules lookups). Run PROXIMAL2's GenerateInput.py, which removes cofactors from substrates and products (using the cofactors CSV with name and InChI) and discards reactions where all substrates or all products are cofactors, producing reachableMolecules.csv and templateReactions.csv. Then run runPROX2.py to execute ExtractPairs (association and redundancy removal), GenerateOperators (rule/operator generation), and the application steps; write the operators to OP_CACHE_DIRECTORY and the metabolite pairs to path_finalReactions. In the BAM study this yielded 42,307 unique rules from the 351,704 RetroRules rules (representing 84,614 reactant–product pairs), of which 17,254 (41%) matched observed precursor mass differences at unit resolution and were applicable to BAM.

## Related tools

- **PROXIMAL2** (Extracts biotransformation rules (operators) from KEGG/RetroRules reaction datasets and applies them by locating suitable reaction centers on query molecules) — https://github.com/HassounLab/PROXIMAL2
- **BAM** (Downstream consumer of the extracted rules; applies them to anchor molecules to generate and rank derivative structures) — https://github.com/HassounLab/BAM
- **Python 3.9** (Analyze PROXIMAL2 results and structure the rules file (with PubChemPy, BioServices, scikit-learn, NumPy))
- **RetroRules** (Source reaction database; its 351,704 rules (84,614 reactant–product pairs) were mined for the 42,307 unique biotransformation rules used here)

## Examples

```
conda create -n p2 -c conda-forge -c bioconda rdkit pubchempy bioservices && conda activate p2 && pip install kcfconvoy && python runPROX2.py
```

## Evaluation signals

- Rule count sanity: from the 351,704 RetroRules rules (84,614 reactant–product pairs), expect on the order of tens of thousands of unique rules (the study obtained 42,307).
- Mass-difference filterability: a subset of rules should match observed precursor mass differences at unit resolution (study: 17,254 rules, 41% of applicable RetroRules rules).
- Schema check: reactions file must contain 'id', 'formula', 'EC' columns; metabolites file must contain 'name', 'hmdb', 'kegg', 'metanetx' columns.
- Cofactor pruning invariant: reactions consisting solely of cofactors as substrates or products must be removed by GenerateInput.py.
- Operator output exists in OP_CACHE_DIRECTORY and a compound-pair list exists at path_finalReactions for downstream BAM runs.

## Limitations

- Some generated products have no SMILES because RDKit 'do not find chemical sense about the prediction' — the modification applies but no chemically valid product is produced.
- Metabolites absent from the metabolites file and RetroRules are excluded from template generation.
- Only rules matching observed precursor mass differences at unit resolution are usable downstream (41% of RetroRules rules in this study), limiting applicable rule coverage.
- Building operators can be slow — the README notes 'some reactions will take a couple of minutes to build the operators'.

## Evidence

- [methods] rules sourced from reaction databases: [evidence span withheld — anchor status: not_found]
- [methods] PROXIMAL2 extracts and applies rules: "To extract and apply these rules, we employed PROXIMAL2."
- [intro] reaction center concept: "BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes."
- [results] rule extraction yield: "yielding 42,307 unique rules—covering a wide range of possible biochemical transformations. These rules were extracted from the 351,704 biotransformation rules in RetroRules"
- [readme] input file schema: [evidence span withheld — anchor status: not_found]
- [readme] cofactor removal: [evidence span withheld — anchor status: external_doc]
- [readme] PROXIMAL2 pipeline steps: [evidence span withheld — anchor status: not_found]
