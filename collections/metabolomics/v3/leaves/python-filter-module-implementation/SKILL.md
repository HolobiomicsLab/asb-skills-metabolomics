---
name: python-filter-module-implementation
description: Use when a predicted reaction network must be constrained during generation
  — e.g., keeping only compounds whose adducted m/z matches unannotated metabolomics
  peaks — by coding the filter as a Python module in the minedatabase framework.
when_to_use_negative:
- The peaks are already annotated with compound identities, so no adducted m/z matching
  is needed.
- The desired pruning is thermodynamic (eQuilibrator) or reaction-feasibility (DeepRFC)
  based rather than metabolomics m/z based.
- The goal is post-hoc database retrieval from a stored MINE rather than on-the-fly
  filtering during an active Pickaxe run.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: Pickaxe
  role: Host framework whose filter mechanism the custom Python module plugs into;
    applies reaction rules and filters on-the-fly during network generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase)
  role: Python package installed via pip that bundles Pickaxe and its filter framework;
    the filter module is developed within this package
  repo: https://github.com/tyo-nu/MINE-Database
- name: Python
  role: Implementation language for the filter module (Python 3.7 or higher)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:c44e1cb584c293993ae6d45145f6f2d2414c0ce8c95dafd13624522a706337e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - MINE-Database (minedatabase)
  - Pickaxe
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# python-filter-module-implementation

## Summary

Implement a Pickaxe on-the-fly filter (e.g., the metabolomics filter) as a Python module within the minedatabase/Pickaxe framework so that compounds failing the filter are pruned during network expansion rather than after it.

## When to use

Use when a predicted reaction network must be constrained during generation — e.g., keeping only compounds whose adducted m/z matches unannotated metabolomics peaks — by coding the filter as a Python module in the minedatabase framework.

## When NOT to use

- The peaks are already annotated with compound identities, so no adducted m/z matching is needed.
- The desired pruning is thermodynamic (eQuilibrator) or reaction-feasibility (DeepRFC) based rather than metabolomics m/z based.
- The goal is post-hoc database retrieval from a stored MINE rather than on-the-fly filtering during an active Pickaxe run.

## Inputs

- List of unannotated experimental m/z peak values (e.g., 200 peaks, m/z 150–300 Da from the E. coli untargeted metabolomics dataset)
- Set of predicted compounds from a Pickaxe network expansion (e.g., EcoCyc compounds <600 Da expanded two generations with the top 50 JN1224min rules)
- User-supplied adduct list applied to each compound's neutral mass
- User-defined mass tolerance for m/z matching

## Outputs

- Filtered network containing only compounds with at least one adduct m/z matching an input peak
- CSV table of retained compounds, each row recording the matched experimental m/z peak and the adduct that produced the match

## How to apply

Write the filter as a Python module inside the minedatabase/Pickaxe filter framework so it executes on-the-fly during network generation, since Pickaxe applies filters (chemical similarity, metabolomics, thermodynamics, reaction feasibility) while expanding compounds. For the metabolomics filter, load the list of unannotated experimental m/z peaks (e.g., 200 peaks in the 150–300 Da range from the E. coli untargeted metabolomics dataset) and, for each predicted compound, compute possible m/z values by applying every adduct in the user-supplied adduct list to the compound's neutral mass. Retain a compound if at least one computed adduct m/z matches an input peak within the user-defined mass tolerance. Write retained compounds to a CSV table that records, per compound, the matched experimental peak and the adduct that produced the match. Base the module on the pickaxe_run_template.py pattern for running Pickaxe through a Python file, and install dependencies via 'pip install minedatabase'.

## Related tools

- **Pickaxe** (Host framework whose filter mechanism the custom Python module plugs into; applies reaction rules and filters on-the-fly during network generation) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase)** (Python package installed via pip that bundles Pickaxe and its filter framework; the filter module is developed within this package) — https://github.com/tyo-nu/MINE-Database
- **Python** (Implementation language for the filter module (Python 3.7 or higher))

## Examples

```
pip install minedatabase
```

## Evaluation signals

- Every compound retained in the output CSV has at least one computed adduct m/z within the user-defined tolerance of an input experimental peak.
- Every row of the output CSV records both the matched experimental m/z peak and the adduct that produced the match.
- The filter executes on-the-fly during network generation (compounds failing the filter are not expanded further), not as a post-hoc pass.
- Recomputing adduct m/z from the compound's neutral mass with the same adduct list and tolerance reproduces the same retained set (deterministic matching).
- The module imports cleanly as part of the minedatabase package and runs in the pickaxe_run_template.py workflow.

## Limitations

- Retention depends entirely on the user-supplied adduct list and mass tolerance; compounds whose true ion species is not in the adduct list will be incorrectly pruned.
- No software version numbers are stated for the minedatabase package in the paper (only Python 3.7+), so reproducibility of filter behavior requires pinning versions manually.
- Runtime performance benchmarks for the paper are referenced as a supplementary file whose content is not included, so filter overhead must be measured locally.

## Evidence

- [results] metabolomics filter keeps only matching compounds: "the metabolomics filter, which only keeps compounds whose m/z value matches one of those in the unannotated metabolomics dataset"
- [results] 200 peaks at m/z 150-300 Da as filter inputs: "200 unannotated peaks with m/z values between 150 and 300 Da were selected from an experimental dataset as inputs to the metabolomics filter"
- [abstract] four filter types available in Pickaxe: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [readme] minedatabase package required for the filters: "The code in this example requires the minedatabase package as well as some additional packages for different filters."
- [readme] Python file template for running Pickaxe: [evidence span withheld — anchor status: not_found]
