---
name: reaction-network-generation-benchmarking
description: Use when running multi-generation network expansions (e.g., 100-rule
  ruleset on YMDB) and you need to measure exponential compound/reaction growth and
  per-generation runtime to identify tractability limits.
when_to_use_negative:
- You only need a single-application network for a downstream purpose (e.g., metabolomics
  peak annotation or similarity-sampling analysis) rather than scaling/benchmark measurements.
- The target question is filtering quality (thermodynamics, metabolomics, similarity
  filters) rather than raw expansion size and runtime behavior.
- An expansion is already known to be tractable and pre-computed; benchmarking adds
  compute cost without new information.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Pickaxe
  role: Iteratively applies reaction rules to metabolites to generate novel compounds
    and reactions per expansion generation; runtime is measured per generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package providing Pickaxe and network generation used by the exponential-growth
    run script
  repo: https://pypi.org/project/minedatabase/
- name: MongoDB / PyMongo
  role: Storage backend for generated compounds and reactions during large expansions
    (12 million compounds in ~9 hours)
  repo: https://www.mongodb.com/try/download/community
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:e5178d5e54c238dd8bc03fa558635fe30427516989a6a941a5b7f46cfc70b238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MongoDB / PyMongo
  - Pickaxe
  - minedatabase
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# reaction-network-generation-benchmarking

## Summary

Quantify how a rule-based metabolic reaction network (Pickaxe/minedatabase) scales across expansion generations by recording per-generation compound and reaction counts and benchmarking runtime, exposing when promiscuous-enzyme expansion becomes computationally intractable.

## When to use

Use when running multi-generation network expansions (e.g., 100-rule ruleset on YMDB) and you need to measure exponential compound/reaction growth and per-generation runtime to identify tractability limits.

## When NOT to use

- You only need a single-application network for a downstream purpose (e.g., metabolomics peak annotation or similarity-sampling analysis) rather than scaling/benchmark measurements.
- The target question is filtering quality (thermodynamics, metabolomics, similarity filters) rather than raw expansion size and runtime behavior.
- An expansion is already known to be tractable and pre-computed; benchmarking adds compute cost without new information.

## Inputs

- Starting compound set (e.g., YMDB or KEGG compounds as a compound list/database file)
- Reaction ruleset (e.g., JN1224min-derived 100-rule or top-500-rule subset from MetaCyc)
- Molecular-weight cutoff parameter (e.g., compounds <300 g/mol for the growth experiment, <600 g/mol for database expansions)
- Number of expansion generations to run

## Outputs

- CSV table of per-generation compound counts, reaction counts, and measured runtimes
- Exponential growth curves (compounds and reactions vs. generation) and runtime benchmark plots reproducing Figure 3 panels
- Optionally, generated compounds/reactions inserted into a mongo database (via PyMongo)

## How to apply

Configure a Pickaxe expansion that applies a 100-rule ruleset to a starting compound set under a molecular-weight/memory cutoff below 300 g/mol, using the minedatabase package, and log compound and reaction counts after each expansion generation. Run the generation script (fig3_gen_exp_growth/run_files/mem_lt300_100r.py) and separately execute runtime benchmarks (fig3_gen_exp_growth/analysis/benchmarks.ipynb) that measure Pickaxe runtime per expansion generation. Consolidate per-generation counts and runtimes into a single CSV data table, then render exponential-growth curves and runtime benchmark plots (fig3_gen_exp_growth/analysis/fig3_plot.ipynb). Rationale: each generation applies rules to all compounds from the prior generation, so unique compound counts grow multiplicatively — the paper's multi-generation YMDB expansion produced two generations and 12 million new compounds, and even a single-generation KEGG expansion (12,731 starting compounds, <600 g/mol, top 500 JN1224min rules) yielded 9,562,940 compounds and 13,759,651 reactions.

## Related tools

- **Pickaxe** (Iteratively applies reaction rules to metabolites to generate novel compounds and reactions per expansion generation; runtime is measured per generation) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package providing Pickaxe and network generation used by the exponential-growth run script) — https://pypi.org/project/minedatabase/
- **MongoDB / PyMongo** (Storage backend for generated compounds and reactions during large expansions (12 million compounds in ~9 hours)) — https://www.mongodb.com/try/download/community

## Examples

```
python pickaxe.py -C ./data/metacyc_generalized_rules.tsv -r ./data metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Per-generation unique compound counts show multiplicative (exponential) growth across generations, matching the qualitative trend of Figure 3.
- Reaction counts exceed compound counts at each generation (e.g., 9,562,940 compounds vs. 13,759,651 reactions in the KEGG one-generation run).
- Benchmark runtimes increase monotonically with generation and are consistent with reported performance (12 million compounds inserted into mongo in ~9 hours).
- Growth/runtime CSV contains one row per generation with compound count, reaction count, and runtime columns populated.
- Reproduced plots match the runtime measurements of the benchmark notebook and the per-generation counts logged by mem_lt300_100r.py.

## Limitations

- Exponential growth makes multi-generation expansions computationally intractable; runtime benchmark data was only referenced as a supplementary file whose content is not fully included in the paper.
- Large expansions require a mongo database (local or remote) and substantial compute; runtime depends on rule count, starting compound count, and the applied MW/memory cutoff (e.g., <300 vs. <600 g/mol).
- Pickaxe software version numbers are not stated in the paper (only Python 3.7+ requirement), so exact reproducibility of runtimes may vary across minedatabase releases.

## Evidence

- [results] In the multi-generation YMDB expansion, the number of unique compounds produced increased exponentially with each generation, resulting in a total of two generations and 12 million new compounds being generated.: "12 million new compounds being generated and inserted into a mongo database in approximately nine hours"
- [abstract] This software iteratively applies reaction rules to a set of metabolites to generate novel reactions.: "This software iteratively applies reaction rules to a set of metabolites to generate novel reactions."
- [readme] The Figure 3 generation script and benchmark/plot notebooks are located in the pickaxe_paper repo.: "Generation code for exponential growth is found in `fig3_gen_exp_growth/run_files/mem_lt300_100r.py`"
- [readme] Runtime benchmarks are generated by a dedicated analysis notebook.: "Generation code for runtime is found in `fig3_gen_exp_growth/analysis/benchmarks.ipynb`"
- [discussion] Runtime performance benchmark data is only available as a supplementary file.: "Additional file 1. Pickaxe runtime performance benchmarks."
