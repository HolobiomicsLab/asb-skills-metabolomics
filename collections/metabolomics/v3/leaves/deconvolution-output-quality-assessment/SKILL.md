---
name: deconvolution-output-quality-assessment
description: Use when GC-MS spectral-match annotations carry cosine similarity and
  balance score columns and you need to select thresholds that minimize top-match
  FDR, e.g. before building molecular networks or validating deconvolved compound
  identifications.
when_to_use_negative:
- Input spectra are raw GC-MS files that still require MSHub auto-deconvolution —
  run the unsupervised non-negative matrix factorization pipeline first; this skill
  only filters existing match annotations.
- The dataset lacks balance score annotations, e.g. matches produced by tools other
  than MSHub/GNPS that do not compute fragmentation-pattern reproducibility across
  samples.
- The goal is LC-MS/MS molecular networking annotation (GNPS library search on MS2
  spectra) rather than GC-MS deconvolved spectral-match quality control.
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MSHub
  role: Produces the auto-deconvolved GC-MS spectral matches (cosine similarity +
    balance score annotations) that are filtered and FDR-assessed by this skill.
  repo: https://github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform whose workflows store, process, and annotate the GC-MS data within
    which the spectral-match filtering and FDR benchmark is executed.
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:52962563337c0f4312f50e73bd4a7c6fcd737d2e9353b6aeab1c31050666a3a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GNPS_GC
  tools:
  - GNPS
  - MSHub
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# deconvolution-output-quality-assessment

## Summary

Assesses the quality of MSHub/GNPS GC-MS auto-deconvolution spectral matches by jointly filtering on cosine similarity (>0.9) and balance score (>60%), or on balance score alone (>80%), and comparing the false discovery rate (FDR) of the top match. This reproduces the Figure 1m,n benchmark thresholds from Aksenov et al. (2020).

## When to use

Use when GC-MS spectral-match annotations carry cosine similarity and balance score columns and you need to select thresholds that minimize top-match FDR, e.g. before building molecular networks or validating deconvolved compound identifications.

## When NOT to use

- Input spectra are raw GC-MS files that still require MSHub auto-deconvolution — run the unsupervised non-negative matrix factorization pipeline first; this skill only filters existing match annotations.
- The dataset lacks balance score annotations, e.g. matches produced by tools other than MSHub/GNPS that do not compute fragmentation-pattern reproducibility across samples.
- The goal is LC-MS/MS molecular networking annotation (GNPS library search on MS2 spectra) rather than GC-MS deconvolved spectral-match quality control.

## Inputs

- MSHub/GNPS deconvolution output table of GC-MS spectral matches with cosine similarity and balance score columns
- Figure 1m-o test dataset of spectral-match annotations
- Reference/validation labels for top-match correctness used to compute FDR

## Outputs

- FDR of the top match per filter regime (cosine >0.9 & balance >60%; balance >80%)
- fdr_filter_comparison.csv with per-filter FDR results
- Selected threshold regime passing the FDR ordering validation

## How to apply

Load the MSHub deconvolution output (spectral matches annotated with cosine similarity and balance score) for the test dataset, such as the Figure 1m-o breathomics dataset. Apply the joint filter of cosine >0.9 combined with balance score >60% and compute the FDR of the top match; then apply the balance-score-only filter of >80% and compute FDR again. Compare FDR values across the two regimes: the article found the lowest FDR of the top match at the combined cosine >0.9 / balance >60% threshold, while the conservative balance >80% filter essentially guarantees the lowest observed FDR even when cosine (match) scores are poor. Write per-filter FDR results to a comparison file (fdr_filter_comparison.csv) and use the threshold that matches the reported FDR ordering. The balance score itself quantifies reproducibility of fragmentation patterns across samples, which is why it can rescue annotations with weak cosine similarity.

## Related tools

- **MSHub** (Produces the auto-deconvolved GC-MS spectral matches (cosine similarity + balance score annotations) that are filtered and FDR-assessed by this skill.) — https://github.com/bittremieux/GNPS_GC
- **GNPS** (Platform whose workflows store, process, and annotate the GC-MS data within which the spectral-match filtering and FDR benchmark is executed.) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
import pandas as pd; d = pd.read_csv('mshub_matches.csv'); r = {'cos_gt09_bal_gt60': d[(d.cosine>0.9)&(d.balance>0.6)].top_match_fdr(), 'bal_gt80': d[d.balance>0.8].top_match_fdr()}; pd.Series(r).to_csv('fdr_filter_comparison.csv')
```

## Evaluation signals

- Reproduced FDR ordering matches the reported relationship: lowest FDR of the top match occurs at the combined cosine >0.9 / balance score >60% threshold.
- The lowest observed FDR is guaranteed under the balance score >80% filter, even for poor cosine scores.
- fdr_filter_comparison.csv exists and contains per-filter FDR values for both filtering regimes.
- All rows entering the filters carry non-null cosine similarity and balance score annotations from the MSHub deconvolution output.
- If the reproduced ordering diverges from the reported one, the task is flagged for expert_review rather than silently accepted.

## Limitations

- Threshold performance (cosine >0.9, balance >60% / >80%) is calibrated on the specific Figure 1m,n test dataset; FDR ordering on other datasets may differ and should trigger expert review.
- The balance-score-only filter of >80% is conservative and essentially guarantees the lowest observed FDR, but may discard valid annotations that pass the joint cosine/balance filter.
- FDR depends on the availability of trustworthy reference labels for the top match; without them the filter comparison cannot be validated.

## Evidence

- [other] combined threshold finding: [evidence span withheld — anchor status: not_found]
