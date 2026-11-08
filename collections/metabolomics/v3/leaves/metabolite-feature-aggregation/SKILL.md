---
name: metabolite-feature-aggregation
description: Use when a metabolomics/lipidomics feature table contains highly correlated
  or chemically related features (e.g. BCAAs, fatty acids) that should be collapsed
  into single nodes before DNEA network estimation.
when_to_use_negative:
- Input is already a collapsed/reduced feature table with no redundant correlated
  or same-class metabolites — aggregation is an optional step and can be skipped
- No chemical-class annotation is available and correlated-feature collapsing is not
  desired (i.e. neither knowledge-based nor correlation-based grouping is justified)
- Data are not metabolomics/lipidomics-style feature intensities from two experimental
  conditions (DNEA is built for two-group differential network analysis)
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0203
tools:
- name: DNEA R package
  role: Provides aggregateFeatures() for correlation-based, knowledge-based, and hybrid
    feature aggregation into a collapsed_DNEAobj
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:45a08552c9bb39db7bcf890074c072e1677831786f1972a4f4b855dce07dce3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolite-feature-aggregation

## Summary

An optional step in DNEA Step 1.2 that collapses highly correlated and/or same-class metabolite features into singular nodes via aggregateFeatures(), producing a collapsed_DNEAobj. Reduces feature redundancy before network estimation (e.g. TEDDY metabolomics: 134 → 130 features).

## When to use

Use when a metabolomics/lipidomics feature table contains highly correlated or chemically related features (e.g. BCAAs, fatty acids) that should be collapsed into single nodes before DNEA network estimation.

## When NOT to use

- Input is already a collapsed/reduced feature table with no redundant correlated or same-class metabolites — aggregation is an optional step and can be skipped
- No chemical-class annotation is available and correlated-feature collapsing is not desired (i.e. neither knowledge-based nor correlation-based grouping is justified)
- Data are not metabolomics/lipidomics-style feature intensities from two experimental conditions (DNEA is built for two-group differential network analysis)

## Inputs

- DNEAobject created by createDNEAobject() from a raw peak-intensity matrix (features x samples) with two experimental-group labels
- feature_groups data frame mapping metabolite names to chemical classes (knowledge-based / hybrid modes)
- correlation_threshold for absolute Pearson correlation (correlation-based / hybrid modes)

## Outputs

- collapsed_DNEAobj in which correlated and/or same-class features are grouped into singular features
- feature_membership slot listing, for every aggregate, its component metabolite names and mean intensity per experimental group

## How to apply

After building the DNEAobject with createDNEAobject() (e.g. TEDDY metabolomics expression data, 134 metabolites x 322 samples, group labels DM:control / DM:case), optionally run aggregateFeatures() to collapse redundant features. Three modes are available: correlation-based (collapse features whose absolute Pearson correlation exceeds a user-supplied correlation_threshold), knowledge-based (aggregate features within user-defined chemical classes supplied as a feature_groups data frame, e.g. isoleucine/leucine/valine assigned to 'BCAAs'), and hybrid (both combined), with 'hybrid' used in the paper's demonstration. Internally, createDNEAobject() is re-run on the collapsed feature set, yielding a collapsed_DNEA object whose feature_membership slot records each aggregate's component metabolite names and mean intensity per experimental group. Rationale: collapsing redundant features reduces the p >> n burden on subsequent GLASSO-based joint network estimation.

## Related tools

- **DNEA R package** (Provides aggregateFeatures() for correlation-based, knowledge-based, and hybrid feature aggregation into a collapsed_DNEAobj) — http://www.github.com/Karnovsky-Lab/DNEA/

## Examples

```
collapsed_DNEA <- aggregateFeatures(object = TEDDYdat, method = "hybrid", correlation_threshold = 0.7)
```

## Evaluation signals

- Output object is of class collapsed_DNEAobj and a collapsed_DNEA object results from the internal re-run of createDNEAobject()
- Feature count is reduced as expected (TEDDY demonstration: from 134 features to 130)
- feature_membership slot is populated with component metabolite names and per-group mean intensity for every aggregate
- Under the hybrid mode with correlation_threshold = 0.7, each collapsed node corresponds either to features with |Pearson correlation| > 0.7 or to members of the same user-defined chemical class
- Downstream DNEA steps (BICtune, stabilitySelection, getNetworks, filterNetworks) accept the collapsed object without errors

## Limitations

- Correlation-based collapsing requires a user-supplied correlation threshold; an inappropriate threshold may over- or under-collapse features
- Knowledge-based aggregation is limited by the quality of the user-provided chemical-class assignments (e.g. 'BCAAs', 'fatty_acids'); unknown metabolites may each remain as their own singleton feature
- Aggregation is optional and demonstrated for illustration ('For demonstration purposes'); skipping it is a valid choice for suitable datasets

## Evidence

- [methods] For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation.: "For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation."
- [intro] The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation: "The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation"
- [abstract] We developed the DNEA R package for the data-driven network analysis of metabolomics data.: "We developed the DNEA R package for the data-driven network analysis of metabolomics data."
- [readme] The package was specifically built for metabolomics and lipidomics data, but can be used on any approximately normal data sets.: "The package was specifically built for metabolomics and lipidomics data, but can be used on any approximately normal data sets."
