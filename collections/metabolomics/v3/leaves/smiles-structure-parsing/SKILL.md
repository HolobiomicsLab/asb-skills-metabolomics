---
name: smiles-structure-parsing
description: Use when a starting-compound file (id + SMILES columns) must be pruned
  by structural properties before Pickaxe network expansion, e.g. enforcing the paper's
  MW < 600 g/mol and carbon < 12 (non-cofactor) cutoffs.
when_to_use_negative:
- The task requires reaction-level rather than compound-level pruning, e.g. thermodynamics
  (physiological Gibbs free energy > 50 kJ/mol), metabolomics (m/z matching of unannotated
  peaks), or chemical-similarity filters.
- The input compounds are already filtered to the target property range, so re-applying
  the property filters would only add overhead.
- The input lacks SMILES structures (only names, IDs, or m/z values), so molecular
  weight and element counts cannot be computed from structure.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: Pickaxe
  role: Consumes the compound property filters to prune the expanding reaction network
    on-the-fly
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package (v2.0.0 on PyPI) whose Pickaxe module provides the on-the-fly
    filter interface the property filters plug into
  repo: https://pypi.org/project/minedatabase/2.0.0/
- name: pandas
  role: Loads the starting-compound CSV (id + SMILES columns) and writes the filtered_compounds.csv
    flag table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:22ee9a996936bf1bfa2a007d1aeeb667b147ece8a249822fad7038719b0018ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - Pickaxe
  - minedatabase
  - pandas
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# SMILES structure parsing for compound property filtering

## Summary

Parse SMILES strings from a starting-compound CSV and compute per-compound structural properties (molecular weight and per-element atom counts) to implement Pickaxe's compound property filters, which prune the reaction network on-the-fly during expansion.

## When to use

Use when a starting-compound file (id + SMILES columns) must be pruned by structural properties before Pickaxe network expansion, e.g. enforcing the paper's MW < 600 g/mol and carbon < 12 (non-cofactor) cutoffs.

## When NOT to use

- The task requires reaction-level rather than compound-level pruning, e.g. thermodynamics (physiological Gibbs free energy > 50 kJ/mol), metabolomics (m/z matching of unannotated peaks), or chemical-similarity filters.
- The input compounds are already filtered to the target property range, so re-applying the property filters would only add overhead.
- The input lacks SMILES structures (only names, IDs, or m/z values), so molecular weight and element counts cannot be computed from structure.

## Inputs

- Starting-compound CSV with id and SMILES columns
- Molecular-weight range threshold (paper value: keep MW < 600 g/mol)
- Atomic-composition range specification (paper value: carbon < 12 for non-cofactor molecules)
- Cofactor list used to exempt cofactors from the carbon-count limit

## Outputs

- filtered_compounds.csv with kept/removed flags per compound for the molecular-weight filter and the atomic-composition filter
- Python filter module compatible with Pickaxe's on-the-fly pruning filter interface in minedatabase

## How to apply

Load the starting-compound CSV (id + SMILES columns) with pandas in Python, then derive each compound's molecular weight from its SMILES structure. Apply the molecular-weight range filter using the paper's threshold of retaining compounds with molecular weights less than 600 g/mol, the same cutoff used when Pickaxe filtered KEGG (12,731 compounds), EcoCyc, and YMDB before expansion. Compute per-element atom counts for each compound and apply the atomic-composition filter, limiting carbon atoms in non-cofactor molecules to fewer than twelve. Package the two tests as a Python module compatible with Pickaxe's on-the-fly network-pruning filter interface in the minedatabase package. Write a CSV flagging each compound as kept or removed under each filter (filtered_compounds.csv) so every pruning decision is auditable.

## Related tools

- **Pickaxe** (Consumes the compound property filters to prune the expanding reaction network on-the-fly) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package (v2.0.0 on PyPI) whose Pickaxe module provides the on-the-fly filter interface the property filters plug into) — https://pypi.org/project/minedatabase/2.0.0/
- **pandas** (Loads the starting-compound CSV (id + SMILES columns) and writes the filtered_compounds.csv flag table)

## Examples

```
import pandas as pd; df = pd.read_csv('starting_compounds.csv'); # compute MW and element counts from SMILES, keep MW < 600 g/mol and C < 12 (non-cofactors); df.to_csv('filtered_compounds.csv', index=False)
```

## Evaluation signals

- filtered_compounds.csv contains a kept/removed decision for every input compound under both filters.
- Every compound flagged removed violates a stated threshold: MW >= 600 g/mol, or carbon count >= 12 for non-cofactors.
- Cofactor molecules are exempt from the carbon-count limit, matching the paper's non-cofactor qualifier.
- Kept compounds all fall within the filter ranges (MW < 600 g/mol, C < 12 for non-cofactors), so no compound that should pass is removed.
- The filter module runs through Pickaxe's on-the-fly pruning interface in minedatabase without schema errors.

## Limitations

- The thresholds (600 g/mol, fewer than twelve carbons in non-cofactors) are specific to the paper's KEGG/EcoCyc/YMDB expansion setup; other applications must re-derive appropriate ranges.
- Pickaxe software version numbers are not stated in the article or README (only Python 3.7+ is specified), so filter-interface compatibility must be checked against the installed minedatabase release.
- Property filters prune compounds before expansion and cannot comment on downstream reaction feasibility, e.g. reactions for which eQuilibrator returns no Gibbs free energy are left in the network.

## Evidence

- [other] MW range / atomic composition ranges: [evidence span withheld — anchor status: not_found]
- [results] MW cutoff applied to databases: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [results] carbon-count filter for non-cofactors: "an atomic composition filter was used to limit the number of carbon atoms contained in non-cofactor molecules to fewer than twelve"
- [readme] Pickaxe applies rules to user-specified compounds: [evidence span withheld — anchor status: external_doc]
