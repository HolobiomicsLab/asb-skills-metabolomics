---
name: pearson-correlation-feature-collapsing
description: Use when a two-condition metabolomics feature table contains redundant
  or highly correlated metabolites (e.g. |Pearson r| > 0.7) or co-classed compounds
  (e.g. BCAAs, fatty acids) that should be collapsed into single nodes before joint
  network estimation.
when_to_use_negative:
- The feature table has already been collapsed or contains few redundant features
  — the step is explicitly optional in the DNEA workflow (Step 1.2) and skipping it
  is valid.
- The data are not two-condition metabolomics/lipidomics peak intensities amenable
  to createDNEAobject(); DNEA requires raw peak intensity/concentration values for
  two experimental conditions.
- The goal is to collapse features across more than two experimental groups, since
  DNEA performs joint estimation across exactly two groups.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: DNEA R package
  role: Provides createDNEAobject() and the aggregateFeatures() function that implements
    the correlation-based, knowledge-based, and hybrid feature-collapsing modes and
    returns the collapsed_DNEAobj.
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: R
  role: Runtime environment in which the DNEA package is executed (Bioconductor install
    via BiocManager).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:2f9560a632b9bd21e3a5010524d1f0078ae41ca2bf010fe3c1acca8d41b5e4b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - R
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Pearson correlation feature collapsing

## Summary

The optional aggregateFeatures() step (DNEA Step 1.2) collapses metabolomics features that are highly correlated and/or belong to the same chemical class into singular features, producing a collapsed_DNEAobj with 134 TEDDY metabolites reduced to 130 aggregated features.

## When to use

Use when a two-condition metabolomics feature table contains redundant or highly correlated metabolites (e.g. |Pearson r| > 0.7) or co-classed compounds (e.g. BCAAs, fatty acids) that should be collapsed into single nodes before joint network estimation.

## When NOT to use

- The feature table has already been collapsed or contains few redundant features — the step is explicitly optional in the DNEA workflow (Step 1.2) and skipping it is valid.
- The data are not two-condition metabolomics/lipidomics peak intensities amenable to createDNEAobject(); DNEA requires raw peak intensity/concentration values for two experimental conditions.
- The goal is to collapse features across more than two experimental groups, since DNEA performs joint estimation across exactly two groups.

## Inputs

- DNEAobject built by createDNEAobject() from a raw peak-intensity/concentration matrix (e.g. TEDDY 134 metabolites x 322 samples) with two condition labels (DM:control / DM:case)
- feature_groups data frame mapping each metabolite name to a chemical class (e.g. 'BCAAs', 'fatty_acids', or its own name)

## Outputs

- collapsed_DNEAobj in which correlated and/or co-classed features are grouped into singular features (134 -> 130 features on TEDDY)
- feature_membership slot listing, per aggregate, the component metabolite names and mean intensity per experimental group

## How to apply

Build the DNEAobject from the peak-intensity matrix and group labels (e.g. DM:control / DM:case) with createDNEAobject(), then run aggregateFeatures() in one of three modes: correlation-based (collapse features whose absolute Pearson correlation exceeds a user-supplied correlation_threshold, e.g. 0.7), knowledge-based (aggregate features within each user-defined chemical class supplied via a feature_groups data frame, e.g. isoleucine/leucine/valine mapped to 'BCAAs' and metabolites containing 'acid' to 'fatty_acids'), or the hybrid mode combining both, which the paper's vignette demonstrates on the TEDDY data. Internally the function re-runs createDNEAobject() on the collapsed intensity matrix and returns a collapsed_DNEAobj. The rationale is to reduce redundant nodes so the downstream GLASSO-based joint network estimation is not dominated by co-measured variants of the same compound.

## Related tools

- **DNEA R package** (Provides createDNEAobject() and the aggregateFeatures() function that implements the correlation-based, knowledge-based, and hybrid feature-collapsing modes and returns the collapsed_DNEAobj.) — http://www.github.com/Karnovsky-Lab/DNEA/
- **R** (Runtime environment in which the DNEA package is executed (Bioconductor install via BiocManager).)

## Examples

```
collapsed_DNEA <- aggregateFeatures(TEDDYdat, method = "hybrid", correlation_threshold = 0.7)
```

## Evaluation signals

- The returned object is of class collapsed_DNEAobj, not the original DNEAobject.
- Feature count decreases from the input dimension (e.g. 134 TEDDY metabolites) to the reported aggregated count (130 features).
- The feature_membership slot is populated with, for every aggregate, its component metabolite names and the mean intensity per experimental group.
- In hybrid mode, both a correlation_threshold (e.g. 0.7) and the user-defined chemical classes (BCAAs, fatty_acids) influence the grouping; perturbing the threshold changes the number of collapsed nodes.
- Downstream steps (BICtune, stabilitySelection, getNetworks) accept the collapsed object without re-creating it, confirming internal re-running of createDNEAobject().

## Limitations

- The correlation threshold is user-supplied, so the collapsed network composition depends on that choice; the demonstrated value is 0.7.
- Feature aggregation is optional and positioned as Step 1.2 only — it does not replace model tuning (BICtune), stability selection, network estimation (getNetworks), or filtering (filterNetworks, e.g. pcor = 0.166).
- Knowledge-based aggregation is only as good as the user-defined feature_groups annotation; metabolites outside defined classes (mapped to their own name) remain uncollapsed.

## Evidence

- [methods] For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation.: "For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation."
- [intro] The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation: "The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation"
- [readme] The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions and uses the GLASSO algorithm to estimate metabolite-metabolite partial correlations.: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions"
- [other] aggregateFeatures() outputs a collapsed_DNEAobj in which features that are highly correlated and/or of the same chemical class are grouped into singular features.: [evidence span withheld — anchor status: not_found]
