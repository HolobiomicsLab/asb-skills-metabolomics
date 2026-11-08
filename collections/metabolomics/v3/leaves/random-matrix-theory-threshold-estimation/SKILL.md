---
name: random-matrix-theory-threshold-estimation
description: Use when you have a computed correlation matrix (e.g. occor.r from omics
  features) and must pick a defensible r_threshold for building a co-occurrence network
  instead of choosing a cutoff arbitrarily.
when_to_use_negative:
- The correlation threshold is already prescribed by the study design or prior protocol
  — RMT re-derivation is unnecessary and may conflict with the fixed cutoff.
- The similarity structure is not a linear correlation matrix — the article notes
  correlation-based network optimization may overlook complex, non-linear, or indirect
  associations.
- You need full multi-omics cross-layer network construction rather than threshold
  determination — use multi_net_build / c_net_set workflows instead.
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: MetaNet
  role: Provides rmt()/RMT_threshold() for the eigenvalue-spacing cutoff sweep, plot.rmt_res()
    for visualization, and c_net_build() which consumes the resulting r_threshold
  repo: github.com/Asa12138/MetaNet
- name: pcutils
  role: Required companion package supplying example correlation data (co_net_rmt
    context, otutab) used as the occor.r input
  repo: github.com/Asa12138/pcutils
- name: R
  role: Runtime environment (R >= 4.0) in which the MetaNet RMT thresholding workflow
    executes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:2fb6461bc84164ab58ab1638cdccc3bd370692ec1ee48f0f168ec69316f35454
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - pcutils
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# random-matrix-theory-threshold-estimation

## Summary

Determines an objective correlation cutoff (r_threshold) for co-occurrence network edges by sweeping candidate cutoffs in MetaNet and selecting the one whose thresholded correlation matrix best matches the random matrix theory (RMT) eigenvalue-spacing expectation, replacing subjective manual threshold choice.

## When to use

Use when you have a computed correlation matrix (e.g. occor.r from omics features) and must pick a defensible r_threshold for building a co-occurrence network instead of choosing a cutoff arbitrarily.

## When NOT to use

- The correlation threshold is already prescribed by the study design or prior protocol — RMT re-derivation is unnecessary and may conflict with the fixed cutoff.
- The similarity structure is not a linear correlation matrix — the article notes correlation-based network optimization may overlook complex, non-linear, or indirect associations.
- You need full multi-omics cross-layer network construction rather than threshold determination — use multi_net_build / c_net_set workflows instead.

## Inputs

- Correlation coefficient matrix (e.g. occor.r) produced by MetaNet c_net_calculate or the packaged co_net_rmt object
- Sweep parameters: min_threshold, max_threshold, and fixed step size defining the candidate cutoff range
- Output directory path (out_dir) for RMT_threshold results

## Outputs

- Objective r_threshold value falling within [min_threshold, max_threshold]
- rmt_threshold.txt file written to the specified out_dir
- RMT sweep result object plottable with plot.rmt_res(x, ...), optionally an animated threshold scan (gif flag)
- Correlation network built via c_net_build(corr, r_threshold, ...)

## How to apply

Install MetaNet from GitHub (devtools::install_github("Asa12138/MetaNet")) together with its required companion pcutils, and load the packaged correlation object co_net_rmt (or your own c_net_calculate output) as the occor.r input. Run rmt(occor.r, min_threshold, max_threshold, step) or RMT_threshold(occor.r, out_dir, min_threshold, max_threshold, step, plot, gif, verbose), sweeping candidate correlation cutoffs across the specified range at a fixed step. At each candidate cutoff, evaluate the eigenvalue-spacing statistic of the thresholded correlation matrix and identify the cutoff that minimizes deviation from the random-matrix-theory expectation for random matrices, thereby minimizing spurious edges. Visualize the sweep with plot.rmt_res(x, ...), optionally enabling the animated threshold scan via the gif flag, and write the identified r_threshold to rmt_threshold.txt in the RMT_threshold out_dir. Feed the resulting r_threshold directly into c_net_build(corr, r_threshold, ...) — e.g. c_net_build(cor, r_threshold = 0.65) — to construct the network.

## Related tools

- **MetaNet** (Provides rmt()/RMT_threshold() for the eigenvalue-spacing cutoff sweep, plot.rmt_res() for visualization, and c_net_build() which consumes the resulting r_threshold) — github.com/Asa12138/MetaNet
- **pcutils** (Required companion package supplying example correlation data (co_net_rmt context, otutab) used as the occor.r input) — github.com/Asa12138/pcutils
- **R** (Runtime environment (R >= 4.0) in which the MetaNet RMT thresholding workflow executes)

## Examples

```
library(MetaNet); rmt_res <- RMT_threshold(occor.r, out_dir = "rmt", min_threshold = 0.5, max_threshold = 0.9, step = 0.01); net <- c_net_build(cor, r_threshold = rmt_res$r_threshold)
```

## Evaluation signals

- Returned r_threshold lies within the swept [min_threshold, max_threshold] range and at a fixed step increment from min_threshold.
- rmt_threshold.txt exists in the RMT_threshold out_dir and contains the same numeric cutoff returned in the rmt_res object.
- The selected cutoff is the sweep point minimizing deviation of the thresholded correlation matrix's eigenvalue spacing from the RMT expectation (fewest spurious edges).
- The threshold is directly accepted as the r_threshold argument of c_net_build and yields a valid, connected co-occurrence network without manual adjustment.
- For reference comparisons, a default c_net_build(cor, r_threshold = 0.65) run on the same correlation object should reproduce the README quick-start network structure.

## Limitations

- The sweep resolution is bounded by the fixed step parameter, so the true optimum may fall between tested cutoffs.
- Correlation-based network construction (and hence RMT-thresholded edges) may overlook complex, non-linear, or indirect associations, per the article's discussion.
- The article notes a lack of standardized protocols for network construction, including thresholding, so RMT-derived cutoffs may differ across datasets and require domain validation.

## Evidence

- [readme] c_net_build threshold usage: [evidence span withheld — anchor status: external_doc]
