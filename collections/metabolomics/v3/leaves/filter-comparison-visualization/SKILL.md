---
name: filter-comparison-visualization
description: Use when a reaction-network expansion has been generated under multiple
  filter combinations and you need per-combination counts of annotated compounds/peaks
  plus a comparison figure (e.g. reproducing Figure 6).
when_to_use_negative:
- You only need a single filter combination's network — this skill exists to contrast
  multiple combinations (metabolomics vs +thermodynamics vs +feasibility).
- Your expansion uses a different starting database, rule count, or generation depth
  (e.g. the KEGG top-500-rule single-generation run of Figure 2) — the Figure 6 counts
  are specific to EcoCyc/two generations/top 50 rules.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pickaxe
  role: Generates the EcoCyc expansion whose filtered variants are compared; part
    of the minedatabase package
  repo: https://github.com/tyo-nu/MINE-Database
- name: MongoDB
  role: Stores the generated compounds and reactions; results queried to produce per-combination
    counts
  repo: https://www.mongodb.com/try/download/community
- name: PyMongo
  role: Python interface used by the analysis notebook to query the Mongo database
    and count annotated peaks per filter combination
  repo: https://pymongo.readthedocs.io/en/stable/
- name: eQuilibrator
  role: Provides physiological Gibbs free energies for the thermodynamics filter (remove
    reactions >50 kJ/mol, retain reactions without computable values)
  repo: https://equilibrator.readthedocs.io/en/latest/
- name: DeepRFC
  role: Provides the reaction feasibility filter, one of the compared filter combinations
  repo: https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:1d01e19900e7e40cbc907bebcc0f50e63dbe39eecf39819f4d3812efcf691d3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - DeepRFC
  - MongoDB
  - Pickaxe
  - PyMongo
  - eQuilibrator
  - minedatabase
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# filter-comparison-visualization

## Summary

Reproduces the Pickaxe paper's Figure 6-style comparison of network-pruning filter combinations (metabolomics-only, metabolomics+thermodynamics, metabolomics+feasibility) on an EcoCyc expansion, counting how many unannotated E. coli metabolomics peaks each combination annotates and rendering the comparison plot.

## When to use

Use when a reaction-network expansion has been generated under multiple filter combinations and you need per-combination counts of annotated compounds/peaks plus a comparison figure (e.g. reproducing Figure 6).

## When NOT to use

- You only need a single filter combination's network — this skill exists to contrast multiple combinations (metabolomics vs +thermodynamics vs +feasibility).
- Your expansion uses a different starting database, rule count, or generation depth (e.g. the KEGG top-500-rule single-generation run of Figure 2) — the Figure 6 counts are specific to EcoCyc/two generations/top 50 rules.

## Inputs

- 200 unannotated metabolomics peaks with m/z values between 150 and 300 Da from the E. coli untargeted metabolomics dataset
- Pickaxe-expanded EcoCyc network (compounds <600 Da, two generations, top 50 JN1224min rules) stored in a MongoDB database
- Thermodynamics annotations (physiological Gibbs free energies from eQuilibrator)
- Reaction feasibility annotations (DeepRFC feasibility scripts)

## Outputs

- Per-filter-combination count table of annotated compounds and compound–peak pairs (e.g. figure6_peak_annotations.csv)
- Comparison plot of peak annotations across filter combinations (e.g. figure6_annotation_plot.png)

## How to apply

Start from a Pickaxe expansion of EcoCyc compounds <600 Da over two generations using the top 50 JN1224min rules (65% MetaCyc mapping), stored in MongoDB. Select 200 unannotated peaks with m/z 150–300 Da from the E. coli untargeted metabolomics dataset as metabolomics-filter inputs. Build the three filter combinations: metabolomics-only (keep compounds whose m/z matches an unannotated peak), thermodynamics (remove reactions with physiological Gibbs free energy >50 kJ/mol via eQuilibrator while retaining reactions without a computable Gibbs energy), and reaction feasibility (DeepRFC). Execute the analysis notebook fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_analysis.ipynb, which queries the Mongo database with PyMongo and counts annotated peaks for each combination. Write the per-combination counts to a CSV table (e.g. figure6_peak_annotations.csv) and render the comparison plot (figure6_annotation_plot.png).

