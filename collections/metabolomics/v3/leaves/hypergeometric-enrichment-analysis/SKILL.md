---
name: hypergeometric-enrichment-analysis
description: Use when you have a list of metabolite annotations (with HMDB identifiers)
  unique to one annotation method and need to test which chemical taxonomy subclasses
  are overrepresented, comparing subset counts against a full background annotation
  set.
when_to_use_negative:
- The annotation list has no HMDB identifiers or no chemical taxonomy mapping, since
  subclass contingency tables cannot be constructed
- You need differential-abundance testing of metabolite intensities across biological
  conditions rather than overrepresentation of annotation classes
- The comparison set is not a proper subset of a defined background (exclusive annotations
  are not contained in the group's full annotation list), invalidating the hypergeometric
  assumption
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: METASPACE-ML
  role: Source of the machine-learning FDR-controlled annotations (at 10% FDR) whose
    exclusive metabolites are tested for subclass enrichment
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Rule-based predecessor engine and knowledge base providing the background
    annotation sets for the contingency tables
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:9d1740d3f2f31a8323bed1f61cb75c5356992d88b0e386ad6668f9b8dc4fb96f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hypergeometric-enrichment-analysis

## Summary

One-tailed Fisher exact (hypergeometric tail) overrepresentation testing of HMDB chemical-taxonomy subclasses among METASPACE-ML-exclusive annotations at 10% FDR, with Benjamini-Hochberg correction, used to identify which metabolite classes are better annotated by machine-learning FDR estimation in animal versus plant imaging mass spectrometry datasets.

## When to use

Use when you have a list of metabolite annotations (with HMDB identifiers) unique to one annotation method and need to test which chemical taxonomy subclasses are overrepresented, comparing subset counts against a full background annotation set.

## When NOT to use

- The annotation list has no HMDB identifiers or no chemical taxonomy mapping, since subclass contingency tables cannot be constructed
- You need differential-abundance testing of metabolite intensities across biological conditions rather than overrepresentation of annotation classes
- The comparison set is not a proper subset of a defined background (exclusive annotations are not contained in the group's full annotation list), invalidating the hypergeometric assumption

## Inputs

- List of metabolite annotations uniquely captured by METASPACE-ML at the 10% FDR threshold, split by animal vs. plant dataset groups
- Full background annotation lists for each dataset group
- HMDB identifiers for each annotated metabolite
- HMDB chemical taxonomy subclass mapping for each metabolite

## Outputs

- Results table with one row per tested HMDB subclass containing exclusive-annotation counts, background counts, odds ratio, raw p-value, and BH-adjusted q-value
- Set of significantly enriched subclasses per dataset group at q <= 0.10 (e.g., fatty acyls, carbohydrates, amino acids, glycerophospholipid sub-classes for animal; lactones, keto acids for plant)

## How to apply

Separate the METASPACE-ML-exclusive annotations at the 10% FDR threshold into animal-tissue and plant dataset groups, then map each annotated metabolite's HMDB identifier to its HMDB chemical taxonomy subclass. Define the background set as all annotations in the corresponding dataset group so each subclass's contingency table contains exclusive-annotation counts versus background counts. For every HMDB subclass run a one-tailed Fisher exact test (equivalently the hypergeometric tail probability) with scipy.stats.fisher_exact using alternative='greater', then apply Benjamini-Hochberg FDR control at the 10% level across subclass p-values. Report a table of counts, odds ratio, raw p-value, and adjusted q-value per subclass.

## Related tools

- **METASPACE-ML** (Source of the machine-learning FDR-controlled annotations (at 10% FDR) whose exclusive metabolites are tested for subclass enrichment) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Rule-based predecessor engine and knowledge base providing the background annotation sets for the contingency tables) — https://github.com/metaspace2020/metaspace

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[exclusive, bg_exclusive], [rest_exclusive, rest_bg]], alternative='greater')
```

## Evaluation signals

- Output table exists and contains exactly one row per tested HMDB subclass with all required statistical fields (counts, odds ratio, raw p-value, adjusted q-value)
- All raw p-values equal the upper-tail hypergeometric probability for the corresponding 2x2 contingency table (verify against scipy.stats.hypergeom.sf)
- Benjamini-Hochberg adjusted q-values are monotonically consistent with raw p-values and bounded between 0 and 1
- Expected enriched subclasses (fatty acyls, carbohydrates, amino acids, glycerophospholipids for animal datasets; lactones, keto acids, carboxylic acids for plant datasets) appear among rows with q <= 0.10
- Exclusive-annotation counts never exceed the corresponding background counts for any subclass

## Limitations

- MSM's sensitivity to its feature scores, particularly rho_spatial, means some annotations are captured by MSM but not METASPACE-ML, so 'exclusive' annotation sets depend on the comparison method's behavior
- Training/annotation data are biased towards public datasets represented in METASPACE (from 47 labs), so enrichment conclusions may not generalize to datasets outside that distribution
- Underrepresented data types (e.g., QTOF-based datasets, single-cell metabolomics from cultured cells) may have unreliable exclusive-annotation sets and thus underpowered enrichment tests

## Evidence

- [results] One-tailed Fisher exact overrepresentation analysis showed that fatty acyls, carbohydrates, amino acids, and most glycerophospholipid sub-classes were better annotated by METASPACE-ML in animal datasets, while lactones and keto acids were better annotated by METASPACE-ML in plant datasets.: [evidence span withheld — anchor status: not_found]
- [methods] For every HMDB subclass, apply a one-tailed Fisher exact test (equivalently the hypergeometric tail probability) using scipy.stats.fisher_exact with alternative='greater' to quantify subclass overrepresentation.: [evidence span withheld — anchor status: not_found]
- [methods] Load the list of metabolite annotations uniquely captured by METASPACE-ML at the 10% FDR threshold, separating animal-tissue datasets from plant datasets.: [evidence span withheld — anchor status: not_found]
- [abstract] METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites.: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity"
- [readme] The METASPACE platform hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites from thousands of public datasets.: "hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites"
