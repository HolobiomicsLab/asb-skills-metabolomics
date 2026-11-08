---
name: compound-molecular-weight-filtering
description: Use when preparing starting-compound sets (e.g. KEGG, EcoCyc, YMDB SMILES/CSV)
  for a Pickaxe/MINE-Database expansion and you need to constrain network size by
  compound molecular weight before rule application.
when_to_use_negative:
- You need to control network size around specific target compounds rather than by
  bulk compound size — use the similarity cutoff or similarity sampling filter (e.g.
  sampling 2000 compounds from similarity-score distributions) instead
- The starting compounds are already restricted (e.g. the fig3 exponential-growth
  script uses a memory/MW cutoff below 300) and re-filtering would double-prune and
  bias the benchmark
- You want to filter generated reactions by thermodynamic feasibility (physiological
  ΔG > 50 kJ/mol via eQuilibrator) — that is a reaction-level filter, not a compound
  property filter
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0092
tools:
- name: Pickaxe
  role: Applies the compound property (molecular-weight range) filter on-the-fly to
    prune compounds before each expansion generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Hosts the Pickaxe filter interface into which the MW-range filter module is
    packaged; results stored via PyMongo/MongoDB
  repo: https://github.com/tyo-nu/MINE-Database
- name: Python
  role: Used with pandas to compute molecular weights from SMILES and apply the MW
    < 600 g/mol threshold to the compound sets
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:83d19c2fc0630cfb878f294304d7b41ad84618dc1db6e685e58148db4e6b7962
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - Pickaxe
  - Python
  - minedatabase
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# compound-molecular-weight-filtering

## Summary

Prunes a Pickaxe expansion network by retaining only compounds within a specified molecular-weight range (e.g. < 600 g/mol), applied on-the-fly before each rule-application generation so exponential compound growth is curtailed.

## When to use

Use when preparing starting-compound sets (e.g. KEGG, EcoCyc, YMDB SMILES/CSV) for a Pickaxe/MINE-Database expansion and you need to constrain network size by compound molecular weight before rule application.

## When NOT to use

- You need to control network size around specific target compounds rather than by bulk compound size — use the similarity cutoff or similarity sampling filter (e.g. sampling 2000 compounds from similarity-score distributions) instead
- The starting compounds are already restricted (e.g. the fig3 exponential-growth script uses a memory/MW cutoff below 300) and re-filtering would double-prune and bias the benchmark
- You want to filter generated reactions by thermodynamic feasibility (physiological ΔG > 50 kJ/mol via eQuilibrator) — that is a reaction-level filter, not a compound property filter

## Inputs

- Starting-compound CSV with id and SMILES columns (e.g. KEGG Release 99.0, EcoCyc 25.1, or YMDB 2.0 compound sets)
- Molecular-weight range specification for the filter (e.g. MW < 600 g/mol)
- Optional per-element atom-count ranges for the atomic-composition filter (e.g. carbon < 12 for non-cofactors), plus a cofactor exclusion list

## Outputs

- filtered_compounds.csv flagging each compound as kept or removed under each filter
- A filtered compound set suitable as Pickaxe expansion input (stored or consumed via the minedatabase filter interface)

## How to apply

Load the starting-compound CSV (id + SMILES columns) with pandas and compute each compound's molecular weight from its SMILES structure. Pickaxe's compound property filters operate by range specification: the molecular weight filter accepts a MW range that a compound must exist in, so flag each compound kept if MW < 600 g/mol (the paper's threshold for KEGG, EcoCyc, and YMDB expansions). Optionally combine with the atomic-composition filter, which limits carbon atoms in non-cofactor molecules to fewer than twelve. Package the filter as a Python module compatible with Pickaxe's on-the-fly network-pruning filter interface in the minedatabase package, since filters remove compounds before each expansion generation and thereby control the exponential growth (e.g. the unfiltered YMDB expansion yielded 575,262 compounds and 892,343 reactions from only 1,035 starting compounds). Write a per-compound kept/removed table (filtered_compounds.csv) for auditability.

## Related tools

- **Pickaxe** (Applies the compound property (molecular-weight range) filter on-the-fly to prune compounds before each expansion generation) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Hosts the Pickaxe filter interface into which the MW-range filter module is packaged; results stored via PyMongo/MongoDB) — https://github.com/tyo-nu/MINE-Database
- **Python** (Used with pandas to compute molecular weights from SMILES and apply the MW < 600 g/mol threshold to the compound sets)

## Examples

```
import pandas as pd; df = pd.read_csv('compounds.csv'); from rdkit import Chem; from rdkit.Chem.Descriptors import MolWt; df['MW'] = [MolWt(Chem.MolFromSmiles(s)) for s in df['SMILES']]; df['kept_mw_lt600'] = df['MW'] < 600; df.to_csv('filtered_compounds.csv', index=False)
```

## Evaluation signals

- filtered_compounds.csv contains a kept/removed decision for every input compound under both filters
- Every compound flagged removed violates a stated threshold: MW >= 600 g/mol, or carbon count >= 12 for non-cofactor molecules
- Expansion counts of the filtered set match reported baselines when the same ruleset is used (e.g. KEGG 12,731 starting compounds -> 9,562,940 compounds / 13,759,651 reactions; EcoCyc 1,923 -> 906,086 / 1,388,653; YMDB 1,035 -> 575,262 / 892,343)
- No compound in the kept set exceeds the MW range threshold, verifiable by recomputing MW from SMILES

## Limitations

- The filter is a hard range cutoff on compound properties; it cannot bias the network toward compounds similar to targets — that requires the similarity cutoff or sampling filter
- Reactions whose properties fall outside tool coverage (e.g. eQuilibrator returning no Gibbs free energy) may need to be retained, so property-based pruning is never exhaustive
- Growth remains exponential even with MW filtering: a two-generation YMDB expansion still generated 12 million new compounds, so additional pruning (cutoff/sampling filters) may be required for tractability

## Evidence

- [results] MW < 600 threshold applied to all three databases: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [results] Range-based operation of compound property filters: [evidence span withheld — anchor status: not_found]
- [results] Companion atomic-composition filter threshold: "an atomic composition filter was used to limit the number of carbon atoms contained in non-cofactor molecules to fewer than twelve"
- [readme] Filter placement in the Pickaxe expansion workflow: [evidence span withheld — anchor status: external_doc]
- [results] Unfiltered growth baseline motivating the filter: [evidence span withheld — anchor status: not_found]
