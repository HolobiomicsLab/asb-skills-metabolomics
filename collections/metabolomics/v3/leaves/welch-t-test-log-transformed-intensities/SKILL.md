---
name: welch-t-test-log-transformed-intensities
description: Use when comparing log-transformed GC-TOF MS metabolite intensity distributions
  between two groups of biological metadata triads (organ/species/disease) and you
  need per-compound p-values with unequal-variance robustness.
when_to_use_negative:
- Input is raw or unnormalized chromatographic intensities — BinBase intensities must
  first be normalized by the sum of FAME ion intensities, deconvoluted, and missing
  values imputed before testing
- The metadata combinations have very few samples or few studies — the paper states
  quantitative comparisons are less robust than differential analyses in that regime,
  and triads with <10 samples or >20% failed FAME samples were already removed
- The question is a single pairwise metadata contrast rather than an ontologically
  expanded broad comparison; OGDA's group-expansion and aggregation machinery is unnecessary
  there
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: SciPy
  role: Provides the statistics routines used to compute the Welch t-tests on log-transformed
    metabolite intensity distributions
- name: NetworkX
  role: Builds the ontology 'belongs to' graph used to expand broad query terms into
    the metadata combinations that form the t-test groups
- name: BinDiscover
  role: Hosts the OGDA query mechanism that this t-test step implements; outputs stored
    in a PostgreSQL database
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:7defe16c84ea185cf0d9251540eff7165610f6d7a1adb84843d1f20a20d1ee5e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinDiscover
  - NetworkX
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# welch-t-test-log-transformed-intensities

## Summary

Computes Welch t-tests on log-transformed metabolite intensity distributions for each pair of metadata combinations across two query groups, as done in BinDiscover's ontologically grouped differential analysis (OGDA) to generate per-compound p-value and fold-change matrices.

## When to use

Use when comparing log-transformed GC-TOF MS metabolite intensity distributions between two groups of biological metadata triads (organ/species/disease) and you need per-compound p-values with unequal-variance robustness.

## When NOT to use

- Input is raw or unnormalized chromatographic intensities — BinBase intensities must first be normalized by the sum of FAME ion intensities, deconvoluted, and missing values imputed before testing
- The metadata combinations have very few samples or few studies — the paper states quantitative comparisons are less robust than differential analyses in that regime, and triads with <10 samples or >20% failed FAME samples were already removed
- The question is a single pairwise metadata contrast rather than an ontologically expanded broad comparison; OGDA's group-expansion and aggregation machinery is unnecessary there

## Inputs

- Per-sample metabolite intensity matrix from BinBase bins (773 identified metabolites, 39 known chemical artifacts, 15,843 unannotated spectra in the paper's snapshot)
- Metadata triads (organ/species/disease) mapped to ontologies (MeSH for organs/diseases, NCBI taxonomy for species), filtered to triads with ≤20% failed FAME samples and ≥10 samples
- Ontology 'belongs to' graph built with NetworkX used to expand broad query terms into constituent metadata combinations

## Outputs

- Per-compound n*m Welch t-test p-value matrix (one row per compound, one column per metadata-combination pair)
- Per-compound n*m fold-change matrix
- Aggregated per-compound result (maximum p-value, minimum fold-change, fold-change set to 0 on sign disagreement), e.g. ogda_results.csv

## How to apply

Log-transform the FAME-normalized metabolite intensities before testing, since intensity distributions span orders of magnitude and imputation was performed as percentage-of-presence multiplied by the minimum detected value. For each of the n*m pairs of metadata combinations across the two query groups (expanded from ontology branches via the 'belongs to' graph), compute a Welch t-test (unequal variances assumed, equal_var=False) on the log-transformed intensity distributions of each compound using SciPy statistics routines. Collect the resulting n*m p-value and fold-change matrices per compound. Aggregate conservatively per the paper's rule: retain the maximum p-value across pairs, the minimum fold-change, and set fold-change to 0 when significant pairs show both positive and negative fold-changes. Note that this differential analysis is preferred over quantitative comparison for metadata combinations with many samples/studies; quantitative comparisons are less robust when few samples are available.

## Related tools

- **SciPy** (Provides the statistics routines used to compute the Welch t-tests on log-transformed metabolite intensity distributions)
- **NetworkX** (Builds the ontology 'belongs to' graph used to expand broad query terms into the metadata combinations that form the t-test groups)
- **BinDiscover** (Hosts the OGDA query mechanism that this t-test step implements; outputs stored in a PostgreSQL database)

## Examples

```
from scipy import stats; t_stat, p_val = stats.ttest_ind(np.log(group1_intensities), np.log(group2_intensities), equal_var=False)
```

## Evaluation signals

- ogda_results.csv contains exactly one aggregated p-value and one aggregated fold-change per compound
- Aggregated p-value equals the maximum across the n*m pairwise p-values for each compound (conservative max-p rule)
- Aggregated fold-change equals the minimum across pairwise fold-changes, and is 0 whenever significant pairs show both positive and negative fold-changes
- A synthetic metadata-annotated intensity fixture run in-task reproduces the expected aggregation output
- Intensity values are log-transformed before testing (e.g. raw intensities span orders of magnitude; log values should be comparable across compounds)

## Limitations

- For biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses
- Different solvent extraction efficiencies across tissues or biofluids may introduce quantitative bias into intensity comparisons
- Abundances can be underestimated when detector saturation occurs during coelution of very high-abundance and very low-abundance compounds
- Cross-organ comparisons may mix biofluids versus tissues, i.e. different units of biomass

## Evidence

- [intro] OGDA transform broad questions into smaller categories: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [methods] SciPy statistics routines: "We heavily employed statistics routines from SciPy and the network analysis framework from NetworkX"
- [methods] FAME normalization of intensities: "we normalized metabolite intensities in each sample by the sum of the FAME ion intensities"
- [discussion] Robustness caveat for small combinations: "for biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses"
