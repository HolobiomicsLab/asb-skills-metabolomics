---
name: chemical-ontology-coverage-quantification
description: Decision support for measuring the fraction of a compound collection
  (e.g. PubChem's >91 million compounds as of June 2016) classified under a manually
  maintained chemical ontology such as MeSH, when arguing for automated classification.
when_to_use_negative:
- The input is already a per-compound classification table — use per-entity taxonomy
  assignment (e.g. ClassyFire's four-step classification process) instead of re-deriving
  coverage from aggregate baselines.
- The classification system's coverage is being measured empirically by running the
  classifier on the database rather than comparing published baseline figures.
- The goal is to evaluate classification accuracy or rule correctness rather than
  to quantify how much of a database an ontology covers.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: ClassyFire
  role: Automated structural classifier whose annotation scale (>77 million compounds)
    supplies the automated-coverage figure contrasted with manual MeSH coverage
  repo: http://classyfire.wishartlab.com
- name: ClassyFire API
  role: Ruby programmatic interface for querying ClassyFire's ChemOnt classification
    at scale when coverage is measured directly rather than from published baselines
  repo: https://bitbucket.org/wishartlab/classyfire_api
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:05365473bf84c01714ed7fc825834d115b260f4873eafffb24b516f9e2ef2580
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - ClassyFire API
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chemical-ontology-coverage-quantification

## Summary

Quantify how much of a large compound database is covered by a chemical classification system by computing absolute classified counts and percent-coverage ratios from reported baseline figures, to expose the manual-annotation gap that motivates automated taxonomies like ClassyFire's ChemOnt.

## When to use

Decision support for measuring the fraction of a compound collection (e.g. PubChem's >91 million compounds as of June 2016) classified under a manually maintained chemical ontology such as MeSH, when arguing for automated classification.

## When NOT to use

- The input is already a per-compound classification table — use per-entity taxonomy assignment (e.g. ClassyFire's four-step classification process) instead of re-deriving coverage from aggregate baselines.
- The classification system's coverage is being measured empirically by running the classifier on the database rather than comparing published baseline figures.
- The goal is to evaluate classification accuracy or rule correctness rather than to quantify how much of a database an ontology covers.

## Inputs

- Database compound-count baseline (e.g. PubChem >91,000,000 compounds as of June 2016)
- Reported classification coverage fraction for a manually maintained ontology (e.g. MeSH thesaurus, 0.12%)
- Reported annotated-compound count for the automated classification system (e.g. ClassyFire >77 million compounds)

## Outputs

- Coverage comparison table CSV (one row per classification system: MeSH manual vs ClassyFire automated; columns for compound count and percent coverage of the database)
- Computed absolute classified-compound count (~109,200 MeSH-classified PubChem compounds)
- Computed automated-system coverage ratio (~0.85, i.e. >77M of >91M PubChem compounds)

## How to apply

Record the reported baseline figures as constants: the database compound count (PubChem >91,000,000, June 2016), the ontology's classification coverage (0.12% via the MeSH thesaurus), and the alternative system's annotated count (ClassyFire >77 million compounds). Compute the absolute number of classified compounds as coverage_fraction × total_count (0.0012 × 91,000,000 ≈ 109,200 MeSH-classified PubChem compounds). Compute the automated system's coverage ratio as annotated_count / total_count (77,000,000 / 91,000,000 ≈ 84.6%). Assemble a one-row-per-classification-system comparison table with compound-count and percent-coverage columns, write it to CSV (e.g. coverage_comparison_table.csv) via pandas, and validate that the recovered MeSH-classified count equals ~109,200 and the ClassyFire ratio is ~0.85 before reporting. The rationale is that the three-order-of-magnitude gap between 0.12% manual coverage and ClassyFire's structural-rule-based annotation establishes the motivation for a purely structure-based taxonomy covering 'essentially all known chemical entities'.

## Related tools

- **ClassyFire** (Automated structural classifier whose annotation scale (>77 million compounds) supplies the automated-coverage figure contrasted with manual MeSH coverage) — http://classyfire.wishartlab.com
- **ClassyFire API** (Ruby programmatic interface for querying ClassyFire's ChemOnt classification at scale when coverage is measured directly rather than from published baselines) — https://bitbucket.org/wishartlab/classyfire_api

## Examples

```
python -c "import pandas as pd; mesh_n = round(0.0012*91000000); rows = [('MeSH (manual)', mesh_n, 0.0012), ('ClassyFire (automated)', 77000000, 77000000/91000000)]; pd.DataFrame(rows, columns=['system','classified','coverage']).to_csv('coverage_comparison_table.csv', index=False)"
```

## Evaluation signals

- Recovered MeSH-classified PubChem count equals ~109,200 (0.0012 × 91,000,000)
- Computed MeSH coverage fraction equals 0.12% of the reported PubChem total
- ClassyFire coverage ratio is ~0.85 (>77M annotated of >91M total PubChem compounds)
- Comparison table has exactly one row per classification system (MeSH vs ClassyFire) with compound-count and percent-coverage columns
- CSV output loads in pandas without schema errors and all numeric fields match the recomputed values

## Limitations

- Baseline figures are as reported in the source (PubChem count as of June 2016; coverage figures from the article's abstract) — recompute against current database sizes when reusing later.
- ClassyFire's coverage ratio uses the abstract's '>77 million compounds' figure, which may overlap only partially with the specific PubChem snapshot being compared.
- The manual MeSH mapping to ClassyFire categories was only partial, so cross-ontology coverage comparisons may be incomplete.
- ClassyFire source code was not yet freely available at the time of writing (open-source release promised by December 2016), limiting reproducibility of coverage measurement from source.

## Evidence

- [abstract] PubChem/MeSH coverage-gap baseline: "ClassyFire has been used to annotate over 77 million compounds and has already been integrated into other software packages"
- [methods] PubChem total and MeSH coverage: "ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database."
- [intro] structural coverage rationale: "which permits automated rule-based structural classification of essentially all known chemical entities"
- [methods] four-step classification process: "the ClassyFire classification process involves four steps: (1) Creation and Preprocessing of the Chemical Entity; (2) Feature Extraction; (3) Rule-based Category Assignment and Category Reduction;"
