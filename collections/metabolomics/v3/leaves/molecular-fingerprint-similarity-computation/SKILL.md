---
name: molecular-fingerprint-similarity-computation
description: Decision support for choosing and computing structural fingerprints when
  building weighted molecular graphs from MetFrag candidate lists, or when testing
  fingerprint-choice sensitivity in annotation propagation benchmarks.
when_to_use_negative:
- When candidates have no computable RDKit representation or the structural-similarity
  matrix is already provided — recomputing fingerprints adds nothing
- When the goal is spectral-library matching rather than structural-similarity graph
  construction; seed nodes come from spectral library matches, not fingerprint similarity
- When only the default ChemWalker fingerprint behavior is needed with no sensitivity
  analysis — a single fingerprint suffices, not all 27
edam_operation: http://edamontology.org/operation_4003
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Source of the 27 molecular fingerprint types used to compute pairwise structural
    similarity among MetFrag candidate structures
- name: ChemWalker
  role: Python package whose pipeline combines fingerprint-derived structural similarity
    with the spectral network via the MetFusion scoring function
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: MetFusion
  role: Scoring function used to weight the graph from structural similarity and spectral
    network edges
- name: pyrwr
  role: Random walk with restart engine that propagates probabilities from spectral-library-match
    seed nodes over the fingerprint-derived weighted graph
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Provides the in silico combinatorial fragmentation candidate structures whose
    pairwise similarities are computed
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:3990b630a78611e6fdab200862e966ae43943ca0fbb9312bca5606e28bcebb55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - RDKit
  - pyrwr
  edam_operation: http://edamontology.org/operation_4003
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-fingerprint-similarity-computation

## Summary

Computes pairwise structural similarity among candidate molecular structures using RDKit molecular fingerprints (27 available types in ChemWalker's NIST 555-spectra benchmark), producing a similarity matrix that is combined with the spectral network via the MetFusion scoring function to build the weighted graph for random-walk annotation propagation.

## When to use

Decision support for choosing and computing structural fingerprints when building weighted molecular graphs from MetFrag candidate lists, or when testing fingerprint-choice sensitivity in annotation propagation benchmarks.

## When NOT to use

- When candidates have no computable RDKit representation or the structural-similarity matrix is already provided — recomputing fingerprints adds nothing
- When the goal is spectral-library matching rather than structural-similarity graph construction; seed nodes come from spectral library matches, not fingerprint similarity
- When only the default ChemWalker fingerprint behavior is needed with no sensitivity analysis — a single fingerprint suffices, not all 27

## Inputs

- MetFrag candidate structure lists (SMILES/SDF) for the NIST 555-spectra benchmark spectra
- Spectral library matches serving as seed nodes
- RDKit fingerprint type selection (27 available types; one designated as default baseline)

## Outputs

- Pairwise structural-similarity matrix among candidate structures per fingerprint type
- MetFusion-weighted graph combining spectral network and structural similarity
- Per-fingerprint top-10 classification metrics aggregated into fingerprint_benchmark_results.csv, including the default-fingerprint baseline and deviations of the other 26 fingerprints

## How to apply

Load the candidate structures provided by MetFrag in silico fragmentation for the NIST 555-spectra benchmark and, for each of the 27 RDKit fingerprint types, convert structures to fingerprint bit vectors and compute pairwise structural similarity (e.g. Tanimoto) among all candidates. Combine this structural-similarity matrix with the spectral network using the MetFusion scoring function to produce a weighted graph, per the ChemWalker design where the weight function departs from 'the combination of a spectral network and the structural similarity among candidate structures'. Feed the weighted graph into pyrwr random-walk propagation from seed nodes (spectral library matches) to rank candidates. To assess fingerprint sensitivity, keep the default RDKit fingerprint as baseline and record the deviation of each of the other 26 fingerprints in top-10 candidate classification, since the ChemWalker-vs-NAP benchmark could not be statistically differentiated on the NIST 555-spectra data.

## Related tools

- **RDKit** (Source of the 27 molecular fingerprint types used to compute pairwise structural similarity among MetFrag candidate structures)
- **ChemWalker** (Python package whose pipeline combines fingerprint-derived structural similarity with the spectral network via the MetFusion scoring function) — https://github.com/computational-chemical-biology/ChemWalker
- **MetFusion** (Scoring function used to weight the graph from structural similarity and spectral network edges)
- **pyrwr** (Random walk with restart engine that propagates probabilities from spectral-library-match seed nodes over the fingerprint-derived weighted graph) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Provides the in silico combinatorial fragmentation candidate structures whose pairwise similarities are computed) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
from rdkit import DataStructs, Chem; from rdkit.Chem import rdMolDescriptors; fps=[rdMolDescriptors.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(s),2) for s in smiles]; sims=[DataStructs.TanimotoSimilarity(fps[i],fps[j]) for i in range(len(fps)) for j in range(len(fps))]
```

## Evaluation signals

- Verify the pairwise similarity matrix is square with dimensions equal to the candidate-list size for each spectrum, with positive weights only (pyrwr requires positively weighted networks)
- Confirm the default RDKit fingerprint run reproduces the published ChemWalker top-10 classification equivalence with NAP on the NIST 555-spectra benchmark
- Check fingerprint_benchmark_results.csv contains 27 rows (default baseline + 26 fingerprint deviations) with top-10 classification metrics per fingerprint
- Confirm the weighted graph rows/columns align with MetFrag candidate IDs and that spectral-library-match seed nodes are present as graph nodes

## Limitations

- pyrwr RWR is defined on positively weighted networks, so negative or zero similarity-derived edge weights are invalid inputs
- The benchmark could not statistically differentiate ChemWalker from NAP in first-10-candidate classification on the NIST 555-spectra data, so fingerprint choice may not produce a detectable top-10 difference
- ChemWalker was tested on MetFrag2.3-CL.jar; other MetFrag versions or command-line jar releases may behave differently

## Evidence

- [abstract] weighted graph from similarity + spectral network: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function"
- [abstract] random walk from seed nodes: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [intro] pyrwr dependency for random walk: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]"
- [readme] ChemWalker tested on MetFrag2.3-CL.jar: "ChemWalker was tested on [MetFrag2.3.-CL.jar](http://ccbl.fcfrp.usp.br/ccbl/MetFrag2.3-CL.jar), download MetFrag CL [here]"
- [readme] pyrwr positive-weight constraint: [evidence span withheld — anchor status: external_doc]
