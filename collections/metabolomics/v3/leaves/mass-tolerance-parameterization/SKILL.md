---
name: mass-tolerance-parameterization
description: Use when constraining a Pickaxe network expansion to an unannotated metabolomics
  peak list and you must choose the adduct list and user-defined mass tolerance that
  decide which predicted compounds are retained.
when_to_use_negative:
- When the goal is annotating peaks with known database compounds rather than constraining
  a novel reaction network — the metabolomics filter only retains predicted compounds,
  it does not assign identities to peaks
- When no experimental peak list or adduct list is available; the filter cannot operate
  without user-supplied peaks, adducts, and a tolerance
- When filtering on other criteria such as chemical similarity to targets, thermodynamics,
  or reaction feasibility — Pickaxe provides separate filters for those
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Pickaxe
  role: Framework within which the metabolomics filter with its adduct-based m/z matching
    and mass tolerance is implemented and applied on-the-fly during network generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase)
  role: Python package providing Pickaxe and the metabolomics filter; installed with
    pip install minedatabase
  repo: https://pypi.org/project/minedatabase/2.0.0/
- name: MongoDB
  role: Stores the results of the E. coli metabolomics-filter network generation (fig6
    example)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:66306d4aafc36a4d489b0c9cb88839e94ef22ae0ff7322bb626d4b7b8c0f7d98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase)
  - MongoDB
  - Pickaxe
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-tolerance-parameterization

## Summary

Parameterizes the Pickaxe metabolomics filter: for each predicted compound, adducted m/z values are computed from a user-supplied adduct list and matched against experimental unannotated peaks within a user-defined mass tolerance, so only compounds consistent with observed peaks survive network expansion.

## When to use

Use when constraining a Pickaxe network expansion to an unannotated metabolomics peak list and you must choose the adduct list and user-defined mass tolerance that decide which predicted compounds are retained.

## When NOT to use

- When the goal is annotating peaks with known database compounds rather than constraining a novel reaction network — the metabolomics filter only retains predicted compounds, it does not assign identities to peaks
- When no experimental peak list or adduct list is available; the filter cannot operate without user-supplied peaks, adducts, and a tolerance
- When filtering on other criteria such as chemical similarity to targets, thermodynamics, or reaction feasibility — Pickaxe provides separate filters for those

## Inputs

- List of unannotated experimental m/z peak values (e.g. 200 peaks in the 150-300 Da range from the E. coli untargeted metabolomics dataset)
- Set of predicted compounds from a Pickaxe network expansion (e.g. EcoCyc compounds <600 Da expanded two generations with top 50 JN1224min rules)
- User-supplied adduct list (adducts applied to compound neutral masses)
- User-defined mass tolerance for m/z matching

## Outputs

- CSV table recording, for each retained compound, the experimental m/z peak and the adduct that produced the match
- Filtered network retaining only compounds whose adducted m/z matches an experimental peak

## How to apply

Load the list of unannotated experimental m/z peaks — in the Pickaxe paper's example, 200 unannotated peaks with m/z values between 150 and 300 Da selected from an experimental E. coli dataset — together with the predicted compounds from expanding EcoCyc compounds <600 Da with the top 50 JN1224min rules. For each predicted compound, apply each adduct in the user-supplied adduct list to the compound's neutral mass to compute the set of possible m/z values. Choose the user-defined mass tolerance (e.g. a ppm or absolute-Da window appropriate to the instrument's mass accuracy) and retain only compounds with at least one computed adducted m/z matching an input peak, consistent with the metabolomics filter that 'only keeps compounds whose m/z value matches one of those in the unannotated metabolomics dataset'. Implement the filter as part of the minedatabase/Pickaxe filter framework (installed via pip install minedatabase) so it runs on-the-fly during network generation, and write a CSV recording for each retained compound the matched experimental peak and the adduct that produced the match.

## Related tools

- **Pickaxe** (Framework within which the metabolomics filter with its adduct-based m/z matching and mass tolerance is implemented and applied on-the-fly during network generation) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase)** (Python package providing Pickaxe and the metabolomics filter; installed with pip install minedatabase) — https://pypi.org/project/minedatabase/2.0.0/
- **MongoDB** (Stores the results of the E. coli metabolomics-filter network generation (fig6 example))

## Examples

```
jupyter notebook fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_gen.ipynb
```

## Evaluation signals

- Every compound retained in the output table has at least one computed adducted m/z within the user-defined mass tolerance of an input experimental peak
- Each row of the retained-compound CSV records both the matched experimental peak and the adduct that produced the match
- No predicted compound whose full adducted m/z set misses all input peaks appears in the filtered network
- Recomputing matches with a tightened tolerance should monotonically reduce (or keep equal) the number of retained compounds

## Limitations

- m/z matching is not structural annotation: a match only says the compound's adducted mass is consistent with an unannotated peak, so multiple compounds can match the same peak
- The retained set depends entirely on the user-supplied adduct list and tolerance; adducts not included can never produce matches
- No Pickaxe software version number or tolerance defaults are stated in the paper; runtime benchmarks are only in a supplementary file whose content is not included
- The E. coli example used 200 unannotated peaks restricted to 150-300 Da, so retention is limited to compounds whose adducted m/z falls in that range

## Evidence

- [results] Metabolomics filter: keep only compounds whose m/z matches peaks in the unannotated dataset: "the metabolomics filter, which only keeps compounds whose m/z value matches one of those in the unannotated metabolomics dataset"
- [results] Peak selection for the E. coli metabolomics filter example: "200 unannotated peaks with m/z values between 150 and 300 Da were selected from an experimental dataset as inputs to the metabolomics filter"
- [results] Network expansion context for the metabolomics filter example: "Compounds less than 600 Da selected from the EcoCyc database were expanded for two generations using the top 50 JN1224min rules (65% MetaCyc mapping)."
- [readme] Filter software dependencies: "The code in this example requires the minedatabase package as well as some additional packages for different filters."
- [readme] Figure 6 metabolomics example code location: "Generation code is found in `fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_gen.ipynb`"
- [abstract] Filters available in Pickaxe including metabolomics: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
