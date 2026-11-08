---
name: conservative-pvalue-foldchange-aggregation
description: Decision support for ontologically grouped differential analysis (OGDA)
  queries in BinDiscover where a broad term (e.g. 'digestive tract') expands to many
  metadata combinations and each compound yields multiple pairwise Welch t-test results
  that must be condensed into one conservative call.
when_to_use_negative:
- When comparing a single specific metadata combination (e.g. one organ vs. one organism)
  rather than a broad ontology term that expands to many combinations — no aggregation
  across pairs is needed.
- When quantitative comparison across few samples/studies is intended; the article
  notes quantitative comparisons are less robust for metadata combinations with few
  samples and few studies, so conservative aggregation on such sparse pairs may be
  misleading.
- When analysing unannotated spectra or chemical artifacts — restrict to the 773 identified
  metabolites as the article does.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BinDiscover
  role: Executes the OGDA query, expands ontology terms into metadata combinations,
    runs pairwise Welch t-tests, and performs the conservative p-value/fold-change
    aggregation.
  repo: https://github.com/GFiehnLab/BinDiscover
- name: SciPy
  role: Provides the statistics routines underlying the pairwise Welch t-tests whose
    outputs are aggregated.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:a1f3d96c4967c21e9bedd7ad75367187cd9d5a8f5b65f5f00b299f677e35453e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GFiehnLab/BinDiscover
  tools:
  - BinBase
  - BinDiscover
  - SciPy
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# conservative-pvalue-foldchange-aggregation

## Summary

Aggregates per-pair Welch t-test p-values and fold-change matrices across all ontology-expanded metadata-combination pairs per compound using the maximum p-value and minimum fold-change as conservative boundaries, so broad OGDA queries report a single significant/non-significant call per metabolite.

## When to use

Decision support for ontologically grouped differential analysis (OGDA) queries in BinDiscover where a broad term (e.g. 'digestive tract') expands to many metadata combinations and each compound yields multiple pairwise Welch t-test results that must be condensed into one conservative call.

## When NOT to use

- When comparing a single specific metadata combination (e.g. one organ vs. one organism) rather than a broad ontology term that expands to many combinations — no aggregation across pairs is needed.
- When quantitative comparison across few samples/studies is intended; the article notes quantitative comparisons are less robust for metadata combinations with few samples and few studies, so conservative aggregation on such sparse pairs may be misleading.
- When analysing unannotated spectra or chemical artifacts — restrict to the 773 identified metabolites as the article does.

## Inputs

- Per-compound matrix of pairwise Welch t-test p-values from all n×m metadata-combination pairs (log-transformed BinBase intensity distributions)
- Per-compound matrix of fold changes from the same n×m metadata-combination pairs

## Outputs

- Per-compound aggregated maximum p-value and minimum fold-change boundaries
- Significant-metabolite table restricted to 773 structurally identified compounds (e.g. Additional file 3: Table S2)

## How to apply

After running pairwise Welch t-tests on log-transformed intensity distributions for all n×m metadata-combination pairs per compound, aggregate the resulting p-value and fold-change matrices conservatively: take the maximum p-value across all pairs and the minimum fold-change across all pairs as the boundaries for that compound. Represent the fold change as 0 when the significant pairs show both positive and negative fold changes, signalling an ambiguous direction. Restrict the output to the 773 structurally identified compounds and export the significant-metabolite table. Rationale: a compound should only be called significant if it is consistently differentiated across every metadata combination matched by the broad ontology term, avoiding inflated calls from a single strong pair among many non-significant ones.

## Related tools

- **BinDiscover** (Executes the OGDA query, expands ontology terms into metadata combinations, runs pairwise Welch t-tests, and performs the conservative p-value/fold-change aggregation.) — https://github.com/GFiehnLab/BinDiscover
- **SciPy** (Provides the statistics routines underlying the pairwise Welch t-tests whose outputs are aggregated.)

## Examples

```
python bdiscover_ogda.py --query 'Human Digestive Tract vs Bacteria' --identified-only --output table_s2.csv
```

## Evaluation signals

- The exported table must contain exactly the 15 significant metabolites reported at increased levels in human digestive system organs, matching Additional file 3: Table S2.
- For every compound, the reported p-value must equal the maximum p-value across all pairwise comparisons, and the reported fold-change must equal the minimum fold-change across all pairs.
- Compounds whose significant pairs span both positive and negative fold changes must be encoded with fold change = 0.
- Runtime for the 773-compound OGDA query should be ~26 s (~1 s per metadata combination query).
- Output restricted to structurally identified compounds (773), excluding the 39 known chemical artifacts and 15,843 unannotated spectra.

## Limitations

- Quantitative comparisons are less robust for metadata combinations with few samples and few studies.
- BinDiscover relies on a static December 2021 snapshot of BinBase, so novel compounds added later cause metadata incongruencies.
- Different solvent extraction efficiencies across tissues or biofluids may introduce quantitative bias affecting fold-change interpretation.
- Abundances can be underestimated when detector saturation occurs during coelution of high- and low-abundance compounds.
- Cross-organ comparisons mix biofluids versus tissues (different units of biomass).

## Evidence

- [methods] Aggregate the per-compound p-value and fold-change matrices conservatively using the maximum p-value and minimum fold-change as boundaries: [evidence span withheld — anchor status: not_found]
- [intro] OGDA queries transform broad questions into sets of smaller categories and combine statistical result outputs into graphs: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [methods] we calculated the averages, medians, and ratios of intensity values and stored the resulting dataset in a PostgreSQL BinDiscover database: "we calculated the averages, medians, and ratios of intensity values and stored the resulting dataset in a PostgreSQL BinDiscover database"
- [methods] Bins included 773 identified metabolites: "Bins included 773 identified metabolites"
