---
name: fisher-exact-overrepresentation-testing
description: Use when you have a list of metabolite annotations exclusive to one method
  (e.g. METASPACE-ML at FDR 10%) plus a background annotation set, each mapped to
  HMDB taxonomy subclasses, and need to find which metabolic classes are overrepresented.
when_to_use_negative:
- The annotation list is not mapped to HMDB chemical taxonomy subclasses, so no contingency
  table per subclass can be built
- You want underrepresentation or two-sided class differences rather than one-tailed
  overrepresentation of a method's exclusive annotations
- You are comparing annotation counts rather than set membership overrepresentation
  — this test models exclusive-vs-background counts, not fold changes
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: METASPACE-ML
  role: Source of the machine-learning annotations at 10% FDR whose exclusive metabolites
    form the enriched (foreground) set
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform/knowledge base providing the rule-based (MSM) annotations and public
    datasets used as the background annotation set
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:ad3b817870297e609630563d9356e8969026865eea5668a53e99a14c32ae4224
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# fisher-exact-overrepresentation-testing

## Summary

A one-tailed Fisher exact (hypergeometric tail) overrepresentation test, with Benjamini-Hochberg FDR correction at 10%, used to identify HMDB chemical-taxonomy subclasses enriched among metabolite annotations uniquely captured by METASPACE-ML versus the full annotation background in animal and plant imaging mass spectrometry datasets.

## When to use

Use when you have a list of metabolite annotations exclusive to one method (e.g. METASPACE-ML at FDR 10%) plus a background annotation set, each mapped to HMDB taxonomy subclasses, and need to find which metabolic classes are overrepresented.

## When NOT to use

- The annotation list is not mapped to HMDB chemical taxonomy subclasses, so no contingency table per subclass can be built
- You want underrepresentation or two-sided class differences rather than one-tailed overrepresentation of a method's exclusive annotations
- You are comparing annotation counts rather than set membership overrepresentation — this test models exclusive-vs-background counts, not fold changes

## Inputs

- List of metabolite annotations uniquely captured by METASPACE-ML at the 10% FDR threshold, tagged by animal-tissue vs plant dataset membership
- Full annotation list per dataset group (background set) from METASPACE/METASPACE-ML
- HMDB identifiers for annotated metabolites
- HMDB chemical taxonomy classification mapping HMDB IDs to subclasses

## Outputs

- Per-subclass enrichment table with exclusive-annotation counts, background counts, odds ratio, raw p-value, and BH-adjusted q-value
- Set of significant enriched subclasses (q <= 0.10) for animal datasets (e.g. fatty acyls, carbohydrates, amino acids, glycerophospholipid sub-classes) and plant datasets (e.g. lactones, keto acids)

## How to apply

Split METASPACE-ML-exclusive annotations (unique versus the rule-based MSM approach at 10% FDR) into animal-tissue and plant dataset groups. Map each annotated metabolite's HMDB identifier to its HMDB chemical taxonomy subclass and define the background as all annotations in the corresponding dataset group, building a 2x2 contingency table (exclusive counts vs background counts) per subclass. For every HMDB subclass run a one-tailed Fisher exact test via scipy.stats.fisher_exact with alternative='greater' — equivalent to the hypergeometric tail probability — to quantify overrepresentation. Correct subclass-level p-values with Benjamini-Hochberg FDR control at the 10% level, then report counts, odds ratio, raw p-value, and adjusted q-value per subclass.

## Related tools

- **METASPACE-ML** (Source of the machine-learning annotations at 10% FDR whose exclusive metabolites form the enriched (foreground) set) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform/knowledge base providing the rule-based (MSM) annotations and public datasets used as the background annotation set) — https://github.com/metaspace2020/metaspace

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[exclusive, exclusive_bg], [background, background_bg]], alternative='greater')
```

## Evaluation signals

- Output table exists with one row per tested HMDB subclass and required fields: counts, odds ratio, raw p-value, adjusted q-value
- All reported q-values are monotone with the Benjamini-Hochberg adjustment of the raw p-values and the significance cut-off is q <= 0.10
- Enriched subclasses expected from the paper appear among significant rows: fatty acyls, carbohydrates, amino acids and glycerophospholipid sub-classes for animal datasets; lactones and keto acids for plant datasets
- Contingency table counts per subclass are consistent with the exclusive-annotation list versus the full background set for the correct dataset group (animal vs plant)
- Odds ratios for significant subclasses are greater than 1, consistent with the one-tailed alternative='greater' test

## Limitations

- Exclusive annotations are defined relative to the rule-based MSM approach; some annotations were captured by MSM but not by METASPACE-ML due to MSM's sensitivity to the rho_spatial score, so the exclusive set is not exhaustive
- Significance is judged at the 10% FDR level, a lenient threshold relative to conventional 5%
- Background composition depends on which public datasets are represented in the METASPACE knowledge base; training/evaluation selection is biased towards datasets from 47 labs and underrepresents QTOF and less common data types

## Evidence

- [results] One-tailed Fisher exact overrepresentation analysis showed that fatty acyls, carbohydrates, amino acids, and most glycerophospholipid sub-classes were better annotated by METASPACE-ML in animal datasets, while lactones and keto acids were better annotated by METASPACE-ML in plant datasets.: "fatty acyls, carbohydrates, amino acids, and most glycerophospholipid sub-classes were better annotated by METASPACE-ML in animal datasets"
- [methods] For every HMDB subclass, apply a one-tailed Fisher exact test (equivalently the hypergeometric tail probability) using scipy.stats.fisher_exact with alternative='greater' to quantify subclass overrepresentation.: [evidence span withheld — anchor status: not_found]
- [methods] Correct subclass-level p-values for multiple testing with Benjamini-Hochberg FDR control at the 10% level.: [evidence span withheld — anchor status: not_found]
- [abstract] METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites.: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity"
- [readme] The METASPACE platform hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites from thousands of public datasets provided by the community.: "a spatial metabolite knowledgebase of the metabolites from thousands of public datasets provided by the community"
