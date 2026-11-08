---
name: mesh-taxonomy-mapping-analysis
description: Decision support for benchmarking a manual chemical classification system's
  coverage (e.g. MeSH over PubChem) against an automated, structure-based classifier
  such as ClassyFire before choosing an annotation pipeline.
when_to_use_negative:
- When the goal is to classify novel structures with ClassyFire/ChemOnt rather than
  to quantify a baseline coverage gap — in that case submit structures directly to
  the ClassyFire webserver instead of computing coverage ratios.
- When comparing two automated structure-based classifiers on the same labeled test
  set (e.g. ClassyFire vs. ChEBI ancestor mapping), where per-compound agreement metrics
  rather than PubChem-wide coverage percentages are the appropriate output.
- When a current PubChem snapshot or a different manual thesaurus version is required
  — the 0.12% MeSH figure and >91M compound count are specific to the June 2016 snapshot
  reported in the article.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0089
tools:
- name: ClassyFire
  role: Automated, structure-based classification engine whose annotation count (>77
    million compounds) supplies the automated-coverage denominator for the coverage-gap
    computation against the manual MeSH baseline.
  repo: http://classyfire.wishartlab.com
- name: manual expert review
  role: The manual curation process underlying the MeSH thesaurus assignments whose
    0.12% PubChem coverage forms the manual baseline in the comparison.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:21d08d2355028ddd3ce5265ae7946ff1353593fcb2eeeedfc28522772f92ade2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - manual expert review
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mesh-taxonomy-mapping-analysis

## Summary

Quantifies the coverage gap between the manually maintained MeSH thesaurus and large-scale automated chemical classification (ClassyFire/ChemOnt) over the PubChem compound space, by computing absolute MeSH-classified compound counts and percentage coverage of PubChem's >91 million structures.

## When to use

Decision support for benchmarking a manual chemical classification system's coverage (e.g. MeSH over PubChem) against an automated, structure-based classifier such as ClassyFire before choosing an annotation pipeline.

## When NOT to use

- When the goal is to classify novel structures with ClassyFire/ChemOnt rather than to quantify a baseline coverage gap — in that case submit structures directly to the ClassyFire webserver instead of computing coverage ratios.
- When comparing two automated structure-based classifiers on the same labeled test set (e.g. ClassyFire vs. ChEBI ancestor mapping), where per-compound agreement metrics rather than PubChem-wide coverage percentages are the appropriate output.
- When a current PubChem snapshot or a different manual thesaurus version is required — the 0.12% MeSH figure and >91M compound count are specific to the June 2016 snapshot reported in the article.

## Inputs

- PubChem compound count for the snapshot of interest (e.g. >91,000,000 as of June 2016)
- Manual classification system coverage statistic (e.g. MeSH thesaurus coverage reported as a fraction, 0.12%)
- Automated classification system annotation count (e.g. ClassyFire annotation count of >77 million compounds)

## Outputs

- Absolute MeSH-classified compound count (~109,200 for the June 2016 PubChem snapshot)
- Automated classification coverage ratio of PubChem (77M / 91M ≈ 84.6%)
- Coverage comparison table (CSV, one row per classification system, columns for compound counts and percent coverage)

## How to apply

Record the three baseline constants reported for the June 2016 PubChem snapshot: the total compound count of >91,000,000, the MeSH thesaurus classification coverage of 0.12%, and the ClassyFire annotation count of >77 million compounds. Compute the absolute number of MeSH-classified PubChem compounds as 0.0012 × 91,000,000 ≈ 109,200, and the ClassyFire deployment coverage ratio as 77,000,000 / 91,000,000 ≈ 84.6%. Assemble a comparison table with one row per classification system (manual MeSH thesaurus coverage vs. automated ClassyFire coverage), with columns for compound counts and percent coverage of PubChem, then write it to a CSV (e.g. coverage_comparison_table.csv) using pandas and report both computed values. Validate that the MeSH-classified count is ~109,200, the MeSH coverage fraction equals 0.12%, and the ClassyFire ratio is ~0.85, matching the baseline figures reported in the article.

## Related tools

- **ClassyFire** (Automated, structure-based classification engine whose annotation count (>77 million compounds) supplies the automated-coverage denominator for the coverage-gap computation against the manual MeSH baseline.) — http://classyfire.wishartlab.com
- **manual expert review** (The manual curation process underlying the MeSH thesaurus assignments whose 0.12% PubChem coverage forms the manual baseline in the comparison.)

## Examples

```
python -c "import pandas as pd; df = pd.DataFrame([{'system':'MeSH thesaurus (manual)','compounds':0.0012*91e6,'coverage_pct':0.12},{'system':'ClassyFire (automated)','compounds':77e6,'coverage_pct':77e6/91e6*100}]); df.to_csv('coverage_comparison_table.csv', index=False)"
```

## Evaluation signals

- Computed MeSH-classified PubChem count equals ~109,200 (0.0012 × 91,000,000).
- Computed MeSH coverage fraction equals 0.12%.
- Computed ClassyFire coverage ratio is ~0.85 (77M / 91M).
- Coverage comparison table contains exactly two rows — one per classification system — with compound-count and percent-coverage columns populated.
- All three baseline constants (>91M PubChem compounds, 0.12% MeSH coverage, >77M ClassyFire annotations) are recorded verbatim and traceable to the source article.

## Limitations

- The MeSH thesaurus is only partially mapped to the ClassyFire/ChemOnt taxonomy, so cross-system agreement cannot be measured compound-by-compound; only aggregate PubChem-wide coverage fractions are comparable.
- The baseline figures (>91,000,000 PubChem compounds as of June 2016, 0.12% MeSH coverage, >77 million ClassyFire annotations) are snapshot-specific and will drift as PubChem and ClassyFire grow.
- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match category description constraints in the underlying taxonomy, so automated coverage ratios can over- or under-state true class assignments for reactive chemical space.

## Evidence

- [methods] Manual MeSH baseline: "ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database."
- [methods] PubChem scale baseline: "The compounds are all included in the PubChem database."
- [abstract] ClassyFire annotation scale: "ClassyFire has been used to annotate over 77 million compounds and has already been integrated into other software packages"
- [abstract] Automated taxonomy definition: "automatically assign all known chemical compounds to a taxonomy consisting of >4800 different categories"
- [results] PubChem classification integration path: "The classification results have been submitted to the PubChem development group."
