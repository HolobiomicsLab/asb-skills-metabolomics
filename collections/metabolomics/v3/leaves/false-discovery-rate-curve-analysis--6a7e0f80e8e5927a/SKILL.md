---
name: false-discovery-rate-curve-analysis
description: Use when matched similarity scores from an MS/MS library search (e.g.,
  434,287 spectra self-searched against NIST20) must be converted into score-threshold
  curves to compare algorithms by FDR/TPR.
when_to_use_negative:
- Matched similarity scores have not yet been computed for the library search — FDR
  curves require per-query, per-algorithm score lists as input
- The goal is raw similarity computation for a single spectrum pair rather than benchmarking
  identification reliability across many queries
- Query and library spectra lack compound-identity information, so true vs. false
  identifications cannot be defined
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: ms_entropy
  role: Provides calculate_entropy_similarity and clean_spectrum used to generate
    the matched entropy-similarity scores that are thresholded into FDR/TPR curves
  repo: https://github.com/YuanyueLi/MSEntropy
- name: Python 3.7.9
  role: Implementation language for the 43 similarity algorithms (Supplementary Note
    1) and the FDR/TPR benchmark code
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: R 3.6.3
  role: Used alongside Python for the benchmarking code producing score-threshold
    curves
- name: SciPy
  role: Optional source of the entropy function for computing spectral entropy without
    the clean_spectrum step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:3046f799463853ba3c1eb773630338264c6fdc57935e70a2c0f30782cbdfae5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - MSEntropy
  - NumPy
  - Python 3.7.9
  - R 3.6.3
  - SciPy
  - ms_entropy
  - multiprocessing
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# false-discovery-rate-curve-analysis

## Summary

Builds false discovery rate (FDR) and true-positive rate (TPR) curves as functions of a similarity-score threshold, so that library-matching scores (e.g., entropy similarity vs. dot product) can be compared on identification reliability rather than raw score scale.

## When to use

Use when matched similarity scores from an MS/MS library search (e.g., 434,287 spectra self-searched against NIST20) must be converted into score-threshold curves to compare algorithms by FDR/TPR.

## When NOT to use

- Matched similarity scores have not yet been computed for the library search — FDR curves require per-query, per-algorithm score lists as input
- The goal is raw similarity computation for a single spectrum pair rather than benchmarking identification reliability across many queries
- Query and library spectra lack compound-identity information, so true vs. false identifications cannot be defined

## Inputs

- Per-query matched similarity scores for each of the 43 algorithms (library self-search of 434,287 NIST20 high-resolution MS/MS spectra)
- True/false match labels derived from compound identity of query vs. library spectra
- Sweep of similarity-score thresholds (e.g., including the reported 0.75 entropy-score operating point)

## Outputs

- Per-algorithm FDR/TPR curves as functions of the similarity-score threshold (similarity_benchmark_fdr_tpr.csv)
- Operating points such as FDR < 10% at entropy similarity score 0.75 (observed on 37,299 experimental natural product spectra)

## How to apply

For each query spectrum, compute similarity scores against the library under all candidate algorithms (43 in the paper, implemented in Python 3.7.9 per Supplementary Note 1, with fragment ions matched at 0.05 Da tolerance and entropy similarity via ms_entropy.calculate_entropy_similarity). Classify each match above a given threshold as a true or false identification using compound identity information from the self-search, then aggregate per-algorithm matched scores across thresholds. Compute false discovery rate and true-positive rate at each score threshold and write the per-algorithm curves (e.g., to similarity_benchmark_fdr_tpr.csv). Judge algorithms by the FDR they yield at comparable score thresholds — entropy similarity outperformed dot product similarity on FDR, but note the paper's caveat that no single similarity threshold can prevent false-positive discoveries entirely.

## Related tools

- **ms_entropy** (Provides calculate_entropy_similarity and clean_spectrum used to generate the matched entropy-similarity scores that are thresholded into FDR/TPR curves) — https://github.com/YuanyueLi/MSEntropy
- **Python 3.7.9** (Implementation language for the 43 similarity algorithms (Supplementary Note 1) and the FDR/TPR benchmark code) — https://github.com/YuanyueLi/SpectralEntropy
- **R 3.6.3** (Used alongside Python for the benchmarking code producing score-threshold curves)
- **SciPy** (Optional source of the entropy function for computing spectral entropy without the clean_spectrum step)

## Examples

```
import ms_entropy as me; score = me.calculate_entropy_similarity(peaks_query, peaks_library)
```

## Evaluation signals

- FDR/TPR curve files must contain one curve per similarity algorithm tested (43 total), keyed by score threshold
- At the reported operating point, entropy similarity achieves false discovery rates below 10% at score 0.75 on the 37,299 natural product spectra
- The entropy-similarity FDR curve should lie below (outperform) the dot-product-similarity FDR curve across the threshold sweep
- FDR should decrease monotonically as the similarity-score threshold increases for a well-behaved score
- True/false classification must exclude the test spectrum itself from the library in the self-search setup

## Limitations

- Even with entropy similarity scores, no single similarity threshold can be defined that would prevent false-positive discoveries
- The benchmark assumes true/false labels are available (NIST20 self-search with the test spectrum left out of the library); spectra without identity annotations cannot yield FDR estimates
- Curve conclusions are specific to the benchmarked libraries (NIST20, Massbank.us, GNPS subsets); the ENTACT study data were not publicly available for benchmarking
- Power transformations at w < 1 decrease similarity scoring performance for ion-rich, high-entropy spectra, which would distort resulting FDR curves

## Evidence

- [abstract] Entropy similarity outperformed 42 alternative similarity algorithms, including dot product similarity, when searching 434,287 spectra against the high-quality NIST20 library: "Entropy similarity outperformed 42 alternative similarity algorithms, including dot product similarity, when searching 434,287 spectra against the high-quality NIST20 library"
- [discussion] entropy similarity outperformed FDR data compared to classic dot product similarity methods: "entropy similarity outperformed FDR data compared to classic dot product similarity methods"
- [discussion] no single similarity threshold can be defined that would prevent false-positive discoveries: "even with entropy similarity scores, no single similarity threshold can be defined that would prevent false-positive discoveries."
- [abstract] false discovery rates of less than 10% were observed at entropy similarity score 0.75: "false discovery rates of less than 10% were observed at entropy similarity score 0.75"
- [discussion] low-abundance ions at <1% base peak intensity were removed, and ions were matched at 0.05 Da tolerance: "low-abundance ions at <1% base peak intensity were removed, and ions were matched at 0.05 Da tolerance."
