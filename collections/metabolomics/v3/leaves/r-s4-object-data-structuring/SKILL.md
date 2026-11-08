---
name: r-s4-object-data-structuring
description: Use when implementing or reconstructing an R Bioconductor workflow (e.g.
  DNEA) that must hold unnormalized peak intensity data, group labels, hyperparameters,
  and feature aggregation metadata in typed S4 objects with defined slots.
when_to_use_negative:
- The data are not from an R/Bioconductor pipeline and no S4 class system is available
  — a plain data.frame or language-native container is appropriate
- The input is already a network edge list or downstream analysis output rather than
  raw peak intensities for two experimental conditions
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DNEA R package
  role: 'Provides the S4 class infrastructure: createDNEAobject() constructs the DNEAobject
    and aggregateFeatures() returns the collapsed_DNEAobj with a feature_membership
    slot'
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: R
  role: Language providing the S4 object system in which DNEAobject and collapsed_DNEAobj
    are defined and persisted via saveRDS()
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:af22aa57d9db36fe9406828bc297ebd3b4b107b6cf1bc433527bac8df24d291c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# r-s4-object-data-structuring

## Summary

Structures metabolomics expression data and its transformations into formal R S4 objects, following the DNEA package's design where createDNEAobject() builds a DNEAobject from raw peak intensities and aggregateFeatures() returns a collapsed_DNEAobj with a populated feature_membership slot.

## When to use

Use when implementing or reconstructing an R Bioconductor workflow (e.g. DNEA) that must hold unnormalized peak intensity data, group labels, hyperparameters, and feature aggregation metadata in typed S4 objects with defined slots.

## When NOT to use

- The data are not from an R/Bioconductor pipeline and no S4 class system is available — a plain data.frame or language-native container is appropriate
- The input is already a network edge list or downstream analysis output rather than raw peak intensities for two experimental conditions

## Inputs

- Unnormalized peak intensity / concentration expression matrix for features from two experimental conditions (e.g. TEDDY metabolomics, 134 metabolites x 322 samples)
- Group labels for the two conditions (e.g. DM:control / DM:case)
- Optional feature_groups data frame mapping metabolites to chemical classes (e.g. 'BCAAs', 'fatty_acids')
- Aggregation parameters: method ('correlation', 'knowledge', 'hybrid') and correlation_threshold (e.g. 0.7)

## Outputs

- DNEAobject (S4) holding expression data and group labels
- collapsed_DNEAobj (S4) with aggregated features and populated feature_membership slot
- Persisted R object file (e.g. collapsed_DNEA.rds)

## How to apply

Wrap the input expression matrix (e.g. the TEDDY metabolomics data, 134 metabolites x 322 samples with group labels DM:control / DM:case) into a DNEAobject via createDNEAobject(), which stores raw peak intensity/concentration values for two experimental conditions. When optional feature aggregation is performed (correlation-based, knowledge-based, or hybrid), the function re-runs createDNEAobject() internally and outputs a collapsed_DNEAobj, e.g. collapsing features whose absolute Pearson correlation exceeds a correlation_threshold such as 0.7 and/or metabolites of the same chemical class (BCAAs, fatty_acids) into singular features. Populate the feature_membership slot with, for every aggregate, its component metabolite names and the mean intensity per experimental group, then persist the object with saveRDS() (e.g. collapsed_DNEA.rds). Keep the S4 slots modular so downstream steps (BICtune, stabilitySelection, getNetworks, filterNetworks) operate on the same typed object.

## Related tools

- **DNEA R package** (Provides the S4 class infrastructure: createDNEAobject() constructs the DNEAobject and aggregateFeatures() returns the collapsed_DNEAobj with a feature_membership slot) — http://www.github.com/Karnovsky-Lab/DNEA/
- **R** (Language providing the S4 object system in which DNEAobject and collapsed_DNEAobj are defined and persisted via saveRDS())

## Examples

```
collapsed_DNEA <- aggregateFeatures(TEDDYdat, method = "hybrid", correlation_threshold = 0.7); saveRDS(collapsed_DNEA, "collapsed_DNEA.rds")
```

## Evaluation signals

- The collapsed_DNEAobj feature count matches the reported reduction from 134 to 130 features on the TEDDY metabolomics data
- The feature_membership slot lists, for every aggregate, its component metabolite names and the mean intensity per experimental group
- createDNEAobject() is re-run internally by aggregateFeatures(), so the collapsed object is itself a valid DNEAobject downstream (usable by BICtune/stabilitySelection)
- The expression matrix passed in retains unnormalized peak intensity values for two experimental conditions (not log-normalized or subsampled data)

## Limitations

- Feature aggregation is an optional step (Step 1.2); collapsing features discards per-metabolite resolution, so downstream differential network results depend on the aggregation choice
- Hybrid aggregation combines correlation- and knowledge-based grouping, so poorly curated feature_groups assignments (e.g. any metabolite whose name contains 'acid') will propagate errors into the collapsed object

## Evidence

- [methods] For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation.: "For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation."
- [intro] The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation: "The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation"
- [readme] The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions and uses the GLASSO algorithm to estimate metabolite-metabolite partial correlations.: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions"
- [methods] The DNEA R package is available to download on GitHub (http://www.github.com/Karnovsky-Lab/DNEA/).: "The DNEA R package is available to download on GitHub (http://www.github.com/Karnovsky-Lab/DNEA/)."
