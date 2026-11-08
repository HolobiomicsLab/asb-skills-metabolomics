---
name: metabolite-candidate-top10-ranking-evaluation
description: Use when ranking in silico-annotated candidate structures per mass spectrum
  and top-k (e.g., top-10) classification against benchmark library matches is needed,
  or when testing whether method variants change top-10 results.
when_to_use_negative:
- Candidate structures were not produced by MetFrag combinatorial in silico fragmentation,
  so no MetFusion-weighted graph can be built
- The dataset has no spectral library matches to serve as seed nodes for the random
  walk
- The analysis target is a different benchmark with a different ground-truth annotation
  set rather than the NIST 555-spectra top-10 classification task
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: ChemWalker
  role: Runs the random-walk annotation propagation over the MetFusion-weighted graph
    and produces ranked candidates per spectrum
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Computes random walk with restart scores from spectral-library-match seed
    nodes over each fingerprint-derived weighted graph
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Performs combinatorial in silico fragmentation of candidate structures for
    the NIST 555 benchmark (tested on MetFrag2.3-CL.jar)
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Supplies the scoring function that combines spectral network and structural
    similarity into the graph weight function
- name: Network Annotation Propagation (NAP)
  role: Reference method whose top-10 classification results define the benchmark
    equivalence that the evaluation reproduces
- name: RDKit
  role: Source of the 27 molecular fingerprint types used to compute pairwise candidate
    structural similarity
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:33d3996d5e88f784dd47f97f3342e62871e0fd3bb8975b1f855be1b0925284c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - Network Annotation Propagation (NAP)
  - RDKit
  - pyrwr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metabolite-candidate-top10-ranking-evaluation

## Summary

Evaluates metabolite annotation rankings by scoring top-10 candidate classification on the NIST 555-spectra benchmark, replicating the ChemWalker-versus-NAP equivalence test across RDKit fingerprint variants.

## When to use

Use when ranking in silico-annotated candidate structures per mass spectrum and top-k (e.g., top-10) classification against benchmark library matches is needed, or when testing whether method variants change top-10 results.

## When NOT to use

- Candidate structures were not produced by MetFrag combinatorial in silico fragmentation, so no MetFusion-weighted graph can be built
- The dataset has no spectral library matches to serve as seed nodes for the random walk
- The analysis target is a different benchmark with a different ground-truth annotation set rather than the NIST 555-spectra top-10 classification task

## Inputs

- NIST 555-spectra benchmark with spectral library matches (seed nodes)
- MetFrag candidate structure lists per spectrum
- MetFrag2.3-CL.jar in silico annotation results
- RDKit fingerprint definitions (27 fingerprint types)
- ChemWalker environment.yml specification

## Outputs

- Per-spectrum ranked candidate lists for each of the 27 fingerprint variants
- fingerprint_benchmark_results.csv with default-fingerprint baseline plus deviations of the other 26 fingerprints
- Top-10 candidate classification metrics for the ChemWalker-versus-NAP comparison

## How to apply

Set up the dedicated chemwalker conda environment from environment.yml and install ChemWalker via pip. Obtain MetFrag2.3-CL.jar and run MetFrag combinatorial in silico fragmentation on the candidate structure lists of the NIST 555-spectra benchmark. For each of the 27 RDKit fingerprint types, compute pairwise structural similarity among candidates and combine it with the spectral network via the MetFusion scoring function to build a weighted graph; run random-walk propagation (pyrwr) from the spectral-library-match seed nodes to compute candidate probabilities. Rank candidates per spectrum for every fingerprint variant and score top-10 classification against the reported ChemWalker–NAP equivalence. Aggregate per-fingerprint top-10 metrics into fingerprint_benchmark_results.csv, including the default-fingerprint baseline and the deviation of each of the other 26 fingerprints from it. Judge correctness by reproducing the statistical non-differentiation of ChemWalker from NAP reported in the Supplementary Material.

## Related tools

- **ChemWalker** (Runs the random-walk annotation propagation over the MetFusion-weighted graph and produces ranked candidates per spectrum) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Computes random walk with restart scores from spectral-library-match seed nodes over each fingerprint-derived weighted graph) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Performs combinatorial in silico fragmentation of candidate structures for the NIST 555 benchmark (tested on MetFrag2.3-CL.jar)) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Supplies the scoring function that combines spectral network and structural similarity into the graph weight function)
- **Network Annotation Propagation (NAP)** (Reference method whose top-10 classification results define the benchmark equivalence that the evaluation reproduces)
- **RDKit** (Source of the 27 molecular fingerprint types used to compute pairwise candidate structural similarity)

## Examples

```
conda env create -f environment.yml && conda activate chemwalker && pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- fingerprint_benchmark_results.csv contains the default-fingerprint baseline row plus the deviation of each of the other 26 RDKit fingerprints
- Top-10 classification for the default-fingerprint ChemWalker run is statistically indistinguishable from NAP, matching the Supplementary Material equivalence claim
- Every fingerprint variant produces a positively weighted graph usable by pyrwr (RWR requires positive edge weights; redundant edge weights are summed)
- Ranked candidate lists are generated for all 555 benchmark spectra, with seed nodes restricted to spectral library matches

## Limitations

- The skill evaluates equivalence to NAP rather than superiority: on the NIST 555-spectra top-10 comparison, ChemWalker cannot be statistically differentiated from NAP
- ChemWalker was tested specifically on MetFrag2.3.-CL.jar; other MetFrag versions are untested
- NAP's limitation that information propagates only locally to first neighbors of a spectral match constrains what top-10 equivalence testing can reveal

## Evidence

- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation"
- [abstract] MetFusion weight function: "we have used MetFusion Scoring function to create a weight function, producing a"
- [abstract] random walk from seed nodes: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes"
- [methods] ChemWalker tested on MetFrag2.3.-CL.jar: "ChemWalker was tested on MetFrag2.3.-CL.jar"
- [readme] ChemWalker installation via pip: "pip install git+https://github.com/computational-chemical-biology/ChemWalker.git"
- [abstract] ChemWalker improvements over NAP: "Compared with NAP, ChemWalker has a series of improvements, on running time, scalability and maintainability and is available as a standalone python package"
