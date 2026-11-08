---
name: conservative-fold-change-aggregation
description: Use when a broad ontology-backed comparison (e.g. 'Digestive Tract' vs
  'Bacteria') expands into multiple metadata-combination contrasts and each compound
  needs one aggregated p-value and one fold-change instead of many pairwise results.
when_to_use_negative:
- When the comparison is a simple pairwise metadata contrast with a single triad pair
  — no aggregation across n*m pairs is needed
- For biological metadata combinations with few samples and few studies, where the
  article warns quantitative comparisons are less robust
- When compounds have not been pre-processed through the BinBase pipeline (FAME normalization,
  QC removal, imputation of presence-percentage times minimum detected value) — aggregation
  on uncurated intensities is not supported
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: SciPy
  role: Provides the statistics routines for the per-pair Welch t-tests on log-transformed
    metabolite intensities whose p-values and fold-changes are aggregated
- name: NetworkX
  role: Builds the ontology 'belongs to' graph that expands broad query terms into
    the n*m metadata-combination pairs before aggregation
- name: Python
  role: Custom Python scripts implement the OGDA aggregation (max-p / min-fold-change
    rule) over the pairwise test outputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:e285ffc51610ea7278840e9cba03d01378cb9e6d38e8cebcf93f5d4a24cb6dfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinDiscover
  - NetworkX
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# conservative-fold-change-aggregation

## Summary

A conservative aggregation rule for OGDA meta-analyses that collapses the n*m pairwise Welch t-test results (p-values and fold-changes on log-transformed GC–TOF MS metabolite intensities) into a single p-value and fold-change per compound per query, deliberately reporting the least favorable evidence.

## When to use

Use when a broad ontology-backed comparison (e.g. 'Digestive Tract' vs 'Bacteria') expands into multiple metadata-combination contrasts and each compound needs one aggregated p-value and one fold-change instead of many pairwise results.

## When NOT to use

- When the comparison is a simple pairwise metadata contrast with a single triad pair — no aggregation across n*m pairs is needed
- For biological metadata combinations with few samples and few studies, where the article warns quantitative comparisons are less robust
- When compounds have not been pre-processed through the BinBase pipeline (FAME normalization, QC removal, imputation of presence-percentage times minimum detected value) — aggregation on uncurated intensities is not supported

## Inputs

- Two broad query terms resolved to sets of metadata combinations (organ/species/disease triads) via ontology graphs (MeSH, NCBI taxonomy)
- Log-transformed metabolite intensity distributions per metadata triad from the BinBase-derived BinDiscover dataset (156,174 samples, 18,290 Bins)
- Per-pair Welch t-test p-value and fold-change matrices (n*m pairs) from SciPy

## Outputs

- One aggregated p-value per compound per OGDA query (maximum over n*m pairs)
- One aggregated fold-change per compound per OGDA query (minimum over n*m pairs, or 0 if significant pairs have conflicting signs)
- Aggregated results table (e.g. ogda_results.csv) feeding OGDA graph visualizations

## How to apply

First expand the two broad query terms into their constituent metadata triads (organ/species/disease combinations) using the ontology 'belongs to' graph built in NetworkX, excluding terms from unrelated ontology branches. For each of the n*m pairs across the two groups, compute Welch t-tests on log-transformed metabolite intensity distributions with SciPy statistics routines, yielding an n*m matrix of p-values and fold-changes per compound. Aggregate per compound using the paper's conservative rule: retain the maximum p-value, retain the minimum fold-change, and set fold-change to 0 when significant pairs show fold-changes of both signs. The conservatism guards against inflating significance from multiple correlated sub-comparisons and against reporting a directional effect that is inconsistent across the expanded contrasts. Applied as part of BinDiscover's ontologically grouped differential analysis (OGDA), which combines the smaller statistical outputs into graph summaries.

## Related tools

- **SciPy** (Provides the statistics routines for the per-pair Welch t-tests on log-transformed metabolite intensities whose p-values and fold-changes are aggregated)
- **NetworkX** (Builds the ontology 'belongs to' graph that expands broad query terms into the n*m metadata-combination pairs before aggregation)
- **Python** (Custom Python scripts implement the OGDA aggregation (max-p / min-fold-change rule) over the pairwise test outputs)

## Examples

```
from scipy import stats; pvals = [stats.ttest_ind(log_intens[grp_a], log_intens[grp_b], equal_var=False).pvalue for grp_a, grp_b in triad_pairs]; agg_p = max(pvals)
```

## Evaluation signals

- ogda_results.csv contains exactly one aggregated p-value and one aggregated fold-change per compound per query
- For each compound, the reported p-value equals the maximum p-value across all n*m significant pairwise Welch t-tests
- For each compound, the reported fold-change equals the minimum fold-change across significant pairs, and equals 0 when significant pairs show both positive and negative fold-changes
- Validation with a synthetic metadata-annotated intensity fixture reproduces the expected max-p / min-fold-change aggregation
- Terms from unrelated ontology branches (e.g. stool mapping to 'fluids and secretions' rather than digestive system) are excluded before pair generation, so no cross-branch pairs contribute to the aggregation

## Limitations

- Quantitative comparisons are less robust for metadata combinations with few samples and few studies
- Solvent extraction efficiency differences across tissues/biofluids may bias fold-changes, and cross-organ comparisons mix biofluids versus tissues (different units of biomass)
- Abundances can be underestimated when detector saturation occurs during coelution of very high and very low abundance compounds
- Aggregation inherits biases from the static December 2021 BinBase snapshot; the max-p/min-fold-change rule can mask true effects that are consistent but weak in a subset of triads

## Evidence

- [intro] OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs.: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [methods] We heavily employed statistics routines from SciPy and the network analysis framework from NetworkX: "We heavily employed statistics routines from SciPy and the network analysis framework from NetworkX"
- [methods] we normalized metabolite intensities in each sample by the sum of the FAME ion intensities: "we normalized metabolite intensities in each sample by the sum of the FAME ion intensities"
- [discussion] for biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses: "for biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses"
