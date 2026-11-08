---
name: fragmentation-graph-depth-pruning
description: Use when building theoretical fragment sets from chemical structures
  (e.g. PubChem-derived SDFs) to annotate MS/MS spectra, and the fragmentation graph
  must be constrained to feasible fragmentations.
when_to_use_negative:
- Input is raw LC-MS/MS spectra needing preprocessing (retaining six top intensity
  peaks per 50 Da window) or scoring/FDR — pruning applies only to structure-side
  fragmentation graphs.
- Goal is decoy generation or molecular-network variant propagation — these consume
  the pruned fragmentation graphs but are separate steps.
- The structure database has already been preprocessed into fragmentation graphs (fragmentation_graphs.pkl
  exists and passed QC) — rerunning pruning would be redundant.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Constructs the metabolite and fragmentation graphs and applies depth-k and
    1-cut/2-cut pruning constraints
  repo: github.com/ablab/npdtools
- name: NPDtools
  role: Toolkit packaging the dereplicator+.py executable and database preprocessing
    scripts used to run the fragmentation-graph pipeline
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:9fcab8592a5481e9dc3f1a69ce95cf9b82f77c6dbec0dda2c94fa2fe73e918ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR+
  - NPDtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fragmentation-graph-depth-pruning

## Summary

Constructs and prunes DEREPLICATOR+ fragmentation graphs: metabolite bonds (C–N, C–O, C–C) are disconnected breadth-first from the intact root structure, and nodes violating depth-k or 1-cut/2-cut path constraints are removed before scoring mass spectra.

## When to use

Use when building theoretical fragment sets from chemical structures (e.g. PubChem-derived SDFs) to annotate MS/MS spectra, and the fragmentation graph must be constrained to feasible fragmentations.

## When NOT to use

- Input is raw LC-MS/MS spectra needing preprocessing (retaining six top intensity peaks per 50 Da window) or scoring/FDR — pruning applies only to structure-side fragmentation graphs.
- Goal is decoy generation or molecular-network variant propagation — these consume the pruned fragmentation graphs but are separate steps.
- The structure database has already been preprocessed into fragmentation graphs (fragmentation_graphs.pkl exists and passed QC) — rerunning pruning would be redundant.

## Inputs

- Chemical structure database in SDF format (e.g. PubChem-derived) of known natural products
- Typed bond connectivity (C–N, C–O, C–C) per metabolite
- Depth cutoff parameter k

## Outputs

- Pruned fragmentation graphs with nodes labeled by elemental composition
- Serialized graph set fragmentation_graphs.pkl

## How to apply

Parse the input chemical structures into molecular graphs with typed bonds (C–N, C–O, C–C), then disconnect all metabolite bonds so each connected component becomes a node labeled by its elemental composition. Enumerate 1-cut and 2-cut bond cleavages breadth-first from the intact root structure to build the fragmentation graph, then apply three pruning filters: remove all nodes whose depth exceeds k from the root; retain only nodes with a root path containing at most one 2-cut fragmentation, with 2-cuts restricted to feasible C–N or C–O bonds; retain only nodes with a path containing at most one C–C bond fragmentation. The rationale is that unconstrained disconnection explodes the candidate fragment space and generates chemically implausible fragments (e.g. amide/2-cut cleavages everywhere), so path-history constraints keep theoretical fragments close to those observable in tandem MS. Serialize the pruned graphs (fragmentation_graphs.pkl) and validate each against the pruning constraints as a qualitative QC step before using them to score spectra by unique matched masses.

## Related tools

- **DEREPLICATOR+** (Constructs the metabolite and fragmentation graphs and applies depth-k and 1-cut/2-cut pruning constraints) — github.com/ablab/npdtools
- **NPDtools** (Toolkit packaging the dereplicator+.py executable and database preprocessing scripts used to run the fragmentation-graph pipeline) — https://github.com/ablab/npdtools

## Examples

```
import pickle; graphs = pickle.load(open('fragmentation_graphs.pkl','rb')); assert all(n.depth <= k for g in graphs for n in g.nodes)
```

## Evaluation signals

- Every serialized fragmentation graph contains no node whose depth from the intact root exceeds k
- Every root-to-node path contains at most one 2-cut fragmentation, and all 2-cuts are restricted to C–N or C–O bonds
- Every root-to-node path contains at most one C–C bond fragmentation
- Each graph's root node matches the intact metabolite's elemental composition, and all nodes carry elemental-composition labels
- fragmentation_graphs.pkl loads cleanly and contains one pruned graph per database structure entry

## Limitations

- The k cutoff and 1-cut/2-cut path constraints discard fragments beyond local cleavages, potentially missing multi-bond rearrangement fragments observable in real spectra.
- The pruning-QC step is qualitative; downstream scoring's simple shared-peaks scheme already produces a high false-negative rate, which pruning constraints cannot compensate for.
- DEREPLICATOR+ is not applicable to all classes of compounds, so graphs built for some metabolite classes may never match real spectra.

## Evidence

- [discussion] all nodes with depths exceeding k are removed from the graph: "all nodes with depths exceeding k are removed from the graph"
- [discussion] only nodes with a path from the root consisting of maximum one 2-cut fragmentation are retained in the graph: "only nodes with a path from the root consisting of maximum one 2-cut fragmentation are retained in the graph"
- [discussion] Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [discussion] DEREPLICATOR+ introduces the concept of the fragmentation graph.: "To address this complication, DEREPLICATOR+ introduces the concept of the fragmentation graph."
- [readme] DEREPLICATOR+ — a tool for identification of metabolites (both peptidic and non-peptidic) through database search of mass spectra: [evidence span withheld — anchor status: not_found]
