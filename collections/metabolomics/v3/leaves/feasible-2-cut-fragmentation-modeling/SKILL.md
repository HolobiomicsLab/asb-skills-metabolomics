---
name: feasible-2-cut-fragmentation-modeling
description: Use when constructing theoretical fragment masses from known chemical
  structures (e.g., PubChem-derived SDF) to annotate LC-MS/MS spectra during dereplication
  with DEREPLICATOR+/NPDtools.
when_to_use_negative:
- Structures outside the metabolite classes DEREPLICATOR+ covers — the article states
  DEREPLICATOR+ is not applicable to all classes of compounds
- Peptidic natural products where the earlier DEREPLICATOR amide-bond-cut model suffices
  (DEREPLICATOR-CN's all C–N cutting is a different, looser model)
- Already-acquired experimental spectra with no candidate structures — this skill
  generates theoretical fragments, not spectral clustering or molecular networking
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: DEREPLICATOR+
  role: Implements the metabolite-graph and feasible fragmentation-graph construction
    for dereplication of diverse (peptidic and non-peptidic) metabolites
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR-CN
  role: Contrasting variant in which all C–N bonds are cut rather than amide bonds
    only
- name: NPDtools
  role: Toolkit distributing the Dereplicator+ pipeline (dereplicator+.py executable)
    used to apply this fragmentation model
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:8cd7893872eb607222562db63483ef3f3ae8a754741a438e2ed1060f0e91889a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR+
  - DEREPLICATOR-CN
  - NPDtools
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# feasible-2-cut-fragmentation-modeling

## Summary

Models feasible in-silico fragmentation of metabolite structures in DEREPLICATOR+ by building a fragmentation graph from the metabolite graph (whose nodes are connected components after disconnecting bonds), then pruning to depth ≤ k, ≤ one 2-cut per root-to-node path restricted to feasible C–N/C–O bonds, and ≤ one C–C bond fragmentation per path.

## When to use

Use when constructing theoretical fragment masses from known chemical structures (e.g., PubChem-derived SDF) to annotate LC-MS/MS spectra during dereplication with DEREPLICATOR+/NPDtools.

## When NOT to use

- Structures outside the metabolite classes DEREPLICATOR+ covers — the article states DEREPLICATOR+ is not applicable to all classes of compounds
- Peptidic natural products where the earlier DEREPLICATOR amide-bond-cut model suffices (DEREPLICATOR-CN's all C–N cutting is a different, looser model)
- Already-acquired experimental spectra with no candidate structures — this skill generates theoretical fragments, not spectral clustering or molecular networking

## Inputs

- Chemical structure database entries (e.g., PubChem-derived SDF) for known natural products
- Bond-typed molecular graphs (C–N, C–O, C–C) of metabolites
- Depth parameter k for the fragmentation graph

## Outputs

- Pruned fragmentation graph per metabolite, rooted at the intact structure, with nodes labeled by elemental composition/fragment mass (serialized e.g. as fragmentation_graphs.pkl)
- Decoy fragmentation graph generated for each target fragmentation graph

## How to apply

First parse each input structure into a molecular graph with bonds typed as C–N, C–O, or C–C, then disconnect all metabolite bonds so each connected component becomes a node of the metabolite graph labeled by its elemental composition. Generate the fragmentation graph breadth-first from the intact root by enumerating 1-cut and 2-cut bond cleavages. Apply the DEREPLICATOR+ pruning constraints: remove all nodes with depth exceeding k from the root; retain only nodes reachable via a path containing at most one 2-cut, and limit 2-cuts to the feasible bond pairs C–N or C–O (e.g., bridges such as amide bonds); finally retain only nodes whose path from the root contains at most one C–C bond fragmentation. The rationale is that unconstrained combinatorial fragmentation explodes in size and generates chemically implausible fragments, so these feasibility constraints keep the graph small enough for efficient database search while preserving the fragments that dominate real MS/MS spectra. Serialize the pruned graph so its nodes provide the theoretical masses scored against processed spectra (six top-intensity peaks per 50 Da window).

## Related tools

- **DEREPLICATOR+** (Implements the metabolite-graph and feasible fragmentation-graph construction for dereplication of diverse (peptidic and non-peptidic) metabolites) — https://github.com/ablab/npdtools
- **DEREPLICATOR-CN** (Contrasting variant in which all C–N bonds are cut rather than amide bonds only)
- **NPDtools** (Toolkit distributing the Dereplicator+ pipeline (dereplicator+.py executable) used to apply this fragmentation model) — https://github.com/ablab/npdtools

## Evaluation signals

- Every serialized fragmentation graph node has depth from the root ≤ k
- Each root-to-node path contains at most one 2-cut, and every 2-cut is restricted to C–N or C–O bonds
- Each root-to-node path contains at most one C–C bond fragmentation
- Fragment node masses equal the measured masses of connected components after disconnecting the corresponding bonds in the metabolite graph
- Each target fragmentation graph has exactly one paired decoy fragmentation graph for FDR estimation

## Limitations

- Simple shared-peaks scoring against fragmentation graphs results in a high false-negative rate; the improved scoring scheme is not yet implemented
- P-value computation is only approximate for non-tree (cyclic) fragmentation graphs; unbiased estimation would require MCMC, which is not implemented
- DEREPLICATOR+ is not applicable to all classes of compounds
- Feasibility constraints (depth k, single 2-cut, single C–C cut) may exclude fragmentation events that occur experimentally

## Evidence

- [discussion] Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [discussion] 2-cut fragmentations are limited to feasible 2-cuts: "2-cut fragmentations are limited to feasible 2-cuts"
- [discussion] only nodes with a path from the root consisting of maximum one C–C bond fragmentation are retained in the graph.: "only nodes with a path from the root consisting of maximum one C–C bond fragmentation are retained in the graph."
- [discussion] all nodes with depths exceeding k are removed from the graph: "all nodes with depths exceeding k are removed from the graph"
- [readme] Dereplicator+ — a tool for identification of metabolites (both peptidic and non-peptidic) through database search of mass spectra: [evidence span withheld — anchor status: external_doc]
