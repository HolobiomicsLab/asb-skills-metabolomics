---
name: spectral-network-propagation-benchmarking
description: Use when an article claims runtime, scalability or maintainability improvements
  of an annotation-propagation tool over a baseline (e.g. ChemWalker vs NAP) and you
  must reproduce those performance claims quantitatively.
when_to_use_negative:
- The research question concerns annotation accuracy alone with no performance claim
  to reproduce (e.g. comparing top-N candidate rankings without runtime/scalability
  metrics).
- The baseline tool's performance is reported on different hardware or different network
  sizes than your benchmark, making a like-for-like runtime comparison invalid.
- The input is not a molecular network with candidate structures (e.g. raw spectra
  without MetFrag in silico fragmentation results).
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ChemWalker
  role: Tool under benchmark; its random-walk propagation timing is measured against
    NAP's reported performance
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: Network Annotation Propagation (NAP)
  role: Baseline whose reported running time, scalability and maintainability are
    used as the comparison reference
- name: pyrwr
  role: Implements the random walk with restart power iteration whose computation
    is timed during benchmarking
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Provides the combinatorial in silico fragmentation candidate structures that
    form the benchmarked network's nodes
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Scoring function used to build the weighted graph whose propagation is benchmarked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:f87327fee6ddb4906cbcd8d4a7f368acef33b19cf20a673c2f317a29cf7b7127
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spectral-network-propagation-benchmarking

## Summary

Benchmarking skill for reproducing runtime, scalability and maintainability comparisons between ChemWalker's pyrwr-based random-walk annotation propagation and the NAP webserver baseline on molecular networks, using the NIST 555-spectra benchmark as reference data.

## When to use

Use when an article claims runtime, scalability or maintainability improvements of an annotation-propagation tool over a baseline (e.g. ChemWalker vs NAP) and you must reproduce those performance claims quantitatively.

## When NOT to use

- The research question concerns annotation accuracy alone with no performance claim to reproduce (e.g. comparing top-N candidate rankings without runtime/scalability metrics).
- The baseline tool's performance is reported on different hardware or different network sizes than your benchmark, making a like-for-like runtime comparison invalid.
- The input is not a molecular network with candidate structures (e.g. raw spectra without MetFrag in silico fragmentation results).

## Inputs

- Processed NIST 555-spectra molecular network file (weighted graph from MetFrag candidates + MetFusion scoring)
- ChemWalker conda environment definition (environment.yml)
- NAP webserver baseline performance figures from the manuscript's discussion section
- pyrwr input edge list (source \t target \t weight format, positive weights)

## Outputs

- comparison_results.csv with columns for runtime, scalability, and maintainability notes
- Measured wall-clock running time, memory usage, and number of nodes processed for ChemWalker propagation
- Statistical comparison of top-10 candidate classifications (ChemWalker vs NAP) when annotation quality is co-evaluated

## How to apply

Set up the tool's dedicated conda environment (conda env create -f environment.yml; conda activate chemwalker) so the benchmarked implementation matches the published dependency stack. Time the core computation — ChemWalker's random-walk propagation via pyrwr over the weighted molecular network built from MetFrag candidate structures and MetFusion scoring — recording wall-clock running time, peak memory usage, and number of nodes processed to characterize scalability. Locate the baseline (NAP webserver) performance figures documented in the manuscript's discussion section and assemble measured vs. reported metrics into a single comparison table (comparison_results.csv) with runtime, scalability, and maintainability notes. When benchmarking annotation quality alongside speed, evaluate top-10 candidate classification against the baseline with appropriate statistical tests, because ChemWalker's top-10 candidates on the NIST 555-spectra network cannot be statistically differentiated from NAP — the differentiating claim is performance, not accuracy. Validate that measured timings are directionally consistent with the article's qualitative improvement claim before accepting the reproduction.

## Related tools

- **ChemWalker** (Tool under benchmark; its random-walk propagation timing is measured against NAP's reported performance) — https://github.com/computational-chemical-biology/ChemWalker
- **Network Annotation Propagation (NAP)** (Baseline whose reported running time, scalability and maintainability are used as the comparison reference)
- **pyrwr** (Implements the random walk with restart power iteration whose computation is timed during benchmarking) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Provides the combinatorial in silico fragmentation candidate structures that form the benchmarked network's nodes) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Scoring function used to build the weighted graph whose propagation is benchmarked)

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- Measured ChemWalker wall-clock running time is consistent with the article's qualitative claim of improvement in running time, scalability and maintainability over NAP.
- comparison_results.csv contains one row pairing measured ChemWalker metrics with the NAP baseline for runtime, scalability, and maintainability notes.
- Number of nodes processed and memory usage are recorded alongside time so scalability is characterized, not just speed.
- Top-10 candidate classification statistics show no statistically significant difference from NAP on the NIST 555-spectra network, matching the reported finding.
- Conda environment resolves cleanly from environment.yml, confirming the benchmarked stack matches the published one.

## Limitations

- ChemWalker's top-10 candidate classification on the NIST 555-spectra benchmark cannot be statistically differentiated from NAP; only performance (running time, scalability, maintainability) differentiates the tools.
- NAP baseline performance is reported in the manuscript discussion and may not have been re-measured under the same conditions, so runtime comparison rests partly on reported rather than re-run figures.
- pyrwr RWR permits only a single seed per query and requires positively weighted networks; redundant edges are summed in the adjacency matrix.
- No changelog was found for the repository, complicating version-pinned reproduction.

## Evidence

- [abstract] ChemWalker improves on NAP in running time, scalability and maintainability, and is available as a standalone python package: "Compared with NAP, ChemWalker has a series of improvements, on running time, scalability and maintainability and is available as a standalone python package"
- [methods] conda environment setup from methods: "conda env create -f environment.yml
conda activate chemwalker"
- [intro] pyrwr random walk implementation from intro: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]"
- [readme] pyrwr readme on weighted graph input and single-seed RWR: [evidence span withheld — anchor status: external_doc]
- [abstract] weighted graph construction via MetFusion scoring: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function, producing a"
