---
name: benchmark-method-comparison-across-simulation-conditions
description: Use when a new pathway-enrichment method must be evaluated against competing
  topology-based methods, and statistical power needs to be estimated per simulation
  condition (e.g., by number of dysregulated genes) with standard errors across replicates.
when_to_use_negative:
- When the goal is a single-method enrichment analysis on real (non-simulated) data
  rather than a comparative power benchmark across methods
- When competing methods require cutoff-based gene filtering and the comparison design
  depends on no-cutoff configurations that cannot be run
- When the input networks lack gene-gene interaction structure needed by topology-based
  (third-generation) methods
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: R package implementing the NetGSA methodology used to run NetGSA.REHE.Cl on
    each simulated dataset
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: Topology-based pathway enrichment method whose REHE with clustering variant
    (NetGSA.REHE.Cl) is benchmarked against competitors
  repo: github.com/mikehellstern/netgsa
- name: Pathway-Express
  role: Competing topology-based method run in its no-cutoff configuration on the
    same simulated datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:291e9dd4ef99041aa604e6ea24990714b2ba066faa758ec05c84176f578bedf1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NetGSA
  - Pathway-Express
  - netgsa
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benchmark-method-comparison-across-simulation-conditions

## Summary

A simulation-based benchmarking skill that compares the statistical power of topology-based pathway enrichment methods (NetGSA.REHE.Cl from the netgsa R package vs DEGraph, CAMERA, and Pathway-Express without cut-offs) across pathway groups stratified by number of dysregulated genes.

## When to use

Use when a new pathway-enrichment method must be evaluated against competing topology-based methods, and statistical power needs to be estimated per simulation condition (e.g., by number of dysregulated genes) with standard errors across replicates.

## When NOT to use

- When the goal is a single-method enrichment analysis on real (non-simulated) data rather than a comparative power benchmark across methods
- When competing methods require cutoff-based gene filtering and the comparison design depends on no-cutoff configurations that cannot be run
- When the input networks lack gene-gene interaction structure needed by topology-based (third-generation) methods

## Inputs

- Simulated gene expression datasets derived from a real dataset (e.g., breast cancer) with varying numbers of dysregulated genes per pathway group
- Pathway/network definitions (e.g., curated gene-gene interaction information from KEGG, Reactome, PantherDB)
- FDR threshold alpha=0.05

## Outputs

- Method-by-group power table with methods as rows and dysregulated-gene-count groups as columns, including standard errors (e.g., fig5b_power_table.csv)
- Empirical power estimates (fraction of true dysregulated pathways recovered) per method per simulation group

## How to apply

Set up simulation frameworks (e.g., on the breast cancer dataset) that vary the number of dysregulated genes per pathway group, then run NetGSA.REHE.Cl from the netgsa R package on each simulated dataset with an FDR cutoff of alpha=0.05. Run the competing topology-based methods DEGraph, CAMERA, and Pathway-Express in their no-cutoff configurations on the same simulated datasets. For each method, classify a pathway as detected when its p-value passes the FDR alpha=0.05 threshold and compute empirical power as the fraction of true dysregulated pathways recovered within each number-of-dysregulated-genes group. Aggregate to mean power per method per group with standard errors across simulation replicates, and write the method-by-group power table (methods as rows, dysregulated-gene-count groups as columns) to a CSV such as fig5b_power_table.csv.

## Related tools

- **netgsa** (R package implementing the NetGSA methodology used to run NetGSA.REHE.Cl on each simulated dataset) — github.com/mikehellstern/netgsa
- **NetGSA** (Topology-based pathway enrichment method whose REHE with clustering variant (NetGSA.REHE.Cl) is benchmarked against competitors) — github.com/mikehellstern/netgsa
- **Pathway-Express** (Competing topology-based method run in its no-cutoff configuration on the same simulated datasets)

## Examples

```
devtools::install_github("mikehellstern/netgsa", build_vignettes=T); results <- NetGSA(x, pathways, group, adjacency.list)
```

## Evaluation signals

- The power table reproduces the Fig 5B pattern: NetGSA.REHE.Cl competitive with the top-performing method in all dysregulated-gene groups
- DEGraph shows stronger power at low dysregulation levels, while CAMERA and Pathway-Express (no cutoff) are stronger at high dysregulation levels
- Standard errors across simulation replicates are computed and reported alongside mean power for every method-group cell
- Each detected pathway corresponds to a p-value passing the FDR alpha=0.05 threshold, and power equals the fraction of true dysregulated pathways recovered within each group

## Limitations

- There is no consistent best choice among topology-based methods for a given problem, so relative rankings depend on the simulation condition
- The netgsa implementation removed a compatibility check between adjacency matrices and input data, so users should verify variable compatibility before running NetGSA
- Before running NetGSA, input data matrices should be filtered to keep only genes belonging to at least one tested pathway

## Evidence

- [results] task_003 workflow: [evidence span withheld — anchor status: not_found]
- [intro] intro: "Many topology-based methods have been proposed in the literature, but there is no consistent best choice for a given problem."
- [readme] readme: [evidence span withheld — anchor status: external_doc]
