---
name: metabolomics-peak-annotation
description: Use when a list of unannotated peaks from an untargeted metabolomics
  run (e.g., 200 peaks with m/z 150-300 Da) needs candidate compound assignments via
  rule-based network expansion and on-the-fly filtering.
when_to_use_negative:
- The peaks of interest are already annotated with known compound identities, so no
  rule-based expansion is needed
- The task requires only experimentally confirmed reactions/compounds, not predicted
  novel compounds from promiscuous-enzyme reaction rules
- The compounds of interest exceed the mass range supported by the setup (expansions
  here filter starting compounds to < 600 Da and peaks to 150-300 Da)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe (minedatabase package)
  role: Applies reaction rules iteratively to starting compounds to expand EcoCyc
    into a novel reaction network for peak annotation
  repo: https://github.com/tyo-nu/MINE-Database
- name: MongoDB / PyMongo
  role: Stores expansion results and provides query interface used by the analysis
    notebook to count annotated peaks per filter combination
  repo: https://pymongo.readthedocs.io/en/stable/
- name: eQuilibrator (equilibrator-api)
  role: 'Thermodynamics filter: computes physiological Gibbs free energy to remove
    reactions with ΔG > 50 kJ/mol'
  repo: https://equilibrator.readthedocs.io/en/latest/
- name: DeepRFC
  role: Reaction feasibility filter scripts used to prune the expanded network and
    reduce compound/peak-pair counts
  repo: https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:d5e416ff782a4049a93d3ca9833fc712d9e39bb39163dc2e64526cf1066b1623
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - DeepRFC
  - MongoDB / PyMongo
  - Pickaxe
  - Pickaxe (minedatabase package)
  - PyMongo
  - eQuilibrator
  - eQuilibrator (equilibrator-api)
  - minedatabase
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# metabolomics-peak-annotation

## Summary

Annotates unannotated untargeted metabolomics peaks by expanding a compound database (e.g., EcoCyc) with Pickaxe reaction rules, then filtering the predicted network by metabolomics m/z match, thermodynamics, and reaction feasibility to identify plausible structures for unknown peaks.

## When to use

Use when a list of unannotated peaks from an untargeted metabolomics run (e.g., 200 peaks with m/z 150-300 Da) needs candidate compound assignments via rule-based network expansion and on-the-fly filtering.

## When NOT to use

- The peaks of interest are already annotated with known compound identities, so no rule-based expansion is needed
- The task requires only experimentally confirmed reactions/compounds, not predicted novel compounds from promiscuous-enzyme reaction rules
- The compounds of interest exceed the mass range supported by the setup (expansions here filter starting compounds to < 600 Da and peaks to 150-300 Da)

## Inputs

- List of unannotated peaks with m/z values from an untargeted metabolomics dataset (e.g., 200 peaks, 150-300 Da)
- Starting compound database (EcoCyc 25.1, compounds < 600 Da)
- Reaction ruleset (JN1224min, top 50 rules, 65% MetaCyc mapping)
- Local or remote MongoDB instance for storing expansion results

## Outputs

- MongoDB collection of generated compounds and reactions from the two-generation expansion
- CSV table of per-filter-combination peak annotation counts (e.g., figure6_peak_annotations.csv)
- Comparison plot of annotation counts across filter combinations (e.g., figure6_annotation_plot.png)

## How to apply

Select the unannotated peaks (e.g., 200 peaks with m/z between 150 and 300 Da) as metabolomics filter inputs, and filter the source database (EcoCyc 25.1) to compounds under 600 Da. Expand the network for two generations using the top 50 JN1224min rules (65% MetaCyc mapping) with Pickaxe via the minedatabase package, storing generated compounds and reactions in MongoDB. Apply filter combinations across the expansion: metabolomics-only (keep compounds whose m/z matches an unannotated peak), thermodynamics (eQuilibrator, removing reactions with physiological Gibbs free energy greater than 50 kJ/mol while retaining reactions without a computable Gibbs energy), and reaction feasibility (DeepRFC feasibility scripts). Query the Mongo database with PyMongo to count annotated peaks per filter combination, write the counts to a CSV table, and render the comparison plot (e.g., figure6_peak_annotations.csv and figure6_annotation_plot.png). Rationale: rule-based expansion enumerates candidate metabolites matching each peak m/z, and successive filters prune chemically or thermodynamically implausible candidates while preserving peak-matching annotations.

## Related tools

- **Pickaxe (minedatabase package)** (Applies reaction rules iteratively to starting compounds to expand EcoCyc into a novel reaction network for peak annotation) — https://github.com/tyo-nu/MINE-Database
- **MongoDB / PyMongo** (Stores expansion results and provides query interface used by the analysis notebook to count annotated peaks per filter combination) — https://pymongo.readthedocs.io/en/stable/
- **eQuilibrator (equilibrator-api)** (Thermodynamics filter: computes physiological Gibbs free energy to remove reactions with ΔG > 50 kJ/mol) — https://equilibrator.readthedocs.io/en/latest/
- **DeepRFC** (Reaction feasibility filter scripts used to prune the expanded network and reduce compound/peak-pair counts) — https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/

## Examples

```
python pickaxe.py -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Metabolomics-only filter and thermodynamics-plus-metabolomics combination annotate a similar number of unannotated peaks
- Adding the feasibility filter on top of the metabolomics filter reduces both generated compounds and compound peak pairs by roughly an order of magnitude while still matching a similar number of peaks
- Peak annotation counts per filter combination are recovered by PyMongo queries against the expansion MongoDB and round-trip into the CSV table
- Expansion parameters match the paper setup: two generations, top 50 JN1224min rules (65% MetaCyc mapping), compounds < 600 Da, peaks in the 150-300 Da range

## Limitations

- eQuilibrator may not return a Gibbs free energy for some reactions (outside its prediction scope); such reactions must be retained since the thermodynamics filter cannot comment on their feasibility
- Network expansions are computationally heavy: applying JN1224min rules to KEGG generated 12 million new compounds inserted into Mongo in approximately nine hours; DeepRFC feasibility filtering requires extra scripts to speed up analysis
- DeepRFC installation is nontrivial and must follow instructions on its Bitbucket site; no software version numbers are stated for the packages used

## Evidence

- [results] Select 200 unannotated peaks with m/z 150-300 Da: "200 unannotated peaks with m/z values between 150 and 300 Da were selected from an experimental dataset as inputs to the metabolomics filter"
- [results] EcoCyc two-generation expansion with top 50 JN1224min rules: "Compounds less than 600 Da selected from the EcoCyc database were expanded for two generations using the top 50 JN1224min rules (65% MetaCyc mapping)."
- [results] Thermodynamics filter cutoff: "The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol."
- [results] Retaining reactions without computable Gibbs energy: "These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility."
- [readme] Figure 6 generation and analysis notebook locations: "Generation code is found in `fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_gen.ipynb`"
- [readme] DeepRFC required for feasibility filters: "The feasibility filters require the use of DeepRFC, which can be installed by [following the instructions found on their website.]"
- [results] eQuilibrator scope limitation: "it is possible that eQuilibrator does not return a Gibbs free energy for a reaction, as the reaction falls outside the scope of its predictions"
