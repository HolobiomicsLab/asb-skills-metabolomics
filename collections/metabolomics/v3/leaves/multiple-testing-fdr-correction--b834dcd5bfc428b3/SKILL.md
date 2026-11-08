---
name: multiple-testing-fdr-correction
description: Use when testing many metabolite classes/subclasses simultaneously for
  overrepresentation among a subset of annotations (e.g. METASPACE-ML-exclusive hits)
  and you need multiplicity-controlled q-values rather than raw p-values.
when_to_use_negative:
- Input is a single pre-computed p-value or a one-off hypothesis test — BH correction
  across tests is meaningless without a family of subclass-level tests
- Annotations were produced by a different engine than METASPACE/METASPACE-ML or without
  an FDR-controlled annotation threshold, since the 10% FDR exclusive-annotation subset
  is the skill's basis
- You need per-annotation significance rather than class/subclass overrepresentation
  — this skill tests HMDB subclasses, not individual ions
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: METASPACE-ML
  role: Source of the machine-learning, FDR-controlled metabolite annotations whose
    exclusive hits at 10% FDR form the tested set
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Engine/knowledgebase providing the background annotations and public imaging
    MS datasets used to build contingency tables
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:aab84dba322d4773486f7213dbd70c6b87902a499a07285fa6238636b3993d6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multiple-testing-fdr-correction

## Summary

Applies a one-tailed Fisher exact (hypergeometric tail) test per HMDB chemical taxonomy subclass over METASPACE-ML-exclusive annotations at 10% FDR, then controls subclass-level p-values with Benjamini-Hochberg FDR at the 10% level to identify enriched metabolite classes in animal vs plant imaging MS datasets.

## When to use

Use when testing many metabolite classes/subclasses simultaneously for overrepresentation among a subset of annotations (e.g. METASPACE-ML-exclusive hits) and you need multiplicity-controlled q-values rather than raw p-values.

## When NOT to use

- Input is a single pre-computed p-value or a one-off hypothesis test — BH correction across tests is meaningless without a family of subclass-level tests
- Annotations were produced by a different engine than METASPACE/METASPACE-ML or without an FDR-controlled annotation threshold, since the 10% FDR exclusive-annotation subset is the skill's basis
- You need per-annotation significance rather than class/subclass overrepresentation — this skill tests HMDB subclasses, not individual ions

## Inputs

- List of metabolite annotations uniquely captured by METASPACE-ML at the 10% FDR threshold, separated into animal-tissue and plant dataset groups
- HMDB identifiers for all annotated metabolites in each dataset group
- HMDB chemical taxonomy subclass mapping (per metabolite)
- Background set of all annotations in the corresponding dataset group

## Outputs

- Results table with one row per tested HMDB subclass containing exclusive-annotation counts, background counts, odds ratio, raw p-value, and Benjamini-Hochberg adjusted q-value
- Set of significant subclasses (q <= 0.10) per dataset group, e.g. fatty acyls, carbohydrates, amino acids and glycerophospholipid subclasses (animal) and lactones, keto acids, carboxylic acids (plant)

## How to apply

Map each annotated metabolite's HMDB identifier to its HMDB chemical taxonomy subclass and split annotations by dataset group (animal-tissue vs plant). For every HMDB subclass, build a 2x2 contingency table of exclusive-annotation counts versus background counts (background = all annotations in the corresponding dataset group) and run a one-tailed Fisher exact test (equivalently the hypergeometric tail probability) with scipy.stats.fisher_exact using alternative='greater' to quantify subclass overrepresentation. Record each subclass's counts, odds ratio, and raw p-value. Correct all subclass-level p-values for multiple testing with Benjamini-Hochberg FDR control at the 10% level, and call a subclass enriched only when its adjusted q-value <= 0.10. Rationale: METASPACE-ML provides FDR-controlled metabolite annotation in imaging MS, so class-level enrichment must itself be FDR-controlled to match the paper's statistical rigor.

## Related tools

- **METASPACE-ML** (Source of the machine-learning, FDR-controlled metabolite annotations whose exclusive hits at 10% FDR form the tested set) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Engine/knowledgebase providing the background annotations and public imaging MS datasets used to build contingency tables) — https://github.com/metaspace2020/metaspace

## Examples

```
from scipy.stats import fisher_exact; from statsmodels.stats.multitest import multipletests; odds, p = fisher_exact([[excl, excl_bg],[tot-excl, tot-bg]], alternative='greater'); q = multipletests(pvals, alpha=0.10, method='fdr_bh')[1]
```

## Evaluation signals

- Output table contains one row per tested HMDB subclass with all required statistical fields (counts, odds ratio, raw p-value, adjusted q-value)
- All reported p-values were computed with a one-tailed Fisher exact test using alternative='greater' (hypergeometric tail), not a two-tailed test
- Adjusted q-values equal Benjamini-Hochberg FDR values computed across the full family of subclass tests, not raw p-values
- Expected enriched subclasses (fatty acyls, carbohydrates, amino acids, glycerophospholipids for animal datasets; lactones, keto acids, carboxylic acids for plant datasets) appear among significant rows at q <= 0.10
- Exclusive counts in each contingency table are <= the corresponding background counts for the same subclass and dataset group

## Limitations

- Enrichment calls depend on the METASPACE-ML annotation threshold (10% FDR); MSM's sensitivity to the rho_spatial score can capture annotations missed by METASPACE-ML, affecting the exclusive set
- Decoy ion heterogeneity in similarity to target ions means FDR estimation itself needs further investigation, which propagates uncertainty into class-level tests
- Training/evaluation data are biased towards public datasets represented in METASPACE (from 47 labs), limiting generalization of subclass enrichments to underrepresented data types such as QTOF or single-cell metabolomics

## Evidence

- [abstract] METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation.: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [intro] We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity.: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds"
- [other] One-tailed Fisher exact overrepresentation analysis showed that fatty acyls, carbohydrates, amino acids, and most glycerophospholipid sub-classes were better annotated by METASPACE-ML in animal datasets: [evidence span withheld — anchor status: not_found]
- [readme] The METASPACE platform is developed by software engineers, data scientists and mass spectrometrists from the Alexandrov team at UCSD.: "The METASPACE platform is developed by software engineers, data scientists and mass spectrometrists from the Alexandrov team at UCSD."
