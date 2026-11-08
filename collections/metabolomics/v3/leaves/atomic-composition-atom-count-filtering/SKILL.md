---
name: atomic-composition-atom-count-filtering
description: Use when expanding a reaction network with Pickaxe/minedatabase and you
  need to prune generated compounds by per-element atom counts (e.g. carbon count
  caps for non-cofactors) before further generations.
when_to_use_negative:
- You need thermodynamic or reaction-feasibility pruning — Pickaxe's eQuilibrator-based
  thermodynamics filter (remove reactions with physiological Gibbs free energy > 50
  kJ/mol) or DeepRFC feasibility filter is the right mechanism, not atom counts
- The goal is prioritizing expansion toward known targets — use Pickaxe's similarity
  cutoff or similarity sampling filters instead, which score compounds against target
  molecules
- Input is unannotated metabolomics peaks — the metabolomics filter (m/z matching,
  e.g. peaks between 150 and 300 Da) is the appropriate filter, not composition ranges
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: Reaction-rule network generator whose on-the-fly filter interface applies
    the atomic-composition filter during expansion
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: PyPI package providing the Pickaxe filter framework into which the composition-filter
    module is packaged
  repo: https://pypi.org/project/minedatabase/
- name: Python
  role: Language for computing atom counts from SMILES (with pandas for CSV I/O) and
    implementing the filter (Python 3.7 or higher)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:417bdce04f99d88d4bc6864e4d57934d02aae899d31529fecd7e53e3a2c81823
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
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Atomic-composition atom-count filtering

## Summary

Implements Pickaxe's compound property filter that prunes a reaction network on-the-fly by atom counts, e.g. limiting carbon atoms in non-cofactor molecules to fewer than twelve. This caps network explosion during expansion by discarding compounds whose per-element composition exceeds user-specified ranges.

## When to use

Use when expanding a reaction network with Pickaxe/minedatabase and you need to prune generated compounds by per-element atom counts (e.g. carbon count caps for non-cofactors) before further generations.

## When NOT to use

- You need thermodynamic or reaction-feasibility pruning — Pickaxe's eQuilibrator-based thermodynamics filter (remove reactions with physiological Gibbs free energy > 50 kJ/mol) or DeepRFC feasibility filter is the right mechanism, not atom counts
- The goal is prioritizing expansion toward known targets — use Pickaxe's similarity cutoff or similarity sampling filters instead, which score compounds against target molecules
- Input is unannotated metabolomics peaks — the metabolomics filter (m/z matching, e.g. peaks between 150 and 300 Da) is the appropriate filter, not composition ranges

## Inputs

- Starting-compound CSV with id and SMILES columns (e.g. iML1515 E. coli GEM compounds, KEGG/EcoCyc/YMDB compound lists)
- Cofactor list identifying compounds exempt from atom-count caps
- Per-element atom-count ranges (e.g. carbon < 12 for non-cofactors)

## Outputs

- filtered_compounds.csv flagging each compound as kept or removed under each filter
- A Python filter module compatible with Pickaxe's on-the-fly pruning interface

## How to apply

Load the starting-compound CSV (id + SMILES columns) with pandas in Python, then compute per-element atom counts for each compound from its SMILES structure. Apply the atomic-composition filter by comparing each element's count against a specified range — in the paper's KEGG/EcoCyc/YMDB expansions, non-cofactor molecules were limited to fewer than twelve carbon atoms. Cofactors are exempt from the cap, so maintain a cofactor list and skip composition checks for those compounds. Package the check as a Python module compatible with Pickaxe's on-the-fly network-pruning filter interface within the minedatabase package, so pruning happens during expansion rather than as post-processing. Optionally pair it with the molecular-weight range filter (compounds with MW less than 600 g/mol retained), which the paper applied alongside the carbon cap to the same databases.

## Related tools

- **Pickaxe** (Reaction-rule network generator whose on-the-fly filter interface applies the atomic-composition filter during expansion) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (PyPI package providing the Pickaxe filter framework into which the composition-filter module is packaged) — https://pypi.org/project/minedatabase/
- **Python** (Language for computing atom counts from SMILES (with pandas for CSV I/O) and implementing the filter (Python 3.7 or higher))

## Examples

```
df = pandas.read_csv('starting_compounds.csv'); df['C'] = df['SMILES'].map(lambda s: Chem.MolFromSmiles(s).GetNumAtoms())
```

## Evaluation signals

- filtered_compounds.csv contains a kept/removed decision for every input compound under both the molecular-weight and atomic-composition filters
- Every compound flagged as removed violates a stated threshold: MW >= 600 g/mol, or carbon count >= 12 for non-cofactor molecules
- Cofactor-listed compounds with >= 12 carbons are kept, confirming the non-cofactor exemption is honored
- Filter module runs within Pickaxe's on-the-fly pruning loop and reduces compound counts between expansion generations (paper's one-generation KEGG expansion produced 9,562,940 compounds without such caps; EcoCyc/YMDB expansions used the carbon-12 cap)

## Limitations

- The threshold values (carbon < 12, MW < 600 g/mol) are application-specific choices from the paper's database-expansion examples; other use cases may need different ranges
- Filters silently drop compounds, so the novelty/exploration scope of the network is constrained — reactions involving oversized compounds will never be generated
- The atomic-composition filter requires distinguishing cofactors from non-cofactors; an incomplete cofactor list will incorrectly prune legitimate coreactants

## Evidence

- [results] Atomic composition filter: limit carbon atoms in non-cofactor molecules to fewer than twelve: "an atomic composition filter was used to limit the number of carbon atoms contained in non-cofactor molecules to fewer than twelve"
- [results] MW range filter: "Each database was filtered to only contain compounds whose molecular weights are less than 600 g/mol."
- [abstract] Filters operate on-the-fly: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [discussion] Pickaxe available as package: "Pickaxe is available as a pypi package at https://pypi.org/project/minedatabase/"
- [discussion] Starting compounds must be specified: "A network expansion requires the specification of starting compounds and reaction rules"
