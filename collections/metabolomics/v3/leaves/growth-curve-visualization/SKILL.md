---
name: growth-curve-visualization
description: Use when a multi-generation reaction-network expansion must be characterized
  by exponential compound/reaction growth per generation and by runtime benchmarks
  that identify when expansion becomes computationally intractable.
when_to_use_negative:
- Single-generation expansions (e.g., the KEGG/EcoCyc/YMDB one-generation comparisons)
  with no growth-over-generations data to plot
- Analyses whose output is filtered-network quality (similarity sampling, metabolomics
  m/z matching, thermodynamics) rather than scaling/runtime behavior
- Small, hand-curated networks where expansion cost is trivial and runtime benchmarking
  adds no information
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: Iteratively applies reaction rules to a compound set to generate novel compounds
    and reactions per generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package (installed via pip install minedatabase) providing the Pickaxe
    engine used by the generation script
  repo: https://pypi.org/project/minedatabase/
- name: MongoDB / PyMongo
  role: Stores generated compounds and reactions; paper reported 12 million compounds
    inserted in ~9 hours
  repo: https://github.com/tyo-nu/MINE-Database
- name: benchmarks.ipynb / fig3_plot.ipynb
  role: Notebooks in tyo-nu/pickaxe_paper that measure per-generation runtime and
    render the growth curves
  repo: https://github.com/tyo-nu/pickaxe_paper
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:278b9dc76f9a6cdab3ea7e43b1cf7ff1e21eeef97ec44750da8165ef55494fe5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MongoDB / PyMongo
  - Pickaxe
  - benchmarks.ipynb / fig3_plot.ipynb
  - minedatabase
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# growth-curve-visualization

## Summary

Reproduces the Figure 3 exponential growth and runtime benchmark analysis from the Pickaxe paper: iterative reaction-rule network expansion with per-generation compound/reaction counts, runtime measurement, and growth-curve plots.

## When to use

Use when a multi-generation reaction-network expansion must be characterized by exponential compound/reaction growth per generation and by runtime benchmarks that identify when expansion becomes computationally intractable.

## When NOT to use

- Single-generation expansions (e.g., the KEGG/EcoCyc/YMDB one-generation comparisons) with no growth-over-generations data to plot
- Analyses whose output is filtered-network quality (similarity sampling, metabolomics m/z matching, thermodynamics) rather than scaling/runtime behavior
- Small, hand-curated networks where expansion cost is trivial and runtime benchmarking adds no information

## Inputs

- Starting compound set (e.g., from YMDB, 1,035 starting compounds in the paper's YMDB expansion)
- Reaction ruleset (100-rule subset of the JN1224min MetaCyc-derived rules)
- Molecular-weight/memory cutoff (<300) governing which generated compounds are retained between generations
- Local or remote mongo database for storing generated compounds and reactions

## Outputs

- Per-generation compound and reaction counts for each expansion generation
- Measured Pickaxe runtime per expansion generation
- Consolidated growth/runtime table in CSV format
- Figure 3 panels: exponential growth curves of compounds/reactions and runtime benchmark plots

## How to apply

Run the exponential-growth generation script fig3_gen_exp_growth/run_files/mem_lt300_100r.py, which expands a compound set with a 100-rule ruleset under a molecular-weight/memory cutoff below 300 using the minedatabase package's Pickaxe, recording unique compound and reaction counts for each generation. Execute the runtime benchmarks in fig3_gen_exp_growth/analysis/benchmarks.ipynb to measure Pickaxe generation runtime per expansion generation (in the paper's YMDB case, two generations produced 12 million new compounds inserted into a mongo database in approximately nine hours). Consolidate the per-generation counts and measured runtimes into a single CSV data table. Render the exponential growth curves and runtime plots with fig3_gen_exp_growth/analysis/fig3_plot.ipynb; correct output shows compounds and reactions growing exponentially across generations with runtimes that grow accordingly.

## Related tools

- **Pickaxe** (Iteratively applies reaction rules to a compound set to generate novel compounds and reactions per generation) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package (installed via pip install minedatabase) providing the Pickaxe engine used by the generation script) — https://pypi.org/project/minedatabase/
- **MongoDB / PyMongo** (Stores generated compounds and reactions; paper reported 12 million compounds inserted in ~9 hours) — https://github.com/tyo-nu/MINE-Database
- **benchmarks.ipynb / fig3_plot.ipynb** (Notebooks in tyo-nu/pickaxe_paper that measure per-generation runtime and render the growth curves) — https://github.com/tyo-nu/pickaxe_paper

## Examples

```
python fig3_gen_exp_growth/run_files/mem_lt300_100r.py
```

## Evaluation signals

- Compound counts increase exponentially (multiplicatively) with each expansion generation, as in the reported two-generation YMDB run yielding 12 million new compounds
- Reaction counts grow alongside compound counts per generation (paper-scale runs reach millions of reactions)
- Runtime per generation from benchmarks.ipynb rises with generation size, delineating where expansion becomes intractable
- Reproduced Figure 3 plots qualitatively match the paper's exponential growth and runtime trends
- CSV table contains one row per generation with both compound/reaction counts and measured runtime

## Limitations

- Exponential growth makes multi-generation expansions computationally intractable; cutoffs such as the <300 molecular-weight/memory limit are needed to bound each generation
- Runtime performance benchmarks were referenced as a supplementary file whose content was not fully included in the paper
- Storing large runs requires a mongo database installation, adding infrastructure overhead
- No software version numbers were stated for the minedatabase package used in the paper's runs

## Evidence

- [readme] Exponential growth script location: "Generation code for exponential growth is found in `fig3_gen_exp_growth/run_files/mem_lt300_100r.py`"
- [readme] Runtime and plotting notebook locations: "Generation code for runtime is found in `fig3_gen_exp_growth/analysis/benchmarks.ipynb`"
- [results] Exponential growth finding: [evidence span withheld — anchor status: not_found]
- [results] Runtime scale of the expansion: "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