## Related tools

- **Pickaxe** (Generates the EcoCyc expansion whose filtered variants are compared; part of the minedatabase package) — https://github.com/tyo-nu/MINE-Database
- **MongoDB** (Stores the generated compounds and reactions; results queried to produce per-combination counts) — https://www.mongodb.com/try/download/community
- **PyMongo** (Python interface used by the analysis notebook to query the Mongo database and count annotated peaks per filter combination) — https://pymongo.readthedocs.io/en/stable/
- **eQuilibrator** (Provides physiological Gibbs free energies for the thermodynamics filter (remove reactions >50 kJ/mol, retain reactions without computable values)) — https://equilibrator.readthedocs.io/en/latest/
- **DeepRFC** (Provides the reaction feasibility filter, one of the compared filter combinations) — https://bitbucket.org/kaistsystemsbiology/deeprfc/src/master/

## Examples

```
jupyter nbconvert --to notebook --execute fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_analysis.ipynb
```

## Evaluation signals

- Adding the feasibility filter to the metabolomics filter reduces both compound counts and compound–peak pairs by roughly an order of magnitude versus metabolomics alone.
- The thermodynamics-plus-metabolomics combination yields annotation counts similar to metabolomics alone (reactions lacking a computable Gibbs energy are retained).
- The number of unannotated peaks matched stays similar between metabolomics-only and metabolomics+feasibility despite the order-of-magnitude reduction in compounds.
- The reproduced CSV count table qualitatively matches the paper's reported Figure 6 pattern.
- The Mongo database contains compounds/reactions for all three filter combinations before counting begins.

## Limitations

- eQuilibrator may not return a Gibbs free energy for reactions outside its prediction scope; such reactions were left in the network rather than removed.
- The analysis code was run both on a laptop and a supercomputer in the paper repo, and the analysis can be sped up with feasibility_first_gen.py, feasibility_second_gen_gen2.py, and get_physiological_dgs.py — expect runtime cost for the feasibility path.
- Results are stored in a MongoDB instance (local or remote) that must be installed separately; the comparison depends on that stored state.

## Evidence

- [results] 200 unannotated peaks with m/z values between 150 and 300 Da were selected from an experimental dataset as inputs to the metabolomics filter: "200 unannotated peaks with m/z values between 150 and 300 Da were selected from an experimental dataset as inputs to the metabolomics filter"
- [results] Compounds less than 600 Da selected from the EcoCyc database were expanded for two generations using the top 50 JN1224min rules (65% MetaCyc mapping).: "Compounds less than 600 Da selected from the EcoCyc database were expanded for two generations using the top 50 JN1224min rules (65% MetaCyc mapping)."
- [results] The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol.: "The thermodynamics filter was set to remove reactions with a physiological Gibbs free energy greater than 50 kJ/mol."
- [results] These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility.: "These reactions were left to remain in the network, as the thermodynamics filter cannot comment on their feasibility."
- [readme] The Analysis code can be sped up using feasibility_first_gen.py, feasibility_second_gen_gen2.py, and get_physiological_dgs.py: "The Analysis code can be sped up using `feasibility_first_gen.py`, `feasibility_second_gen_gen2.py`, and `get_physiological_dgs.py`"
- [readme] PyMongo provides methods to interact with the database via python code.: "[PyMongo](https://pymongo.readthedocs.io/en/stable/) provides methods to interact with the database via python code."
- [results] it is possible that eQuilibrator does not return a Gibbs free energy for a reaction, as the reaction falls outside the scope of its predictions: "it is possible that eQuilibrator does not return a Gibbs free energy for a reaction, as the reaction falls outside the scope of its predictions"
