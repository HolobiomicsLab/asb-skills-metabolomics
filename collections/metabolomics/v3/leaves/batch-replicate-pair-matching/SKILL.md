---
name: batch-replicate-pair-matching
description: Use when you have replicate pairs of identical biological samples run
  in subsequent batches, log(e)-transformed intensities, and need to rank normalization
  methods by cross-batch correspondence rather than by QC-sample RSD reduction or
  differential-abundance counts.
when_to_use_negative:
- When the data set contains no between-batch replicate pairs (identical biological
  samples split across subsequent batches) — this metric requires true replicate-pair
  structure and cannot be computed from QC samples alone.
- When the goal is to select a normalization method by maximizing the number of differentially
  abundant metabolites or by minimizing QC-sample RSD — the authors explicitly flag
  these as misleading benchmarking proxies.
- When only a single batch exists and no cross-batch correspondence assessment is
  needed.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: Metanorm
  role: R package that generates the normalized intensities (tGAM, rGAM, rLOESS, QC-RLSC,
    QC-RSC) whose between-batch replicate concordance is then scored via the MAD metric;
    also supplies the rawdata/batch/metanorm.qc input structures
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Environment in which the MAD computation over rescaled replicate-pair differences
    is performed and results are tabulated
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:a0fca7f95ec94a6d35ac28623b676f086c1e4fd03ac397cdf253006f5ebcf992
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# batch-replicate-pair-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies cross-batch intensity correspondence between between-batch replicate run pairs by computing the MAD of mean-rescaled intensity differences for each normalization method (tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, QC-RSC, unnormalized), a replication-based benchmarking metric the authors explicitly recommend over QC-RSD proxies.

## When to use

Use when you have replicate pairs of identical biological samples run in subsequent batches, log(e)-transformed intensities, and need to rank normalization methods by cross-batch correspondence rather than by QC-sample RSD reduction or differential-abundance counts.

## When NOT to use

- When the data set contains no between-batch replicate pairs (identical biological samples split across subsequent batches) — this metric requires true replicate-pair structure and cannot be computed from QC samples alone.
- When the goal is to select a normalization method by maximizing the number of differentially abundant metabolites or by minimizing QC-sample RSD — the authors explicitly flag these as misleading benchmarking proxies.
- When only a single batch exists and no cross-batch correspondence assessment is needed.

## Inputs

- Unnormalized metabolite intensity matrix (rows = compounds, columns = sample runs), log-transformed (base e) prior to normalization
- Vector assigning each sample run to its batch
- Vector flagging which sample runs are QC vs. biological samples (metanorm.qc style)
- Identity mapping between between-batch replicate runs: pairs of identical biological samples processed in subsequent batches
- Normalized intensity matrices produced by each compared method (tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, QC-RSC) via Metanorm

## Outputs

- Per-method MAD of mean-rescaled between-replicate differences
- Per-method percentage MAD reduction versus the unnormalized baseline
- Tabulated results file (e.g. replicate_concordance_mad.tsv)

## How to apply

For each replicate pair and compound, take the two intensities (log base e, already processed through Metanorm under each method being compared), subtract them, and rescale the difference by the mean of the two intensities. In R, compute the MAD of these rescaled differences pooled across all pairs and compounds, once per normalization approach — including an unnormalized baseline run. Convert each method's MAD into a percentage reduction relative to the unnormalized baseline to rank methods. Use the Metanorm R package (metanorm function, with model='tGAM' etc., batch argument and type vector marking QCs) to generate the normalized intensities feeding the metric; interpret lower MAD as better between-batch correspondence. This replicate-concordance benchmark is preferred by the authors to proxies such as QC-sample relative standard deviation reduction or counts of differentially abundant metabolites, which they flag as common pitfalls in benchmarking normalization strategies.

## Related tools

- **Metanorm** (R package that generates the normalized intensities (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC) whose between-batch replicate concordance is then scored via the MAD metric; also supplies the rawdata/batch/metanorm.qc input structures) — https://github.com/UGent-LIMET/Metanorm
- **R** (Environment in which the MAD computation over rescaled replicate-pair differences is performed and results are tabulated) — https://cloud.r-project.org/index.html

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, batch = batch); mad(repdiff, constant = 1.4826)
```

## Evaluation signals

- Robust methods (tGAM, rGAM, rLOESS) should show a 49–52% reduction in MAD of rescaled between-replicate differences relative to unnormalized data.
- SERFF should yield approximately a 41% reduction and TIGER approximately 40%.
- QC-RLSC and QC-RSC should show only 18–22% reduction on the BioHEART data.
- Verify the MAD is computed on differences rescaled by the pair's mean intensity, not on raw differences.
- Sanity-check that lower MAD indeed corresponds to the methods whose pre/post-normalization intensity-vs-order plots and PCA score plots (from Metanorm's plotdir and plotPCA outputs) show diminished batch structure.

## Limitations

- The authors recommend careful neutrality in benchmarking: 'care should be taken to ensure neutrality, i.e., not potentially favoring any of the benchmarked methods' — the metric alone should not drive method selection.
- Unexpected differences between QC samples and biological samples 'should receive due analyst scrutiny'; automated acceptance of normalization performance is not provided.
- Multivariate outlier handling (as in SERFF/TIGER) was not benchmarked with this metric in the article's main evaluation, and benchmarking a broader set of methods 'remains as important future work'.
- Multiple testing correction is essential for downstream statistical inference but was out of scope for this normalization benchmarking work.

## Evidence

- [results] Normalized between-batch replicate intensities showed the lowest between-replicate normalized differences: [evidence span withheld — anchor status: not_found]
- [intro] Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples: "Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant"
- [discussion] care should be taken to ensure neutrality, i.e., not potentially favoring any of the benchmarked methods: "care should be taken to ensure neutrality, i.e., not potentially favoring any of the benchmarked methods"
- [discussion] unexpected differences between QC samples and biological samples should receive due analyst scrutiny: "unexpected differences between QC samples and biological samples should receive due analyst scrutiny"
- [readme] The example dataset contains three objects: rawdata, a numerical matrix containing the unnormalized data: "The example dataset contains three objects:
- *rawdata*, a numerical matrix containing the unnormalized data (rows = compounds, columns = samples)
- *batch*, a vector containing for each sample run"
