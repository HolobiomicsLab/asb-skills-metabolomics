---
name: hmdb-subclass-annotation-mapping
description: Use when you have a list of metabolite annotations uniquely captured
  by METASPACE-ML at the 10% FDR threshold (split by animal vs. plant datasets) and
  need to identify which HMDB chemical taxonomy subclasses are significantly overrepresented.
when_to_use_negative:
- When comparing annotations from METASPACE-ML against its rule-based MSM predecessor
  at the level of individual ions rather than chemical classes — the underlying overrepresentation
  logic does not apply
- When annotations lack valid HMDB identifiers or HMDB chemical taxonomy subclass
  mappings, since the contingency tables cannot be constructed
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: METASPACE-ML
  role: Machine-learning annotation engine whose exclusive, FDR-controlled annotations
    (at 10% FDR) form the foreground set for subclass enrichment
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Hosts the annotations database and knowledge base from which exclusive and
    background annotation lists are queried
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:2d2210feafda6a134494126157163ec4b45e07bff79c113e4adbe5d3f0b9d799
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hmdb-subclass-annotation-mapping

## Summary

Maps METASPACE-ML-exclusive metabolite annotations (identified at 10% FDR in imaging MS datasets) to HMDB chemical taxonomy subclasses and tests each subclass for overrepresentation with a one-tailed Fisher exact test, comparing animal-tissue versus plant datasets.

## When to use

Use when you have a list of metabolite annotations uniquely captured by METASPACE-ML at the 10% FDR threshold (split by animal vs. plant datasets) and need to identify which HMDB chemical taxonomy subclasses are significantly overrepresented.

## When NOT to use

- When comparing annotations from METASPACE-ML against its rule-based MSM predecessor at the level of individual ions rather than chemical classes — the underlying overrepresentation logic does not apply
- When annotations lack valid HMDB identifiers or HMDB chemical taxonomy subclass mappings, since the contingency tables cannot be constructed

## Inputs

- List of metabolite annotations uniquely captured by METASPACE-ML at 10% FDR, separated into animal-tissue and plant dataset groups
- HMDB identifiers for each annotated metabolite with HMDB chemical taxonomy subclass mappings
- Background set of all annotations per dataset group (animal / plant)

## Outputs

- Results table with one row per tested HMDB subclass: exclusive vs. background counts, odds ratio, raw p-value, Benjamini-Hochberg adjusted q-value
- Set of significantly enriched subclasses per group (q <= 0.10): fatty acyls, carbohydrates, amino acids, glycerophospholipid subclasses for animal datasets; lactones, keto acids, carboxylic acids for plant datasets

## How to apply

Load the METASPACE-ML-exclusive annotations at the 10% FDR threshold, keeping animal-tissue and plant dataset groups separate. Map each annotated metabolite's HMDB identifier to its HMDB chemical taxonomy subclass. For each subclass, build a contingency table contrasting exclusive-annotation counts against the background of all annotations in the same dataset group, then run a one-tailed Fisher exact test (hypergeometric tail, scipy.stats.fisher_exact with alternative='greater') to quantify overrepresentation. Correct subclass-level p-values with Benjamini-Hochberg FDR control at the 10% level before declaring a subclass enriched.

## Related tools

- **METASPACE-ML** (Machine-learning annotation engine whose exclusive, FDR-controlled annotations (at 10% FDR) form the foreground set for subclass enrichment) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Hosts the annotations database and knowledge base from which exclusive and background annotation lists are queried) — https://github.com/metaspace2020/metaspace

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[n_exclusive, n_bg_exclusive], [n_subclass_rest, n_bg_rest]], alternative='greater')
```

## Evaluation signals

- Output table exists and contains one row per tested HMDB subclass with counts, odds ratio, raw p-value, and adjusted q-value
- All reported p-values have been Benjamini-Hochberg corrected and significant rows satisfy q <= 0.10
- Expected enriched subclasses appear among significant rows: fatty acyls, carbohydrates, amino acids and glycerophospholipid subclasses for animal datasets; lactones, keto acids and carboxylic acids for plant datasets
- Contingency table background equals the full annotation set of the corresponding dataset group (no leakage between animal and plant groups)
- Fisher exact tests use alternative='greater' (one-tailed overrepresentation), not the default two-tailed test

## Limitations

- The analysis inherits the bias of METASPACE-ML's training data, which was 'biased towards public datasets represented in METASPACE (from 47 labs)'
- Annotations captured by the rule-based MSM but missed by METASPACE-ML exist because 'MSM's sensitivity to its feature scores, particularly the rho_spatial score' — so 'exclusive' annotations are model-dependent
- Underrepresented data types (e.g., QTOF-based datasets, single-cell metabolomics from cultured cells, industrial lab data) limit generalizability of enrichment patterns

## Evidence

- [abstract] METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites.: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites."
- [intro] we introduce METASPACE-ML, a machine-learning model for FDR-controlled metabolite annotation within specific contexts in imaging MS: "we introduce METASPACE-ML, a machine-learning model for FDR-controlled metabolite annotation within specific contexts in imaging MS"
- [readme] A Python library and set of example Jupyter notebooks for performing data analysis on the annotations database: [evidence span withheld — anchor status: external_doc]
