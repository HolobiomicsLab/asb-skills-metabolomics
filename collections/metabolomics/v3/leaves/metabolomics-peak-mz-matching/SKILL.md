---
name: metabolomics-peak-mz-matching
description: Use when annotating untargeted metabolomics peaks (e.g. 200 unannotated
  peaks, m/z 150-300 Da) by constraining a Pickaxe-generated reaction network to compounds
  whose adducted m/z matches an input peak list.
when_to_use_negative:
- Input is already an annotated compound list with assigned metabolite identities
  — no peak-matching is needed.
- The goal is retaining compounds by chemical similarity, thermodynamics, or reaction
  feasibility — Pickaxe provides separate filters for those purposes.
- Predicted compounds need to be kept regardless of whether they correspond to an
  observed peak (e.g. during unrestricted network expansion).
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0605
tools:
- name: Pickaxe
  role: Provides the filter framework in which the metabolomics m/z filter is implemented
    and applied on-the-fly during reaction network generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: MINE-Database (minedatabase python package)
  role: Python package containing Pickaxe, installed via pip, used to run the network
    expansion and the metabolomics filter
  repo: https://pypi.org/project/minedatabase/
- name: Python
  role: Implementation language for the m/z computation, matching, and CSV output
    (requires Python 3.7 or higher)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:350450382156deebbbf2727f66cbcea56e5f0c2a13f25aea1b0b7910ced4ff17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - MINE-Database (minedatabase python package)
  - Pickaxe
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolomics-peak-mz-matching

## Summary

A Pickaxe metabolomics filter that computes possible m/z values for each predicted compound from user-supplied adducts and retains only compounds whose adducted m/z matches an unannotated experimental peak within a user-defined mass tolerance.

## When to use

Use when annotating untargeted metabolomics peaks (e.g. 200 unannotated peaks, m/z 150-300 Da) by constraining a Pickaxe-generated reaction network to compounds whose adducted m/z matches an input peak list.

## When NOT to use

- Input is already an annotated compound list with assigned metabolite identities — no peak-matching is needed.
- The goal is retaining compounds by chemical similarity, thermodynamics, or reaction feasibility — Pickaxe provides separate filters for those purposes.
- Predicted compounds need to be kept regardless of whether they correspond to an observed peak (e.g. during unrestricted network expansion).

## Inputs

- List of unannotated experimental m/z peak values (e.g. 200 peaks, 150-300 Da, from an E. coli untargeted metabolomics dataset)
- Set of predicted compounds from a Pickaxe network expansion
- User-supplied adduct list applied to each compound's neutral mass
- User-defined mass tolerance for m/z matching

## Outputs

- Filtered network containing only compounds whose adducted m/z matches at least one experimental peak
- CSV table recording, for each retained compound, the matched experimental m/z peak and the adduct that produced the match

## How to apply

Load the unannotated experimental peak list (e.g. 200 peaks with m/z 150-300 Da selected from an E. coli untargeted metabolomics dataset) alongside the predicted compounds from the Pickaxe network expansion (e.g. EcoCyc compounds <600 Da expanded for two generations with the top 50 JN1224min rules). For each predicted compound, compute possible m/z values by applying each adduct in the user-supplied adduct list to the compound's neutral mass. Match computed m/z values against the peak list within the user-defined mass tolerance, keeping only compounds with at least one match — the filter 'only keeps compounds whose m/z value matches one of those in the unannotated metabolomics dataset'. Implement the filter as a Python module within the minedatabase/Pickaxe filter framework so it can be applied on-the-fly during network generation, and write matched compounds to a CSV table recording the experimental m/z peak and adduct for each retained compound.

## Related tools

- **Pickaxe** (Provides the filter framework in which the metabolomics m/z filter is implemented and applied on-the-fly during reaction network generation) — https://github.com/tyo-nu/MINE-Database
- **MINE-Database (minedatabase python package)** (Python package containing Pickaxe, installed via pip, used to run the network expansion and the metabolomics filter) — https://pypi.org/project/minedatabase/
- **Python** (Implementation language for the m/z computation, matching, and CSV output (requires Python 3.7 or higher))

## Examples

```
python pickaxe.py -C ./data/metacyc_generalized_rules.tsv -r ./data/metacyc_coreactants.tsv -g 1 -c ../example_data/iML1515_ecoli_GEM.csv
```

## Evaluation signals

- Every compound retained in the output table must have at least one computed adduct m/z within the user-defined tolerance of an input experimental peak.
- Each output row must record both the matched experimental m/z peak and the adduct that produced the match.
- No compound whose adducted m/z falls outside tolerance of every input peak should survive the filter.
- Applying the filter to the E. coli case study (200 peaks, m/z 150-300 Da) should shrink the expanded EcoCyc network to only peak-matching compounds.
- Compounds and reactions rejected by the filter should be absent from downstream network outputs.

## Limitations

- The filter can only annotate peaks matching predicted compounds; peaks with no corresponding network compound remain unannotated.
- Matching is limited by the user-supplied adduct list — compounds are missed if the correct adduct is not included.
- Accuracy depends on the user-defined mass tolerance; too loose a tolerance retains false matches, too tight loses true ones (tolerance value not specified in the article).
- Runtime performance benchmarks are referenced as a supplementary file but their content is not included, so filter runtime expectations are not documented here.

## Evidence

- [results] metabolomics filter m/z matching: "the metabolomics filter, which only keeps compounds whose m/z value matches one of those in the unannotated metabolomics dataset"
- [results] peak selection for filter inputs: "200 unannotated peaks with m/z values between 150 and 300 Da were selected from an experimental dataset as inputs to the metabolomics filter"
- [results] network expansion setup for E. coli metabolomics case: "Compounds less than 600 Da selected from the EcoCyc database were expanded for two generations using the top 50 JN1224min rules"
- [abstract] on-the-fly filtering capability: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [readme] E. coli metabolomics figure code location: "Generation code is found in `fig6_ecocyc_metabolomics/laptop/analysis/ecoli_metabolomics_gen.ipynb`"
