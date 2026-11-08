---
name: metabolomics-cross-study-meta-analysis
description: Use when many GC/MS (or similar) metabolomics studies must be compared
  across broad biological categories — e.g., 'human digestive tract' vs 'bacteria'
  — and study metadata are heterogeneous, unstructured, or ontology-definable.
when_to_use_negative:
- The query concerns a single study or bulk compound-centric lookup rather than cross-study
  differential comparison (compound-centric meta-analysis across databases like HMDB/MetabolomicsWorkbench
  is what this approach avoids)
- Metadata combinations have few samples and few studies, where quantitative comparisons
  are less robust than for differential analyses
- Data are LC/MS or unprocessed instrument files — this skill presumes GC/TOF MS data
  already deconvoluted, de-noised, and binned (e.g., by BinBase)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: BinDiscover
  role: Webtool executing the ontology-based OGDA query, pairwise Welch t-tests, conservative
    p-value/fold-change aggregation, and export of significant-metabolite tables
- name: BinBase
  role: Source database providing the standardized multi-study GC/MS metabolomics
    corpus (156,174 samples, 18,290 Bins, 773 identified metabolites) downloaded as
    the December 2021 snapshot
- name: SciPy
  role: Statistics routines for the pairwise Welch t-tests on log-transformed intensity
    distributions
- name: PostgreSQL
  role: Stores the curated dataset of averages, medians, and ratios of intensity values
    backing BinDiscover queries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:cf7da424f44f1ce1e05047b137bf0e2ecd2743bc5e545e5ec30cf2ce93473ea7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinBase
  - BinDiscover
  - PostgreSQL
  - SciPy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metabolomics-cross-study-meta-analysis

## Summary

Perform multi-study differential analysis of GC/MS metabolomics data (e.g., the BinBase corpus of 156,174 samples / 18,290 Bins from ~2000 studies) by expanding broad biological questions into ontology-grouped categories and running pairwise statistics that are conservatively aggregated. BinDiscover operationalizes this as ontologically grouped differential analysis (OGDA).

## When to use

Use when many GC/MS (or similar) metabolomics studies must be compared across broad biological categories — e.g., 'human digestive tract' vs 'bacteria' — and study metadata are heterogeneous, unstructured, or ontology-definable.

## When NOT to use

- The query concerns a single study or bulk compound-centric lookup rather than cross-study differential comparison (compound-centric meta-analysis across databases like HMDB/MetabolomicsWorkbench is what this approach avoids)
- Metadata combinations have few samples and few studies, where quantitative comparisons are less robust than for differential analyses
- Data are LC/MS or unprocessed instrument files — this skill presumes GC/TOF MS data already deconvoluted, de-noised, and binned (e.g., by BinBase)

## Inputs

- Multi-study GC/MS metabolomics intensity matrix exported from a processing pipeline (e.g., BinBase bins across 156,174 samples)
- FAME internal-standard ion intensities for normalization
- Sample metadata (biological triads such as species/organ/disease) to be mapped into ontologies
- Ontology triads defining the contrast groups for an OGDA query

## Outputs

- Per-compound aggregated p-value and fold-change matrices (Welch t-test on log-transformed intensities)
- Significant-metabolite table restricted to structurally identified compounds (e.g., 15 metabolites elevated in human digestive tract organs vs bacteria)
- Stored averages, medians, and ratios of intensity values in a PostgreSQL BinDiscover database

## How to apply

First curate the merged dataset: remove samples with poor FAME patterns (extremely high or low FAME intensity), drop entire biological metadata triads with >20% failed FAME samples or fewer than 10 samples, normalize metabolite intensities per sample by the sum of FAME ion intensities, combine duplicate bins into a single metabolite intensity, and impute missing values as percentage-of-presence times the minimum detected bin value per metadata combination. Map all metadata to established ontologies (correcting misspellings) so generic terms like 'digestive tract' or 'rodents' expand into their 'belongs to' node sets of metadata combinations. For an OGDA query, define ontology triads (e.g., Human/Digestive Tract/no Disease vs Bacteria/Cells/no Disease), run pairwise Welch t-tests on log-transformed intensity distributions over all n*m metadata-combination pairs per compound, then aggregate per-compound p-values and fold changes conservatively (maximum p-value, minimum fold-change; represent fold change as 0 when significant pairs show both positive and negative fold changes). Restrict output to structurally identified compounds (773 in the BinBase December 2021 snapshot) and export the significant-metabolite table for hypothesis generation.

## Related tools

- **BinDiscover** (Webtool executing the ontology-based OGDA query, pairwise Welch t-tests, conservative p-value/fold-change aggregation, and export of significant-metabolite tables)
- **BinBase** (Source database providing the standardized multi-study GC/MS metabolomics corpus (156,174 samples, 18,290 Bins, 773 identified metabolites) downloaded as the December 2021 snapshot)
- **SciPy** (Statistics routines for the pairwise Welch t-tests on log-transformed intensity distributions)
- **PostgreSQL** (Stores the curated dataset of averages, medians, and ratios of intensity values backing BinDiscover queries)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(log_intensities_group_a, log_intensities_group_b, equal_var=False)  # pairwise Welch t-test per metadata-combination pair
```

## Evaluation signals

- The exported significant-metabolite table must match the article's gold standard: exactly 15 significant metabolites reported at increased levels in human digestive system organs (Additional file 3: Table S2)
- Metadata triads remaining after filtering must have no more than 20% failed FAME samples and at least 10 samples each
- Per-compound results must reflect conservative aggregation: maximum p-value and minimum fold-change across metadata-combination pairs, with fold change set to 0 when significant pairs show both positive and negative fold changes
- Human stool must be excluded from the 'Digestive Tract' group because it maps to the MeSH ontology term 'fluids and secretions'
- Runtime scale check: the full query over 773 structurally identified compounds should complete in roughly 26 s (~1 s per metadata combination query)

## Limitations

- BinDiscover relies on a static December 2021 snapshot of BinBase, so novel compounds added later cause metadata incongruencies
- For biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses
- Different solvent extraction efficiencies across tissues or biofluids may introduce quantitative bias; cross-organ comparisons may mix biofluids versus tissues (different units of biomass)
- Abundances can be underestimated when detector saturation occurs during coelution of very high and very low abundance compounds

## Evidence

- [intro] OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs.: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [methods] FAME normalization: "we normalized metabolite intensities in each sample by the sum of the FAME ion intensities"
- [methods] Triad filtering thresholds: "we removed entire biological metadata triads if they showed more than 20% failed FAME samples"
- [methods] Imputation rule: "we imputed the percentage of presence, multiplied by the minimum value of detected peaks (bins) for each metadata combination"
- [abstract] Ontology aggregation rationale: "we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies."
- [discussion] Robustness caveat: "for biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses"
