---
name: gnps-spectral-network-seeding
description: Use when you have a GNPS-derived molecular network plus a spectral library
  annotation table, and you need to designate a limited subset of seed nodes to test
  or run annotation propagation over candidate structures.
when_to_use_negative:
- The network has no spectral library matches at all — there are no seed nodes to
  depart from, so propagation cannot start
- You already have annotation for every node directly, and only need a single-source
  annotation rather than propagation from limited seeds
- Your goal is de novo structure elucidation from a spectrum alone rather than propagating
  library-match identities through a candidate-structure network
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ChemWalker
  role: Python package that loads the GNPS network and propagates spectral library
    match identities through candidate structures via random walk
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: Network Annotation Propagation (NAP)
  role: Baseline propagation method used for comparison on the same GNPS-seeded network
    and candidate lists
- name: pyrwr
  role: Random walk engine ChemWalker uses to compute RWR probabilities from the seed
    nodes over the weighted graph
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Produces the in silico combinatorial fragmentation candidates whose structural
    similarity forms the weighted graph
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Supplies the scoring function used to create the weight function combining
    the spectral network and structural similarity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:12bc23f0b2f23f2dae5ebc822f71a0034c484461ce55bd2a0f14d2fbcacc3343
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - GNPS (Global Natural Products Social Molecular Networking)
  - MetFrag
  - MetFusion
  - Network Annotation Propagation (NAP)
  - pyrwr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# gnps-spectral-network-seeding

## Summary

Seed a GNPS molecular network with spectral library matches so that a small fraction (~10%) of the 555-spectra NIST benchmark network acts as starting nodes for downstream annotation propagation (ChemWalker random walks or NAP).

## When to use

Use when you have a GNPS-derived molecular network plus a spectral library annotation table, and you need to designate a limited subset of seed nodes to test or run annotation propagation over candidate structures.

## When NOT to use

- The network has no spectral library matches at all — there are no seed nodes to depart from, so propagation cannot start
- You already have annotation for every node directly, and only need a single-source annotation rather than propagation from limited seeds
- Your goal is de novo structure elucidation from a spectrum alone rather than propagating library-match identities through a candidate-structure network

## Inputs

- GNPS-derived molecular network of 555 spectra (NIST benchmark)
- Spectral library match annotations identifying which spectra have library identities
- Candidate structures from MetFrag combinatorial in silico fragmentation
- MetFusion scoring function weights for structural similarity among candidate structures

## Outputs

- Seeded graph: weighted graph combining the spectral network and structural similarity, with a designated ~10% seed-node subset
- Random-walk candidate probability scores departing from the seed nodes
- Top-10 candidate classification per unseeded node, comparable against NAP output

## How to apply

Load the GNPS-derived NIST network containing 555 spectra into the ChemWalker python package environment. From the spectral library match results, select at most ~10% of the spectra as seed nodes, mirroring the benchmark design in which candidate classification performance was tested under limited seeds. Build the weighted graph combining the spectral network with structural similarity among candidate structures using the MetFusion scoring function, with candidate structures produced by MetFrag combinatorial in silico fragmentation. Departing from the seed nodes (represented by spectral library matches), execute the random walk via pyrwr to compute candidate probabilities over the weighted graph. Correct application is judged by whether the seeded subset reproduces the reported benchmark behavior: top-10 candidate classifications statistically comparable to NAP on the same seed set.

## Related tools

- **ChemWalker** (Python package that loads the GNPS network and propagates spectral library match identities through candidate structures via random walk) — https://github.com/computational-chemical-biology/ChemWalker
- **Network Annotation Propagation (NAP)** (Baseline propagation method used for comparison on the same GNPS-seeded network and candidate lists)
- **pyrwr** (Random walk engine ChemWalker uses to compute RWR probabilities from the seed nodes over the weighted graph) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Produces the in silico combinatorial fragmentation candidates whose structural similarity forms the weighted graph) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Supplies the scoring function used to create the weight function combining the spectral network and structural similarity)

## Examples

```
conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- The seeded subset is at most ~10% of the 555 NIST spectra (≈55 seed nodes), matching the benchmark design
- The weighted graph is built using the MetFusion scoring function before the random walk is executed
- Random walk probabilities are computed departing only from seed nodes represented by spectral library matches
- Top-10 candidate classifications from the seeded propagation cannot be statistically differentiated from NAP on the same 555-spectra network
- The random walk runs through pyrwr, not a bespoke propagation routine

## Limitations

- One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match
- Annotation of the mass signals is still the biggest bottleneck for the untargeted mass spectrometry analysis of complex mixtures
- ChemWalker was tested specifically on MetFrag2.3.-CL.jar; other MetFrag release versions may behave differently

## Evidence

- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using random walk: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using random walk"
- [abstract] Seed nodes represented by spectral library matches feed the random walk over the weighted MetFusion graph: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [abstract] Departing from the combination of a spectral network and structural similarity, MetFusion scoring produces the weight function: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function"
- [abstract] ChemWalker relies on MetFrag combinatorial in silico fragmentation searching biologically relevant databases: "ChemWalker relies on combinatorial in silico fragmentation results, performed by MetFrag, searching biologically relevant databases"
- [abstract] One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match: "One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match"
- [readme] ChemWalker was tested on MetFrag2.3.-CL.jar: "ChemWalker was tested on MetFrag2.3.-CL.jar"
- [results] Comparing the classification of the 10 first candidates on the 555-spectra NIST network, ChemWalker cannot be statistically differentiated from NAP: [evidence span withheld — anchor status: not_found]
