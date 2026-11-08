---
name: computational-runtime-profiling
description: Use when you need to quantify whether an iterative reaction-rule network
  expansion (e.g., Pickaxe generations) grows exponentially in compounds/reactions
  and when its runtime or memory cost becomes prohibitive.
when_to_use_negative:
- Your goal is a single-generation expansion with curated rules (e.g., top 500 JN1224min
  rules for one generation) rather than characterizing multi-generation growth behavior
- The network is already filtered/pruned (e.g., similarity sampling filter selecting
  2000 compounds), since pruning deliberately caps the exponential growth this skill
  is meant to measure
- You only need final network contents stored in a mongo database, not per-generation
  scaling or timing data
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: Reaction-rule-based network expansion engine executed iteratively per generation
    while compound/reaction counts and runtimes are recorded
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package providing the Pickaxe API used by the generation script (mem_lt300_100r.py)
    and benchmark notebooks
  repo: https://pypi.org/project/minedatabase/
- name: MongoDB
  role: Storage backend into which expanded compounds and reactions are inserted;
    insertion throughput (e.g., 12 million compounds in ~9 hours) is part of the runtime
    profile
  repo: https://www.mongodb.com/try/download/community
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:d08037c8104e3a55ca68be2fea4327535ef625e9afb0ab14ceea164f52bff6eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MongoDB
  - Pickaxe
  - minedatabase
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# computational-runtime-profiling

## Summary

Profiles how network-expansion runtime and output size scale with each expansion generation, using Pickaxe (via the minedatabase package) to expand a compound set iteratively and benchmarking the compute cost per generation until expansion becomes intractable.

## When to use

Use when you need to quantify whether an iterative reaction-rule network expansion (e.g., Pickaxe generations) grows exponentially in compounds/reactions and when its runtime or memory cost becomes prohibitive.

## When NOT to use

- Your goal is a single-generation expansion with curated rules (e.g., top 500 JN1224min rules for one generation) rather than characterizing multi-generation growth behavior
- The network is already filtered/pruned (e.g., similarity sampling filter selecting 2000 compounds), since pruning deliberately caps the exponential growth this skill is meant to measure
- You only need final network contents stored in a mongo database, not per-generation scaling or timing data

## Inputs

- Starting compound set (e.g., YMDB compounds, optionally filtered to molecular weight < 300 g/mol)
- Reaction ruleset (e.g., a 100-rule subset of the JN1224min ruleset derived from MetaCyc)
- Number of expansion generations to run (e.g., two generations)
- Benchmarking configuration for the runtime notebook (benchmarks.ipynb)

## Outputs

- Per-generation counts of unique compounds and reactions (growth data table)
- Measured Pickaxe runtime per expansion generation (benchmark data)
- Consolidated CSV growth/runtime data table
- Figure 3-style plots: exponential growth curves of compounds/reactions and runtime benchmark panels

## How to apply

Run a multi-generation expansion script, such as fig3_gen_exp_growth/run_files/mem_lt300_100r.py, which expands a starting compound set with a 100-rule ruleset under a molecular-weight cutoff below 300 g/mol, recording unique compound and reaction counts after each generation. In parallel, execute the runtime benchmark notebook fig3_gen_exp_growth/analysis/benchmarks.ipynb to measure Pickaxe generation runtime per expansion generation. Consolidate per-generation compound/reaction counts and measured runtimes into a single CSV growth/runtime table. Finally, render exponential growth curves and runtime benchmark plots with fig3_gen_exp_growth/analysis/fig3_plot.ipynb to reproduce the Figure 3 panels. The rationale: Pickaxe applies reaction rules iteratively, so compound counts compound multiplicatively each generation; benchmarking per generation reveals at which generation the mongo database insertion (e.g., 12 million compounds in ~9 hours in the YMDB case) becomes the bottleneck.

## Related tools

- **Pickaxe** (Reaction-rule-based network expansion engine executed iteratively per generation while compound/reaction counts and runtimes are recorded) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package providing the Pickaxe API used by the generation script (mem_lt300_100r.py) and benchmark notebooks) — https://pypi.org/project/minedatabase/
- **MongoDB** (Storage backend into which expanded compounds and reactions are inserted; insertion throughput (e.g., 12 million compounds in ~9 hours) is part of the runtime profile) — https://www.mongodb.com/try/download/community

## Examples

```
python -m fig3_gen_exp_growth.run_files.mem_lt300_100r  # then run fig3_gen_exp_growth/analysis/benchmarks.ipynb and fig3_plot.ipynb
```

## Evaluation signals

- Reproduced plots must show exponential (not linear) growth of unique compounds and reactions across expansion generations, matching the qualitative trends of Figure 3
- Per-generation runtime values must be consistent with the measurements produced by benchmarks.ipynb (runtime increases with generation as the compound pool grows)
- The reported YMDB multi-generation case should be reproducible in magnitude: two generations yielding ~12 million new compounds
- Counts recorded per generation must be cumulative unique compounds/reactions, not raw generation outputs, and the consolidated CSV must align row-for-row with the plotted points

## Limitations

- Runtime performance benchmark details were referenced as a supplementary file (Additional file 1: Pickaxe runtime performance benchmarks) whose content is not included in the article text
- No software version numbers were stated for Pickaxe or the minedatabase package (only a Python >= 3.7 requirement), so exact reproducibility of timing figures may vary
- Unbounded expansion is computationally intractable: in the reported case, 12 million new compounds required approximately nine hours to generate and insert into a mongo database, so memory/runtime cutoffs (e.g., MW < 300) are needed to keep runs feasible

## Evidence

- [results] Exponential growth of expansion: "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
- [abstract] Per-generation iterative expansion: "This software iteratively applies reaction rules to a set of metabolites to generate novel reactions."
- [readme] Repository structure for Figure 3 profiling: "Generation code for exponential growth is found in `fig3_gen_exp_growth/run_files/mem_lt300_100r.py`"
- [readme] Runtime benchmark location: "Generation code for runtime is found in `fig3_gen_exp_growth/analysis/benchmarks.ipynb`"
- [discussion] Missing benchmark supplementary content: "Additional file 1. Pickaxe runtime performance benchmarks."
