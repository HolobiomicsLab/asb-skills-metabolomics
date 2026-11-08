---
name: nap-webserver-baseline-comparison
description: 'Use when reproducing or validating performance claims for annotation
  propagation: you have a molecular spectral network (e.g. NIST 555-spectra benchmark)
  and need to compare measured ChemWalker runtime/scalability against the NAP webserver
  baseline.'
when_to_use_negative:
- The goal is to reproduce or improve NAP's own annotation-quality numbers rather
  than compare a ChemWalker implementation against the published NAP baseline.
- Input is raw mass spectrometry data or a spectral network that has not yet been
  processed with MetFrag fragmentation and MetFusion scoring — the weighted graph
  required by the random walk does not yet exist.
- Statistical differentiation of annotation accuracy (not runtime/scalability) is
  the target, since top-10 candidate classification cannot be statistically differentiated
  from NAP on this benchmark.
tools:
- name: ChemWalker
  role: Python library whose random-walk propagation over the weighted NIST spectral
    network is timed and compared against the NAP baseline
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: Network Annotation Propagation (NAP)
  role: Reference baseline whose webserver performance (running time, scalability,
    maintainability) is documented in the discussion section and used for comparison
- name: pyrwr
  role: Underlying implementation used by ChemWalker to compute the random walk with
    restart scores that are being timed
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Provides combinatorial in silico fragmentation results for candidate structures
    that form the candidate set of the benchmark network
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Scoring function used to create the weight function producing the weighted
    graph over which propagation is benchmarked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:5693b111ef9f09ec5c3305cfdade1b311ecff59e363b3d72f821d3ceace6b3ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - Network Annotation Propagation (NAP)
  - pyrwr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# nap-webserver-baseline-comparison

## Summary

Benchmarks ChemWalker's random-walk annotation propagation (pyrwr implementation) against the NAP webserver baseline on the NIST 555-spectra molecular network, measuring running time, scalability and maintainability to validate the article's claim that ChemWalker improves on NAP.

## When to use

Use when reproducing or validating performance claims for annotation propagation: you have a molecular spectral network (e.g. NIST 555-spectra benchmark) and need to compare measured ChemWalker runtime/scalability against the NAP webserver baseline.

## When NOT to use

- The goal is to reproduce or improve NAP's own annotation-quality numbers rather than compare a ChemWalker implementation against the published NAP baseline.
- Input is raw mass spectrometry data or a spectral network that has not yet been processed with MetFrag fragmentation and MetFusion scoring — the weighted graph required by the random walk does not yet exist.
- Statistical differentiation of annotation accuracy (not runtime/scalability) is the target, since top-10 candidate classification cannot be statistically differentiated from NAP on this benchmark.

## Inputs

- Processed NIST 555-spectra spectral network file (weighted molecular network with MetFusion scoring)
- MetFrag in silico fragmentation results for candidate structures
- Spectral library match seed nodes
- NAP webserver baseline performance documented in the manuscript's discussion section

## Outputs

- comparison_results.csv with columns for runtime, scalability, and maintainability notes
- Measured ChemWalker timing metrics (wall-clock running time, memory usage, nodes processed)
- Top-10 candidate classification results for statistical comparison against NAP

## How to apply

Create the dedicated conda environment from environment.yml and install ChemWalker, whose random walk relies on the pyrwr implementation of Random Walk with Restart. Load the processed NIST 555-spectra network built from MetFrag candidate structures and the MetFusion-derived weight function, then run ChemWalker's random-walk propagation over the weighted graph from seed nodes (spectral library matches) while recording wall-clock time, memory usage and number of nodes processed to characterize scalability. Retrieve the NAP webserver baseline performance documented in the manuscript's discussion section and assemble both into a comparison_results.csv table with columns for runtime, scalability and maintainability notes. Judge success by confirming the measured running time is consistent with the article's qualitative claim of improvement over NAP, while noting the reported result that ChemWalker's top-10 candidate classification cannot be statistically differentiated from NAP on this benchmark.

## Related tools

- **ChemWalker** (Python library whose random-walk propagation over the weighted NIST spectral network is timed and compared against the NAP baseline) — https://github.com/computational-chemical-biology/ChemWalker
- **Network Annotation Propagation (NAP)** (Reference baseline whose webserver performance (running time, scalability, maintainability) is documented in the discussion section and used for comparison)
- **pyrwr** (Underlying implementation used by ChemWalker to compute the random walk with restart scores that are being timed) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Provides combinatorial in silico fragmentation results for candidate structures that form the candidate set of the benchmark network) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Scoring function used to create the weight function producing the weighted graph over which propagation is benchmarked)

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; pyrwr --query-type rwr --graph-type undirected --input-path network.tsv --output-path scores.tsv --seeds 0 --device cpu
```

## Evaluation signals

- Measured ChemWalker wall-clock running time on the NIST 555-spectra network is consistent with the article's qualitative claim of improvement in running time over NAP.
- Recorded memory usage and number of nodes processed demonstrate the claimed scalability improvement over the NAP webserver.
- comparison_results.csv contains both ChemWalker measured metrics and the NAP baseline with runtime, scalability and maintainability columns populated.
- Top-10 candidate classification from ChemWalker cannot be statistically differentiated from NAP (matching the reported finding), confirming the benchmark reproduces the published trade-off.

## Limitations

- Top-10 candidate classification on the NIST 555-spectra benchmark cannot be statistically differentiated between ChemWalker and NAP; the claimed improvements are in running time, scalability and maintainability, not annotation accuracy.
- The improvement claim over NAP is qualitative in the article; the NAP baseline is a webserver whose measured performance is taken from the manuscript's discussion rather than a directly reproducible local run.
- No changelog was found for the repository, so version drift between the tested MetFrag2.3-CL.jar / ChemWalker release and newer code may affect reproducing the original timings.

## Evidence

- [abstract] Compared with NAP, ChemWalker has a series of improvements, on running time, scalability and maintainability and is available as a standalone python package: "Compared with NAP, ChemWalker has a series of improvements, on running time, scalability and maintainability and is available as a standalone python package"
- [abstract] the process of propagating annotations from spectral library matches on molecular networks can be automated using Network Annotation Propagation (NAP): "the process of propagating annotations from spectral library matches on molecular networks can be automated using Network Annotation Propagation (NAP)"
- [methods] conda env create -f environment.yml conda activate chemwalker: "conda env create -f environment.yml
conda activate chemwalker"
- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by in silico fragmentation, using random walk: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random"
- [readme] pyrwr focuses on computing a single source RWR score vector w.r.t. a given query (seed) node: [evidence span withheld — anchor status: external_doc]
