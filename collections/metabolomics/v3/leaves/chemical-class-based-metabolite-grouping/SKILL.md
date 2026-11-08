---
name: chemical-class-based-metabolite-grouping
description: Use when a metabolomics/lipidomics feature table contains redundant or
  chemically related features (e.g. correlated isomers, shared chemical class) that
  should be collapsed into single nodes prior to DNEA network construction.
when_to_use_negative:
- The feature table is already reduced or contains only distinct, chemically unrelated
  metabolites with no correlated duplicate features
- You need every individual metabolite as a separate network node for downstream pathway
  or ID-level interpretation, since aggregation merges features into singular nodes
- The data are not from two experimental conditions with approximately normal peak-intensity
  values, which DNEA's joint network estimation requires
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DNEA R package
  role: Provides createDNEAobject() and aggregateFeatures() implementing the knowledge-based,
    correlation-based, and hybrid feature aggregation modes that yield the collapsed_DNEAobj
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: R
  role: Runtime environment in which the DNEA aggregation workflow is executed and
    the collapsed object saved as .rds
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:1dc742bc597b8df08c6435b06cfcaa5934504d1c747f73a8324bd106b1935660
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - R
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chemical-class-based-metabolite-grouping

## Summary

The aggregateFeatures() step of the DNEA R package (Step 1.2) collapses metabolomic features that are highly correlated and/or belong to the same chemical class into singular features, producing a collapsed_DNEAobj that reduces network dimensionality before joint estimation.

## When to use

Use when a metabolomics/lipidomics feature table contains redundant or chemically related features (e.g. correlated isomers, shared chemical class) that should be collapsed into single nodes prior to DNEA network construction.

## When NOT to use

- The feature table is already reduced or contains only distinct, chemically unrelated metabolites with no correlated duplicate features
- You need every individual metabolite as a separate network node for downstream pathway or ID-level interpretation, since aggregation merges features into singular nodes
- The data are not from two experimental conditions with approximately normal peak-intensity values, which DNEA's joint network estimation requires

## Inputs

- DNEAobject built via createDNEAobject() from a raw peak-intensity/concentration feature table (e.g. TEDDY metabolomics data, 134 metabolites x 322 samples, two groups such as DM:control / DM:case)
- feature_groups data frame mapping each feature name to a chemical class label
- user-supplied correlation_threshold for correlation-based or hybrid aggregation (e.g. 0.7)

## Outputs

- collapsed_DNEAobj (e.g. collapsed_DNEA.rds) with features reduced (TEDDY example: 134 -> 130 features)
- feature_membership slot listing, per aggregate, the component metabolite names and mean intensity per experimental group

## How to apply

Build a feature_groups data frame assigning each metabolite to a user-defined chemical class (e.g. in the TEDDY example, isoleucine/leucine/valine assigned to 'BCAAs', any metabolite containing 'acid' to 'fatty_acids', remaining metabolites to their own name). Choose an aggregation mode: knowledge-based (collapse within each chemical class), correlation-based (collapse features whose absolute Pearson correlation exceeds a user-supplied correlation_threshold), or hybrid (both). Run aggregateFeatures() with the chosen method (e.g. method='hybrid', correlation_threshold=0.7) on the DNEAobject; the function re-runs createDNEAobject() internally and populates the feature_membership slot with, for every aggregate, its component metabolite names and the mean intensity per experimental group. The rationale is to reduce the p >> n problem and merge redundant signals before GLASSO-based joint network estimation across the two experimental groups.

## Related tools

- **DNEA R package** (Provides createDNEAobject() and aggregateFeatures() implementing the knowledge-based, correlation-based, and hybrid feature aggregation modes that yield the collapsed_DNEAobj) — http://www.github.com/Karnovsky-Lab/DNEA/
- **R** (Runtime environment in which the DNEA aggregation workflow is executed and the collapsed object saved as .rds)

## Examples

```
collapsed_DNEA <- aggregateFeatures(object = TEDDYdat, method = 'hybrid', correlation_threshold = 0.7, feature_groups = feature_groups)
```

## Evaluation signals

- The output object is of class collapsed_DNEAobj, not the original DNEAobject
- Feature count decreases as expected (TEDDY validation: 134 -> 130 features with hybrid aggregation)
- feature_membership slot is populated: every aggregate lists its component metabolite names and mean intensity per experimental group
- All component features of an aggregate satisfy the chosen criterion (same feature_groups class label, and/or absolute Pearson correlation above the supplied correlation_threshold)
- The collapsed object can be passed directly into the next DNEA step (BICtune / stabilitySelection) without re-instantiation errors

## Limitations

- Chemical-class assignment is fully user-supplied via feature_groups, so aggregation quality depends on correct class annotation of each metabolite
- For demonstration purposes the paper used the 'hybrid' approach; the correlation-based mode requires a user-chosen correlation_threshold whose value affects how aggressively features merge
- Collapsing features loses per-metabolite resolution, which may complicate later interpretation of individual metabolite contributions

## Evidence

- [methods] aggregateFeatures() outputs a collapsed_DNEAobj in which features that are highly correlated and/or of the same chemical class are grouped into singular features.: [evidence span withheld — anchor status: not_found]
- [methods] For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation.: "For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation."
- [intro] The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation: "The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation"
- [readme] The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions"
