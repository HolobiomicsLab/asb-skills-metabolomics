---
name: thermodynamics-filtering-of-generated-reactions
description: Use when a rule-based network expansion (e.g. Pickaxe over EcoCyc with
  JN1224min rules) yields more reactions than needed and thermodynamic plausibility
  should be used to prune physiologically infeasible reactions before metabolomics
  peak annotation.
when_to_use_negative:
- When reactions lack a computable Gibbs free energy (eQuilibrator returns nothing
  outside its prediction scope) — these cannot be judged and are retained by design
- When the goal is peak annotation without thermodynamic pruning, since Figure 6 shows
  thermodynamics plus metabolomics matches a similar number of unannotated peaks as
  metabolomics alone, so it adds no annotation gain in that setting
- When pruning by reaction feasibility (DeepRFC) is intended instead — that is a separate
  filter with its own skill
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: eQuilibrator
  role: Computes the physiological Gibbs free energy used to decide which generated
    reactions exceed the 50 kJ/mol cutoff
  repo: https://equilibrator.readthedocs.io/en/latest/
- name: Pickaxe
  role: Generates the novel reaction network (stored in MongoDB) that the thermodynamics
    filter prunes
  repo: https://github.com/tyo-nu/MINE-Database
- name: PyMongo
  role: Queries the Mongo database to count retained compounds and compound peak pairs
    per filter combination
  repo: https://pymongo.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:76bd879327396fbbde77fad8641190d274ebfd671a7b020d564f200c25f86ef0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepRFC
  - Pickaxe
  - PyMongo
  - eQuilibrator
  - minedatabase
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# thermodynamics-filtering-of-generated-reactions

## Summary

Prunes a Pickaxe-generated reaction network by computing physiological Gibbs free energies with eQuilibrator and removing reactions whose ΔG′ exceeds 50 kJ/mol, while retaining reactions with no computable Gibbs energy.

## When to use

Use when a rule-based network expansion (e.g. Pickaxe over EcoCyc with JN1224min rules) yields more reactions than needed and thermodynamic plausibility should be used to prune physiologically infeasible reactions before metabolomics peak annotation.

## When NOT to use

- When reactions lack a computable Gibbs free energy (eQuilibrator returns nothing outside its prediction scope) — these cannot be judged and are retained by design
- When the goal is peak annotation without thermodynamic pruning, since Figure 6 shows thermodynamics plus metabolomics matches a similar number of unannotated peaks as metabolomics alone, so it adds no annotation gain in that setting
- When pruning by reaction feasibility (DeepRFC) is intended instead — that is a separate filter with its own skill

## Inputs

- Pickaxe-expanded reaction network (compounds and reactions stored in MongoDB, e.g. from an EcoCyc two-generation expansion with JN1224min rules)
- Reaction stoichiometries/identifiers suitable for eQuilibrator physiological Gibbs energy calculation
- Optional metabolomics filter input: list of unannotated peak m/z values (e.g. 200 peaks, m/z 150–300 Da from an E. coli untargeted metabolomics dataset)

## Outputs

- Filtered network in MongoDB with thermodynamically infeasible reactions (physiological ΔG′ > 50 kJ/mol) removed
- Per-filter-combination counts of generated compounds and compound peak pairs (e.g. figure6_peak_annotations.csv)
- Reproduced comparison plot of peak annotations (e.g. figure6_annotation_plot.png)

## How to apply

After Pickaxe expands the starting compounds (e.g. EcoCyc compounds <600 Da over two generations with the top 50 JN1224min rules) and stores compounds/reactions in MongoDB, compute the physiological Gibbs free energy of each generated reaction with eQuilibrator (equilibrator-api; the paper repo provides get_physiological_dgs.py to speed this up). Remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol — this is the paper's chosen cutoff. Critically, reactions for which eQuilibrator cannot return a Gibbs energy (outside its prediction scope) are left in the network, since the filter cannot comment on their feasibility. The filter can be combined with other Pickaxe filters such as the metabolomics filter (keep compounds whose m/z matches unannotated peaks, e.g. 200 peaks with m/z 150–300 Da); in the Figure 6 E. coli case, the thermodynamics-plus-metabolomics combination annotated a similar number of unannotated peaks as the metabolomics filter alone. Verify counts by querying the Mongo database with PyMongo via the analysis notebook and writing per-combination counts to CSV.

## Related tools

- **eQuilibrator** (Computes the physiological Gibbs free energy used to decide which generated reactions exceed the 50 kJ/mol cutoff) — https://equilibrator.readthedocs.io/en/latest/
- **Pickaxe** (Generates the novel reaction network (stored in MongoDB) that the thermodynamics filter prunes) — https://github.com/tyo-nu/MINE-Database
- **PyMongo** (Queries the Mongo database to count retained compounds and compound peak pairs per filter combination) — https://pymongo.readthedocs.io/en/stable/

## Examples

```
conda install -c condaforge equilibrator-api
```

## Evaluation signals

- Every removed reaction has a computable physiological Gibbs free energy greater than 50 kJ/mol; reactions without a computable Gibbs energy remain in the network
- Filtered network has fewer reactions than the unfiltered expansion (e.g. EcoCyc two-generation expansion baseline of 1,388,653 reactions)
- Combined metabolomics + thermodynamics annotation counts are similar to metabolomics-only counts, matching the paper's reported finding
- Per-combination counts written to the CSV table qualitatively reproduce the paper's Figure 6 comparison
- All Gibbs energies were computed with eQuilibrator (equilibrator-api) rather than assumed

## Limitations

- eQuilibrator does not return a Gibbs free energy for reactions outside the scope of its predictions, so such reactions cannot be filtered and stay in the network
- The 50 kJ/mol physiological Gibbs energy cutoff is the paper's specific threshold and would need justification if reused in another setting
- Run performance depends on MongoDB availability; the paper repo notes analysis can be sped up with helper scripts like get_physiological_dgs.py

## Evidence

- [results] Thermodynamics filter: remove reactions with physiological Gibbs free energy greater than 50 kJ/mol: "The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol."
- [results] Reactions without computable Gibbs energy are retained: "These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility."
- [results] eQuilibrator may not return a Gibbs energy for out-of-scope reactions: "it is possible that eQuilibrator does not return a Gibbs free energy for a reaction, as the reaction falls outside the scope of its predictions"
- [abstract] Pickaxe ships thermodynamics as one of its on-the-fly filters: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [readme] Thermodynamics filter requires eQuilibrator installation: "The thermodynamics filters require the use of [eQuilibrator](https://equilibrator.readthedocs.io/en/latest/). This can be installed via conda"
- [readme] Helper script to speed up physiological Gibbs energy computation: "The Analysis code can be sped up using `feasibility_first_gen.py`, `feasibility_second_gen_gen2.py`, and `get_physiological_dgs.py`"
