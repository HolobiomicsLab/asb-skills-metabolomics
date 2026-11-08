---
name: molecular-bond-disconnection-graph-construction
description: Use when converting a database of known metabolite structures (e.g.,
  PubChem-derived SDFs) into fragmentation graphs so that MS/MS spectra can be annotated
  by matching observed peaks to theoretical fragment masses.
when_to_use_negative:
- Input is already experimental MS/MS spectra with no chemical structure database
  — this skill constructs theoretical graphs from structures, not from data
- The metabolite class falls outside DEREPLICATOR+'s scope — the article states it
  is 'not applicable to all the classes of compounds'
- Only peptidic natural products (PNPs) are targeted — plain DEREPLICATOR handles
  PNPs via amide-bond cuts without the general fragmentation-graph machinery
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Implements the metabolite-graph and fragmentation-graph construction and pruning
    used to annotate spectra
  repo: https://github.com/ablab/npdtools
- name: NPDtools
  role: Toolkit distribution containing the Dereplicator+ executable and chemical
    structure database preprocessing
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:6629cb67c582c0e9bde436d67e29aee463bdd2d376d0ec62dfe2a9e07c7dd83e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR+
  - NPDtools
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# molecular-bond-disconnection-graph-construction

## Summary

Builds theoretical fragmentation graphs for natural products by disconnecting bonds (1-cuts and 2-cuts such as amide, C–N, C–O) in a metabolite structure, then pruning infeasible fragments, as implemented in DEREPLICATOR+ for spectral dereplication.

## When to use

Use when converting a database of known metabolite structures (e.g., PubChem-derived SDFs) into fragmentation graphs so that MS/MS spectra can be annotated by matching observed peaks to theoretical fragment masses.

## When NOT to use

- Input is already experimental MS/MS spectra with no chemical structure database — this skill constructs theoretical graphs from structures, not from data
- The metabolite class falls outside DEREPLICATOR+'s scope — the article states it is 'not applicable to all the classes of compounds'
- Only peptidic natural products (PNPs) are targeted — plain DEREPLICATOR handles PNPs via amide-bond cuts without the general fragmentation-graph machinery

## Inputs

- Chemical structure database of known natural products (e.g., PubChem-derived SDF)
- Molecular graph with bonds typed as C–N, C–O, or C–C
- Maximum fragment-graph depth parameter k

## Outputs

- Metabolite graph whose nodes are connected components labeled by elemental composition
- Pruned fragmentation graph rooted at the intact metabolite
- Serialized fragmentation graphs (fragmentation_graphs.pkl)

## How to apply

Parse each chemical structure into a molecular graph with bonds typed as C–N, C–O, or C–C. Construct the metabolite graph by disconnecting all metabolite bonds so each connected component becomes a node labeled by its elemental composition. Generate the fragmentation graph breadth-first from the intact root by enumerating 1-cut and 2-cut bond cleavages, then apply four pruning rules: remove all nodes with depth exceeding k from the root; retain only nodes reachable via a path containing at most one 2-cut fragmentation; restrict 2-cuts to feasible C–N or C–O bonds (e.g., amide-type disconnections); and retain only nodes whose root path contains at most one C–C bond fragmentation. The pruning rationale is to keep the combinatorics tractable while still covering chemically feasible fragments; serialize the pruned graph (e.g., to fragmentation_graphs.pkl) for downstream scoring against spectra.

## Related tools

- **DEREPLICATOR+** (Implements the metabolite-graph and fragmentation-graph construction and pruning used to annotate spectra) — https://github.com/ablab/npdtools
- **NPDtools** (Toolkit distribution containing the Dereplicator+ executable and chemical structure database preprocessing) — https://github.com/ablab/npdtools

## Evaluation signals

- Every serialized fragmentation graph node has depth ≤ k from the root
- No root-to-node path contains more than one 2-cut fragmentation, and all 2-cuts are restricted to C–N or C–O bonds
- No root-to-node path contains more than one C–C bond fragmentation
- Metabolite graph nodes correctly label connected components by elemental composition after full bond disconnection
- Fragment masses derived from connected components match the theoretical fragment masses used for spectral annotation

## Limitations

- P-value computation via recursion is only approximate for non-tree fragmentation graphs; unbiased MCMC estimation is described as possible but not implemented
- Simple shared-peaks scoring against fragmentation graphs yields a high false-negative rate
- The pruning rules (depth k, single 2-cut, single C–C cut) intentionally exclude some fragments, so fragments outside these constraints cannot be annotated

## Evidence

- [discussion] Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [discussion] DEREPLICATOR+ introduces the concept of the fragmentation graph: "To address this complication, DEREPLICATOR+ introduces the concept of the fragmentation graph."
- [discussion] all nodes with depths exceeding k are removed from the graph: "all nodes with depths exceeding k are removed from the graph"
- [discussion] only nodes with a path from the root consisting of maximum one 2-cut fragmentation are retained in the graph: "only nodes with a path from the root consisting of maximum one 2-cut fragmentation are retained in the graph"
- [discussion] 2-cut fragmentations are limited to feasible 2-cuts: "2-cut fragmentations are limited to feasible 2-cuts"
- [readme] Database search pipelines (Dereplicator, VarQuest and Dereplicator+) also require a chemical structure database of known natural products.: [evidence span withheld — anchor status: external_doc]
