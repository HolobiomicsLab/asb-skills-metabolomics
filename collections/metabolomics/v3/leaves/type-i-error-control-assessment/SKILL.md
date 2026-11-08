---
name: type-i-error-control-assessment
description: Use when evaluating whether a pathway enrichment method (e.g. NetGSA
  REHE or REML) maintains type-I error control under null conditions while retaining
  power on dysregulated gene groups in simulated replicate studies.
when_to_use_negative:
- When the goal is running pathway enrichment on real (non-simulated) expression data
  rather than assessing method-level statistical properties
- When no null (undysregulated) condition can be simulated, since type-I error control
  cannot be evaluated without a 'None' dysregulation group
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: netgsa
  role: R package providing NetGSA.REHE (with and without clustering) and REML implementations
    used to compute test statistics and pathway-level calls per replicate
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: Overhauled topology-based method whose statistical power and type-I error
    control are being evaluated in the simulation
  repo: github.com/mikehellstern/netgsa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:3cae68d77d17278c540a3e938684cd62a33e344f71a0da88992db2e093e30225
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NetGSA
  - netgsa
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# type-i-error-control-assessment

## Summary

Empirically assess type-I error control (and power) of topology-based pathway enrichment methods — here NetGSA REHE (with and without clustering) vs REML — by simulating null and dysregulated signal conditions and measuring mean power per dysregulation-group bin.

## When to use

Use when evaluating whether a pathway enrichment method (e.g. NetGSA REHE or REML) maintains type-I error control under null conditions while retaining power on dysregulated gene groups in simulated replicate studies.

## When NOT to use

- When the goal is running pathway enrichment on real (non-simulated) expression data rather than assessing method-level statistical properties
- When no null (undysregulated) condition can be simulated, since type-I error control cannot be evaluated without a 'None' dysregulation group

## Inputs

- Gene expression dataset (e.g. breastcancer2012), centered and scaled to unit variance
- Pathway/gene interaction network definitions (e.g. from KEGG, Reactome, PantherDB)
- Simulated dysregulation signal levels (0.2, 0.3, 0.4) applied to gene groups of varying sizes
- Multiple simulated replicates per dysregulation framework

## Outputs

- Per-pathway empirical power estimates for each method (REHE with clustering, REHE without clustering, REML)
- Aggregated mean power (+/- SE) table within dysregulation-group bins (None, (0,5], (5,10], >10)
- power_summary_fig5a.csv — aggregated power summary table

## How to apply

Load a gene expression dataset (e.g. breastcancer2012) and center/scale it to unit variance. Simulate dysregulated-gene signals under multiple dysregulation frameworks by adding mean signals of 0.2, 0.3, and 0.4 to groups of dysregulated genes of varying sizes, keeping a 'None' group as the null condition. For each replicate and pathway, run the competing estimators — NetGSA.REHE with clustering of genes into subnetworks, NetGSA.REHE without clustering, and REML — to obtain test statistics and pathway-level calls. Estimate per-pathway empirical power across replicates, then aggregate mean power (+/- standard error) within dysregulation-group bins (None, (0,5], (5,10], >10). Correct application is judged by the 'None' bin showing mean power of approximately 0 for all methods (type-I error well controlled), while dysregulated bins should show REHE with clustering achieving higher mean power than REHE alone or REML.

## Related tools

- **netgsa** (R package providing NetGSA.REHE (with and without clustering) and REML implementations used to compute test statistics and pathway-level calls per replicate) — github.com/mikehellstern/netgsa
- **NetGSA** (Overhauled topology-based method whose statistical power and type-I error control are being evaluated in the simulation) — github.com/mikehellstern/netgsa

## Examples

```
library(netgsa); data(breastcancer2012); X <- scale(X); res <- NetGSA.REHE(x, A, pathways, clustering=TRUE)
```

## Evaluation signals

- Mean power in the 'None' dysregulation group must be approximately 0 for all three methods, demonstrating type-I error control
- Mean power should increase with dysregulation signal magnitude (0.2 < 0.3 < 0.4) across the dysregulated gene-group bins
- REHE with clustering should show higher mean power than REHE without clustering and REML in all non-null dysregulation bins
- Aggregated output table matches the expected schema of power_summary_fig5a.csv with bins None, (0,5], (5,10], >10
- Standard errors of mean power across replicates should be reported alongside means

## Limitations

- Results depend on the specific simulated dysregulation frameworks and signal magnitudes (0.2, 0.3, 0.4); power/type-I error conclusions may not transfer to other simulation designs
- The README notes the adjacency-matrix compatibility check between groups was removed for block-diagonal structures and 'should be added later', so input validation is incomplete
- Variables in the input data matrix must be filtered to keep only those belonging to at least one tested pathway before running NetGSA

## Evidence

- [results] None bin null control: [evidence span withheld — anchor status: not_found]
- [methods] workflow simulation design: [evidence span withheld — anchor status: not_found]
- [abstract] power without sacrifice claim: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [readme] input filtering requirement: [evidence span withheld — anchor status: external_doc]
