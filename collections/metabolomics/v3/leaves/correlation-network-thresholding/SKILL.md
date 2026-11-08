---
name: correlation-network-thresholding
description: Use when building a correlation-based co-occurrence network from an omics
  feature table and you need an objective, data-driven r_threshold instead of a manually
  chosen cutoff (e.g. 0.6/0.65) to avoid biasing topology.
when_to_use_negative:
- You already have a biologically or literature-motivated fixed cutoff and only need
  to build the network — then call c_net_build() directly with that r_threshold instead
  of running RMT.
- The input is a raw feature abundance table; the skill operates on a precomputed
  correlation matrix, not the count table itself.
- You need multi-layer network assembly across omics layers — use multi_net_build()
  rather than single-matrix RMT thresholding.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: MetaNet
  role: R package providing RMT-based threshold estimation (rmt()/RMT_threshold())
    and downstream network construction (c_net_build())
  repo: github.com/Asa12138/MetaNet
- name: R
  role: Runtime environment in which MetaNet's correlation calculation, RMT threshold
    sweep, and network build are executed
  repo: https://github.com/Asa12138/MetaNet
- name: igraph
  role: Underlying graph library MetaNet builds network objects on, used after the
    RMT cutoff selects edges
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:7d246028cf25f42476cb8ef5929c8b5e97f08ec01c335f510f61ab26fb3271da
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Correlation network thresholding

## Summary

Objective, data-driven selection of the correlation cutoff (r_threshold) that defines edges in a co-occurrence network, replacing arbitrary manual cutoffs. The article frames subjective thresholding as a key gap and implements random matrix theory (RMT) in MetaNet to identify the cutoff that minimizes spurious edges.

## When to use

Use when building a correlation-based co-occurrence network from an omics feature table and you need an objective, data-driven r_threshold instead of a manually chosen cutoff (e.g. 0.6/0.65) to avoid biasing topology.

## When NOT to use

- You already have a biologically or literature-motivated fixed cutoff and only need to build the network — then call c_net_build() directly with that r_threshold instead of running RMT.
- The input is a raw feature abundance table; the skill operates on a precomputed correlation matrix, not the count table itself.
- You need multi-layer network assembly across omics layers — use multi_net_build() rather than single-matrix RMT thresholding.

## Inputs

- Correlation matrix/correlation object (occor.r) from c_net_calculate() on a transposed omics feature table (e.g. totu)
- Sweep parameters: min_threshold, max_threshold, step for candidate correlation cutoffs
- Output directory (out_dir) for RMT results

## Outputs

- Objective r_threshold value within [min_threshold, max_threshold]
- rmt_res object (plottable via plot.rmt_res) describing the threshold sweep
- rmt_threshold.txt file written to out_dir
- Correlation network built via c_net_build(corr, r_threshold = <RMT cutoff>)

## How to apply

Start from a correlation matrix produced by c_net_calculate() (Pearson or Spearman with p-value adjustment via p.adjust.method/p.adjust.mode) rather than a network object. Pass this correlation object as occor.r to the RMT function (rmt()/RMT_threshold()) and sweep candidate cutoffs from min_threshold to max_threshold at a fixed step; at each candidate cutoff evaluate the eigenvalue-spacing statistic of the thresholded correlation matrix and select the r_threshold that minimizes the deviation from the random-matrix-theory expectation, i.e. the cutoff that minimizes spurious edges. Visualize the sweep with plot.rmt_res(x, ...), optionally enabling the animated threshold scan via the gif flag, and write the identified r_threshold to rmt_threshold.txt in the chosen out_dir. Use the returned value directly as the r_threshold argument of c_net_build(corr, r_threshold, ...) so that only correlations at or above the RMT-selected cutoff become edges; confirm the threshold falls within [min_threshold, max_threshold] as a sanity check.

## Related tools

- **MetaNet** (R package providing RMT-based threshold estimation (rmt()/RMT_threshold()) and downstream network construction (c_net_build())) — github.com/Asa12138/MetaNet
- **R** (Runtime environment in which MetaNet's correlation calculation, RMT threshold sweep, and network build are executed) — https://github.com/Asa12138/MetaNet
- **igraph** (Underlying graph library MetaNet builds network objects on, used after the RMT cutoff selects edges)

## Examples

```
cor <- c_net_calculate(totu); rmt_res <- RMT_threshold(cor, out_dir="rmt/", min_threshold=0.3, max_threshold=0.9, step=0.01); net <- c_net_build(cor, r_threshold=rmt_res$r_threshold)
```

## Evaluation signals

- Returned r_threshold lies within the configured [min_threshold, max_threshold] interval
- rmt_threshold.txt is produced in the specified out_dir containing the selected cutoff
- The RMT-selected threshold, when passed to c_net_build(), yields a network whose edge set is not populated by correlations below the cutoff (p-value adjustment state checked — build warns if p-values were not adjusted)
- The sweep can be replotted with plot.rmt_res(x, ...) and the eigenvalue-spacing statistic shows a minimum at the selected cutoff

## Limitations

- The RMT sweep is sensitive to the chosen min_threshold, max_threshold and step; an inappropriate range can miss the true cutoff.
- MetaNet warns 'Have not do p-value adjustment!' when c_net_build() is called without adjusted p-values, so edge selection depends on whether p.adjust was applied during c_net_calculate().
- The article notes no standardized protocols exist for thresholding and null model selection, and that correlation-based networks may overlook non-linear or indirect associations.
- No explicit RMT parameter defaults (step size, thresholds) are stated in the provided README, so users must supply them.

## Evidence

- [results] MetaNet addresses subjective thresholding by incorporating random matrix theory (RMT), which automatically determines a correlation cutoff (r_threshold) that minimizes spurious edges based on the structure of the data.: [evidence span withheld — anchor status: not_found]
- [readme] The README shows the manual-cutoff baseline the RMT skill replaces: a 0.65 r_threshold hard-coded in c_net_build().: "net <- c_net_build(cor, r_threshold = 0.65)
#> Have not do p-value adjustment! use the p.value to build network."
- [intro] The article flags subjective threshold selection as a bias source that this skill mitigates.: [evidence span withheld — anchor status: not_found]
- [methods] RMT_threshold writes the selected cutoff to a text file in a caller-specified output directory.: [evidence span withheld — anchor status: not_found]
