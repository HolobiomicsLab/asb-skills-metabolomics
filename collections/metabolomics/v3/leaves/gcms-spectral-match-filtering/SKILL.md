---
name: gcms-spectral-match-filtering
description: Use when top-hit GC-MS spectral match annotations (with cosine similarity
  and balance score columns) from MSHub/GNPS auto-deconvolution need FDR-controlled
  filtering, e.g. reproducing Figure 1m,n-style threshold benchmarks.
when_to_use_negative:
- Input spectra have not yet been auto-deconvolved — MSHub/GNPS non-negative matrix
  factorization deconvolution must run first before match filtering is meaningful
- The goal is molecular networking construction or chemical class propagation (e.g.
  via pyMolNetEnhancer) rather than FDR-controlled top-match annotation filtering
- The dataset lacks balance score annotations, which quantify fragmentation-pattern
  reproducibility across samples and are required for the >60%/>80% thresholds
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MSHub
  role: Machine learning auto-deconvolution engine producing GC-MS spectral matches
    with cosine similarity and balance score annotations used as filter inputs
  repo: github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform storing, processing and molecular-networking the GC-MS data; hosts
    the ProteoSAFe workflows whose deconvolution output is filtered
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:2dbda710c3c0fbb0a73beba0f5302dcbd17c7fdc7b8aa147010e54dc99cb6234
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MSHub
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gcms-spectral-match-filtering

## Summary

Jointly filters GC-MS top-match spectral annotations from MSHub/GNPS auto-deconvolution using cosine similarity and balance score thresholds (cosine >0.9 & balance >60%; balance >80% alone) to minimize the false discovery rate of top match annotations on a test dataset.

## When to use

Use when top-hit GC-MS spectral match annotations (with cosine similarity and balance score columns) from MSHub/GNPS auto-deconvolution need FDR-controlled filtering, e.g. reproducing Figure 1m,n-style threshold benchmarks.

## When NOT to use

- Input spectra have not yet been auto-deconvolved — MSHub/GNPS non-negative matrix factorization deconvolution must run first before match filtering is meaningful
- The goal is molecular networking construction or chemical class propagation (e.g. via pyMolNetEnhancer) rather than FDR-controlled top-match annotation filtering
- The dataset lacks balance score annotations, which quantify fragmentation-pattern reproducibility across samples and are required for the >60%/>80% thresholds

## Inputs

- MSHub deconvolution output (GC-MS spectral matches annotated with cosine similarity and balance score) for the Figure 1m-o test dataset

## Outputs

- Per-filter FDR comparison table (fdr_filter_comparison.csv) with top-match FDR under cosine>0.9 & balance>60% versus balance>80% filters
- Filtered set of top-match spectral annotations at each threshold regime

## How to apply

Load the MSHub deconvolution output for the test dataset, which contains per-spectral-match cosine similarity and balance score annotations; the balance score quantifies the reproducibility of fragmentation patterns across all samples. Apply the joint filter of cosine >0.9 with balance score >60% and compute the false discovery rate of the top match. Independently apply the balance-score-only filter of >80% and compute the top-match FDR. Compare FDR values across the two filtering regimes: the lowest FDR of the top match is expected at the combined cosine >0.9 / balance >60% threshold, while the more conservative balance score >80% essentially ensures the lowest observed FDR even for poor cosine match scores. Write the per-filter FDR results to fdr_filter_comparison.csv for validation.

## Related tools

- **MSHub** (Machine learning auto-deconvolution engine producing GC-MS spectral matches with cosine similarity and balance score annotations used as filter inputs) — github.com/bittremieux/GNPS_GC
- **GNPS** (Platform storing, processing and molecular-networking the GC-MS data; hosts the ProteoSAFe workflows whose deconvolution output is filtered) — http://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
df_top = df[(df.cosine > 0.9) & (df.balance_score > 0.60)]; df_bal = df[df.balance_score > 0.80]  # then compute top-match FDR per regime and write fdr_filter_comparison.csv
```

## Evaluation signals

- Reproduced FDR ordering must match the reported relationship: lowest FDR of the top match occurs with the combined cosine >0.9 / balance >60% threshold
- The balance score >80% filter must yield the lowest observed FDR of the top match, including for matches with poor cosine scores
- Both threshold regimes are computed on the same test dataset (Figure 1m,n) before comparison — never on different subsets
- fdr_filter_comparison.csv is produced and contains one FDR value per filtering regime
- Any deviation from the expected FDR ordering is flagged for expert_review rather than silently accepted

## Limitations

- FDR relationship was validated on the specific Figure 1m,n test dataset; ordering may differ on other GC-MS datasets and should be flagged for expert review if not reproduced
- The conservative balance score >80% filter can discard true matches with poor cosine scores — it guarantees the lowest observed FDR but at reduced match coverage
- The companion repository README provides no parameter-level details or scripts for the filtering benchmark itself

## Evidence
