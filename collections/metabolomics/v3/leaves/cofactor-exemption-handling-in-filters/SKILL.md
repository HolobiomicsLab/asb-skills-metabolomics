---
name: cofactor-exemption-handling-in-filters
description: Use when implementing or re-running Pickaxe compound property filters
  (molecular weight range, atomic composition) where cofactor/coreactant entries in
  the starting-compound list must bypass the thresholds applied to all other compounds.
when_to_use_negative:
- When the compound list contains no designated cofactors or coreactants — a plain
  range filter without exemptions is sufficient
- When the goal is to filter reactions rather than compounds (e.g. the thermodynamics
  filter removing reactions with physiological Gibbs free energy greater than 50 kJ/mol)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Pickaxe
  role: Network-expansion tool whose on-the-fly filter interface consumes the property
    filters; cofactors are specified as part of a Pickaxe run
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package (pip install minedatabase) in which the filter module is packaged
    and which runs Pickaxe
  repo: https://pypi.org/project/minedatabase/
- name: Python
  role: pandas-based computation of molecular weights and per-element atom counts
    from SMILES, with requires Python 3.7 or higher
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:2c6d8ab11b450d4b9e7ac3f7ab797e04284afdc38856699041035f0964e2068b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - Pickaxe
  - Python
  - minedatabase
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# cofactor-exemption-handling-in-filters

## Summary

Applies Pickaxe compound property filters while exempting designated cofactors from property thresholds, e.g. the atomic composition filter that limits non-cofactor molecules to fewer than twelve carbon atoms. This prevents ubiquitous small-molecule coreactants from being pruned simply because they violate size constraints meant for novel compounds.

## When to use

Use when implementing or re-running Pickaxe compound property filters (molecular weight range, atomic composition) where cofactor/coreactant entries in the starting-compound list must bypass the thresholds applied to all other compounds.

## When NOT to use

- When the compound list contains no designated cofactors or coreactants — a plain range filter without exemptions is sufficient
- When the goal is to filter reactions rather than compounds (e.g. the thermodynamics filter removing reactions with physiological Gibbs free energy greater than 50 kJ/mol)

## Inputs

- Starting-compound CSV with id and SMILES columns
- Cofactor/coreactant TSV used by Pickaxe (e.g. metacyc_coreactants.tsv)
- Generalized reaction-rules TSV (e.g. metacyc_generalized_rules.tsv)

## Outputs

- filtered_compounds.csv with kept/removed flags per compound under each filter
- Python filter module compatible with Pickaxe's on-the-fly network-pruning filter interface (minedatabase package)

## How to apply

Load the starting-compound CSV (id + SMILES columns) with pandas and compute each compound's molecular weight and per-element atom counts. Apply the molecular-weight filter with the paper's threshold of retaining compounds below 600 g/mol, and the atomic-composition filter limiting carbon atoms to fewer than twelve. Before flagging a compound as removed, check it against the cofactor/coreactant list supplied to the Pickaxe run (e.g. the coreactants TSV such as metacyc_coreactants.tsv, alongside the generalized rules file); compounds matching a cofactor are exempted from the carbon-count threshold and kept regardless. Package the result as a Python module compatible with Pickaxe's on-the-fly network-pruning filter interface in the minedatabase package. The rationale is that cofactors participate as coreactants in rule expansion, so pruning them by property ranges would break network generation rather than shrink it meaningfully.

## Related tools

- **Pickaxe** (Network-expansion tool whose on-the-fly filter interface consumes the property filters; cofactors are specified as part of a Pickaxe run) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package (pip install minedatabase) in which the filter module is packaged and which runs Pickaxe) — https://pypi.org/project/minedatabase/
- **Python** (pandas-based computation of molecular weights and per-element atom counts from SMILES, with requires Python 3.7 or higher)

## Examples

```
python pickaxe.py -C ./data/metacyc_generalized_rules.tsv -r ./data metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- filtered_compounds.csv contains a kept/removed decision for every input compound under both filters
- Every removed compound violates a stated threshold (MW >= 600 g/mol, or carbon count >= 12 for non-cofactors)
- No compound flagged as a cofactor/coreactant is removed by the atomic composition filter even if it has 12 or more carbons
- Filter module imports and conforms to Pickaxe's on-the-fly network-pruning filter interface within minedatabase

## Limitations

- Thresholds are run-specific: the <600 g/mol and <12 carbon (non-cofactor) cutoffs come from the paper's example networks, not defaults for all expansions
- The filter requires an explicit cofactor/coreactant list; if a coreactant is missing from that list it will be incorrectly pruned, and Pickaxe run options (compounds, rules, cofactors) must be fully specified

## Evidence

- [results] Atomic composition filter: limit carbon atoms in non-cofactor molecules to fewer than twelve: "an atomic composition filter was used to limit the number of carbon atoms contained in non-cofactor molecules to fewer than twelve"
- [readme] Cofactors specified alongside compounds and rules in a Pickaxe run: [evidence span withheld — anchor status: external_doc]
- [results] Molecular weight threshold used for database filtering: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
