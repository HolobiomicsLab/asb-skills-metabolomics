---
name: welch-test-log-transformed-intensity-comparison
description: Use when comparing metabolite intensities across two broad biological
  ontology categories (e.g. 'digestive tract' vs 'Bacteria') over many metadata-combination
  pairs, where unequal variances and log-normal intensity distributions require Welch's
  t-test on log-transformed values.
when_to_use_negative:
- Quantitative (not differential) comparisons for biological metadata combinations
  with few samples and few studies, where the article states such comparisons are
  less robust
- Cross-organ comparisons mixing biofluids versus tissues (different units of biomass),
  which the article flags as a bias in fold-change interpretation
- Bulk metabolite queries across heterogeneous external databases (HMDB, MetabolomicsWorkbench)
  lacking standardized metadata; this skill requires harmonized BinBase triad metadata
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BinDiscover
  role: Hosts the OGDA query interface, ontology expansion into 'belongs to' metadata
    combinations, and aggregation of pairwise test statistics into graphs
- name: SciPy
  role: Supplies the statistical routines (Welch t-tests) applied to log-transformed
    intensity distributions
- name: BinBase
  role: Source of the QC-filtered, FAME-normalized GC/TOF-MS sample intensity data
    (156,174 samples, 18,290 Bins) feeding the comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:0ba902afcbd0f7a7551f66e41b6c0d9203f5bd780a173aedb4d4a8e818d5d95d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinBase
  - BinDiscover
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# welch-test-log-transformed-intensity-comparison

## Summary

Performs pairwise Welch t-tests on log-transformed GC/MS metabolite intensity distributions across ontology-defined metadata combinations, then conservatively aggregates per-compound p-values and fold changes into a single differential-analysis result per metabolite. This is the statistical core of BinDiscover's ontologically grouped differential analysis (OGDA).

## When to use

Use when comparing metabolite intensities across two broad biological ontology categories (e.g. 'digestive tract' vs 'Bacteria') over many metadata-combination pairs, where unequal variances and log-normal intensity distributions require Welch's t-test on log-transformed values.

## When NOT to use

- Quantitative (not differential) comparisons for biological metadata combinations with few samples and few studies, where the article states such comparisons are less robust
- Cross-organ comparisons mixing biofluids versus tissues (different units of biomass), which the article flags as a bias in fold-change interpretation
- Bulk metabolite queries across heterogeneous external databases (HMDB, MetabolomicsWorkbench) lacking standardized metadata; this skill requires harmonized BinBase triad metadata

## Inputs

- FAME-normalized, imputed GC/MS metabolite intensity matrix (773 identified metabolites x samples) from the BinBase December 2021 snapshot
- Two ontology triads defining the OGDA contrast (e.g. organism/tissue/disease: Human, Digestive Tract, no Disease vs Bacteria, Cells, no Disease)
- Ontology-expanded 'belongs to' metadata combinations for each triad
- Metadata-combination sample counts and failed-FAME statistics for QC filtering

## Outputs

- Per-compound aggregated p-value and fold-change matrix (max p-value, min fold-change boundaries)
- Significant-metabolite table (e.g. the 15 metabolites increased in human digestive system organs)
- Direction-coded fold change (0 where significant pairs conflict in sign)

## How to apply

First restrict input to the 773 structurally identified metabolite Bins from the BinBase snapshot (excluding the 39 known chemical artifacts and 15,843 unannotated spectra), using FAME-normalized intensities from samples that passed QC (metadata triads with ≤20% failed FAME samples and ≥10 samples, missing values imputed as percentage of presence multiplied by the minimum detected value). For each compound, let the ontology expansion enumerate all n*m ('belongs to') metadata-combination pairs between the two query triads, and run SciPy's pairwise Welch t-tests on log-transformed intensity distributions for every pair. Aggregate the per-compound p-value and fold-change matrices conservatively: take the maximum p-value and the minimum fold-change across pairs as the boundaries, and represent the fold change as 0 when significant pairs show both positive and negative fold changes. Expected runtime is ~26 s for the 773-compound OGDA query (~1 s per metadata combination query); export the significant-metabolite table for downstream comparison against reference tables such as Additional file 3: Table S2.

## Related tools

- **BinDiscover** (Hosts the OGDA query interface, ontology expansion into 'belongs to' metadata combinations, and aggregation of pairwise test statistics into graphs)
- **SciPy** (Supplies the statistical routines (Welch t-tests) applied to log-transformed intensity distributions)
- **BinBase** (Source of the QC-filtered, FAME-normalized GC/TOF-MS sample intensity data (156,174 samples, 18,290 Bins) feeding the comparison)

## Examples

```
from scipy.stats import ttest_ind; import numpy as np; stat, p = ttest_ind(np.log(a_intensities), np.log(b_intensities), equal_var=False)
```

## Evaluation signals

- The exported significant-metabolite table must contain exactly the 15 significant metabolites reported at increased levels in human digestive system organs, matching Additional file 3: Table S2 of the article
- The tested compound set must be restricted to the 773 structurally identified metabolites (not the 39 chemical artifacts or 15,843 unannotated spectra)
- Aggregated p-values must equal the maximum and fold changes the minimum across all n*m metadata-combination pairs per compound
- Fold change must be 0 for any compound where significant pairs show both positive and negative directions
- Runtime should be approximately 26 s for the full 773-compound OGDA query (~1 s per metadata combination query)

## Limitations

- Abundances can be underestimated when detector saturation occurs during coelution of very high abundance and very low abundance compounds
- Different solvent extraction efficiencies across different tissues or biofluids may introduce bias in fold-change estimates
- Comparisons across organs may include biofluids versus tissues, i.e. different units of biomass, complicating interpretation
- Human stool maps to the MeSH ontology 'fluids and secretions' and is therefore excluded from the digestive-tract OGDA query, so results are not exhaustive over all digestive tract-adjacent samples

## Evidence

- [methods] OGDA performs pairwise Welch t-tests on log-transformed intensities and aggregates conservatively: [evidence span withheld — anchor status: not_found]
- [other] Aggregation rule for p-values and fold changes: [evidence span withheld — anchor status: not_found]
- [methods] QC filtering thresholds applied before testing: [evidence span withheld — anchor status: not_found]
- [abstract] Ontology-based aggregation motivation underlying the test design: "we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies."
- [discussion] Robustness caveat for small metadata combinations: "for biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses"
