---
name: random-walk-runtime-profiling
description: Use when you need to benchmark runtime, memory and scalability of random-walk
  annotation propagation (e.g. ChemWalker vs NAP) on a molecular network benchmark,
  or validate performance claims about a propagation tool.
when_to_use_negative:
- Your goal is to compare annotation accuracy (top-10 candidate classification) rather
  than runtime — ChemWalker's top-10 classification cannot be statistically differentiated
  from NAP on this benchmark.
- The molecular network has not been preprocessed into the weighted graph format (MetFrag
  candidates plus MetFusion weights) required by the pyrwr-based propagation.
- You are profiling a propagation method other than a random-walk implementation (e.g.
  NAP itself) whose execution path differs from the pyrwr power-iteration pipeline.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ChemWalker
  role: Python package whose random-walk propagation is being timed; installed into
    a dedicated conda environment from environment.yml
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Implements the random walk with restart (power iteration) over the weighted
    molecular network; the code path whose runtime is profiled
  repo: https://github.com/jinhongjung/pyrwr
- name: Network Annotation Propagation (NAP)
  role: Baseline webserver whose reported performance is used for the runtime/scalability/maintainability
    comparison
- name: MetFrag
  role: Provides the in silico fragmentation candidate structures that, combined with
    MetFusion scoring, form the weighted graph being propagated
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:675b8b68d972c30d73e2db0274d13957a39cc60d7f3d750a0fcf95368fcca3ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - Network Annotation Propagation (NAP)
  - pyrwr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# random-walk-runtime-profiling

## Summary

Profile ChemWalker's random-walk propagation (pyrwr implementation) over the NIST 555-spectra weighted molecular network to measure wall-clock running time, memory usage and scalability, and compare against the NAP webserver baseline to validate the paper's qualitative improvement claims.

## When to use

Use when you need to benchmark runtime, memory and scalability of random-walk annotation propagation (e.g. ChemWalker vs NAP) on a molecular network benchmark, or validate performance claims about a propagation tool.

## When NOT to use

- Your goal is to compare annotation accuracy (top-10 candidate classification) rather than runtime — ChemWalker's top-10 classification cannot be statistically differentiated from NAP on this benchmark.
- The molecular network has not been preprocessed into the weighted graph format (MetFrag candidates plus MetFusion weights) required by the pyrwr-based propagation.
- You are profiling a propagation method other than a random-walk implementation (e.g. NAP itself) whose execution path differs from the pyrwr power-iteration pipeline.

## Inputs

- Processed NIST 555-spectra spectral network file
- Weighted molecular network derived from MetFrag candidate structures and MetFusion scoring
- ChemWalker conda environment (environment.yml)
- NAP webserver baseline performance values from the manuscript discussion

## Outputs

- comparison_results.csv (columns: runtime, scalability, maintainability notes)
- Measured wall-clock running time of ChemWalker's random-walk propagation
- Memory usage and node counts characterizing scalability

## How to apply

Load the dedicated conda environment (conda env create -f environment.yml; conda activate chemwalker) and the processed NIST 555-spectra spectral network file, whose weighted graph is built from MetFrag candidate structures combined with MetFusion scoring. Time the random-walk propagation performed with pyrwr (RWR via power iteration, restart probability c=0.15, epsilon=1e-9, max_iters=100 by default), recording wall-clock time, peak memory usage and number of nodes processed. Compare the measured metrics against the NAP webserver baseline performance documented in the article's discussion, and assemble both into comparison_results.csv with runtime, scalability and maintainability columns. Correctness is judged by consistency between the measured running time and the paper's claim that ChemWalker improves on NAP in running time, scalability and maintainability, while top-10 candidate classification remains statistically indistinguishable from NAP.

## Related tools

- **ChemWalker** (Python package whose random-walk propagation is being timed; installed into a dedicated conda environment from environment.yml) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Implements the random walk with restart (power iteration) over the weighted molecular network; the code path whose runtime is profiled) — https://github.com/jinhongjung/pyrwr
- **Network Annotation Propagation (NAP)** (Baseline webserver whose reported performance is used for the runtime/scalability/maintainability comparison)
- **MetFrag** (Provides the in silico fragmentation candidate structures that, combined with MetFusion scoring, form the weighted graph being propagated) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; python -c "from pyrwr.rwr import RWR; rwr=RWR(); rwr.read_graph('nist555.tsv','undirected'); r=rwr.compute(seed,0.15,1e-9,100)"
```

## Evaluation signals

- Measured wall-clock running time is consistent with the article's qualitative claim that ChemWalker improves on NAP in running time, scalability and maintainability.
- comparison_results.csv contains rows for both ChemWalker measured metrics and the NAP baseline, with populated runtime, scalability and maintainability columns.
- pyrwr run completes within its documented parameters (c=0.15 restart probability, epsilon=1e-9 error tolerance, max_iters=100) without convergence failure.
- Node count recorded matches the size of the NIST 555-spectra benchmark network, confirming the whole graph was processed.
- Top-10 candidate classification results remain statistically indistinguishable from NAP, confirming profiling did not alter annotation output.

## Limitations

- Only wall-clock time, memory and node counts are directly measurable; the NAP baseline comes from the manuscript's discussion section rather than a re-run, so the comparison mixes measured and reported values.
- The improvement claim is qualitative ('a series of improvements, on running time, scalability and maintainability'); on the NIST 555-spectra benchmark, ChemWalker's top-10 candidate classification cannot be statistically differentiated from NAP.
- pyrwr's RWR requires positively weighted networks and only positive weights are allowed; GPU mode requires the graph to fit in GPU memory.

## Evidence

- [abstract] Compared with NAP, ChemWalker has a series of improvements, on running time, scalability and maintainability and is available as a standalone python package: "Compared with NAP, ChemWalker has a series of improvements, on running time, scalability and maintainability and is available as a standalone python package"
- [intro] using [random walk] (pyrwr): "using [random walk](https://github.com/jinhongjung/pyrwr)"
- [methods] conda env creation: "conda env create -f environment.yml
conda activate chemwalker"
- [abstract] MetFusion weight function: "we have used MetFusion Scoring function to create a weight function, producing a weighted graph"
- [readme] pyrwr compute parameters: [evidence span withheld — anchor status: external_doc]
