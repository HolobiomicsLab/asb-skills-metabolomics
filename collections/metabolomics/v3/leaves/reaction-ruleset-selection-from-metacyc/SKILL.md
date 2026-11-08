---
name: reaction-ruleset-selection-from-metacyc
description: Use when setting up a Pickaxe/MINE-Database network expansion and you
  must choose which reaction ruleset and rule subset (e.g., top 500 vs. top 50 JN1224min
  rules) to apply to a starting compound set.
when_to_use_negative:
- You already have user-defined custom reaction rules rather than needing to pick
  from the MetaCyc-derived JN1224min ruleset
- The analysis goal is filtering or annotating an already-generated reaction network
  (e.g., similarity sampling, metabolomics m/z filtering, or thermodynamic cutoffs),
  not choosing the ruleset itself
- The expansion requires non-enzymatic or hand-curated transformation patterns that
  the MetaCyc-derived ruleset does not encode
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Pickaxe
  role: Applies the selected reaction rules iteratively to a set of metabolites to
    generate novel reactions and compounds
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package that ships Pickaxe and the prepackaged JN1224min ruleset; installed
    via pip and provides the Pickaxe run template
  repo: https://pypi.org/project/minedatabase/
- name: MongoDB / PyMongo
  role: Stores the generated reactions and compounds produced by the rule-driven expansion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:0b4defe87cd74c8c95fb9757a7c19ee26dafc829d7961fbd43d05b493cf3d843
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MongoDB / PyMongo
  - Pickaxe
  - minedatabase
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# reaction-ruleset-selection-from-metacyc

## Summary

Selects reaction rules from the MetaCyc-derived JN1224min ruleset (or user-defined custom rules) to drive Pickaxe's rule-based network expansion, balancing rule count against MetaCyc reaction-mapping coverage (e.g., top 500 rules = 94% mapping, top 50 rules = 65% mapping).

## When to use

Use when setting up a Pickaxe/MINE-Database network expansion and you must choose which reaction ruleset and rule subset (e.g., top 500 vs. top 50 JN1224min rules) to apply to a starting compound set.

## When NOT to use

- You already have user-defined custom reaction rules rather than needing to pick from the MetaCyc-derived JN1224min ruleset
- The analysis goal is filtering or annotating an already-generated reaction network (e.g., similarity sampling, metabolomics m/z filtering, or thermodynamic cutoffs), not choosing the ruleset itself
- The expansion requires non-enzymatic or hand-curated transformation patterns that the MetaCyc-derived ruleset does not encode

## Inputs

- Prepackaged JN1224min reaction ruleset (MetaCyc-derived, distributed with the MINE-Database package)
- MetaCyc coreactants file (e.g., metacyc_coreactants.tsv) paired with the rules file (e.g., metacyc_generalized_rules.tsv)
- Starting compound set (optionally pre-filtered by molecular weight, e.g., <600 g/mol)

## Outputs

- Selected rule subset (e.g., top 500 or top 50 rules) with its MetaCyc mapping percentage
- Generated novel reactions and compounds from the one- or multi-generation expansion, stored in a mongo database or written to .tsv files

## How to apply

Start from the prepackaged JN1224min ruleset derived from MetaCyc, which the paper found maps the largest percentage of KEGG and BRENDA reactions of any open-source ruleset. Rank rules and select a subset sized to your expansion goal: the paper used the top 500 rules (94% of MetaCyc mapped) for one-generation expansions of KEGG (Release 99.0), EcoCyc (Version 25.1), and YMDB (Version 2.0) compounds <600 g/mol, but the top 50 rules (65% MetaCyc mapping) for the two-generation EcoCyc expansion to keep combinatorial growth manageable. Supply the rules file (e.g., metacyc_generalized_rules.tsv) together with the corresponding coreactants file (e.g., metacyc_coreactants.tsv) to Pickaxe via the -C and -r arguments, or specify them in a pickaxe_run_template.py run file, then set the generation count (-g). Larger rule sets trade broader coverage for exponential network growth, so match the subset size to the planned number of generations.

## Related tools

- **Pickaxe** (Applies the selected reaction rules iteratively to a set of metabolites to generate novel reactions and compounds) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package that ships Pickaxe and the prepackaged JN1224min ruleset; installed via pip and provides the Pickaxe run template) — https://pypi.org/project/minedatabase/
- **MongoDB / PyMongo** (Stores the generated reactions and compounds produced by the rule-driven expansion)

## Examples

```
python pickaxe.py -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- The rule subset size matches the intended design (e.g., exactly 500 or 50 rules selected from JN1224min)
- The MetaCyc mapping coverage of the selected rules matches the paper's reported values (94% for top 500 rules, 65% for top 50 rules)
- Both the rules file and the coreactants file are supplied to Pickaxe; running without the coreactant file should be treated as an error
- One-generation expansion with the top 500 rules on <600 g/mol compounds reproduces the paper's counts (KEGG: 13,759,651 reactions / 9,562,940 compounds; EcoCyc: 1,388,653 / 906,086; YMDB: 892,343 / 575,262)
- Rule selection is recorded alongside the generation count (-g) and starting-compound filter in the run configuration for reproducibility

## Limitations

- Rule subset size drives exponential network growth — 500 rules on KEGG yielded ~13.7M reactions in one generation, so larger subsets or more generations quickly become computationally intractable
- No version or accession is specified for the MetaCyc source of the JN1224min reaction rules, complicating exact reproducibility
- The command-line Pickaxe entry point must be run as a module (-m flag) from the MINE-Database directory due to relative imports

## Evidence

- [abstract] prepackaged ruleset choice: "Users can select rules from the prepackaged JN1224min ruleset, derived from MetaCyc, or define their own custom rules."
- [intro] JN1224min coverage advantage: "the JN1224min ruleset maps the largest percentage of KEGG and BRENDA reactions, providing the most comprehensive coverage of any ruleset"
- [results] top 500 rule selection parameters: "The top 500 rules (94% of MetaCyc mapped) from the JN1224min ruleset were applied to the starting compounds for one generation."
- [results] smaller subset for multi-generation: "Compounds less than 600 Da selected from the EcoCyc database were expanded for two generations using the top 50 JN1224min rules (65% MetaCyc mapping)."
- [readme] rules file and coreactants invocation: [evidence span withheld — anchor status: external_doc]
