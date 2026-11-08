---
name: elemental-composition-node-labeling
description: Use when constructing metabolite graphs from chemical structures (e.g.,
  a PubChem-derived SDF) as the first stage of the DEREPLICATOR+ fragmentation-graph
  pipeline before enumerating bond cleavages for spectral dereplication.
when_to_use_negative:
- Input is already an experimental mass spectrum or peak list — this skill operates
  on chemical structures, not spectra (spectra are handled later by node annotation
  and MSM scoring)
- The goal is spectral preprocessing rather than structure-based graph construction
  — for spectra, DEREPLICATOR+ instead retains six top intensity peaks per 50 Da window
tools:
- name: DEREPLICATOR+
  role: Implements the metabolite-graph construction in which disconnected connected
    components become elemental-composition-labeled nodes feeding the fragmentation
    graph
  repo: github.com/ablab/npdtools
- name: NPDtools
  role: Toolkit distribution providing the dereplicator+ executable (bin/dereplicator+.py)
    that runs the pipeline containing this graph-construction step
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:f2734e4dea1c12474369203759ad6079d64e14cb87e2362da20f5ec0c69033cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR+
  - NPDtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# elemental-composition-node-labeling

## Summary

Labels the nodes of a DEREPLICATOR+ metabolite graph: every metabolite bond (C–N, C–O, C–C) is disconnected and each resulting connected component becomes a node labeled by its elemental composition, providing the substrate from which the fragmentation graph is built.

## When to use

Use when constructing metabolite graphs from chemical structures (e.g., a PubChem-derived SDF) as the first stage of the DEREPLICATOR+ fragmentation-graph pipeline before enumerating bond cleavages for spectral dereplication.

## When NOT to use

- Input is already an experimental mass spectrum or peak list — this skill operates on chemical structures, not spectra (spectra are handled later by node annotation and MSM scoring)
- The goal is spectral preprocessing rather than structure-based graph construction — for spectra, DEREPLICATOR+ instead retains six top intensity peaks per 50 Da window

## Inputs

- Chemical structure database of known natural products (e.g., PubChem-derived SDF)
- Molecular graphs with bonds typed as C–N, C–O, or C–C

## Outputs

- Metabolite graph whose nodes are connected components labeled by elemental composition
- Root node representing the intact metabolite structure for breadth-first fragmentation-graph generation

## How to apply

Parse the input chemical structures into molecular graphs with bond types typed as C–N, C–O, or C–C. Disconnect all metabolite bonds so the structure breaks into connected components; each connected component becomes a node of the metabolite graph. Label each node by its elemental composition so that downstream fragment masses can be computed from node labels. This metabolite graph then serves as the intact root from which the fragmentation graph is generated breadth-first via 1-cut and 2-cut bond cleavages (2-cuts restricted to feasible C–N or C–O bonds). The rationale is that the masses of connected components form the theoretical fragments used to annotate spectra and score metabolite–spectrum matches.

## Related tools

- **DEREPLICATOR+** (Implements the metabolite-graph construction in which disconnected connected components become elemental-composition-labeled nodes feeding the fragmentation graph) — github.com/ablab/npdtools
- **NPDtools** (Toolkit distribution providing the dereplicator+ executable (bin/dereplicator+.py) that runs the pipeline containing this graph-construction step) — https://github.com/ablab/npdtools

## Evaluation signals

- Every node of the constructed metabolite graph carries a non-empty elemental-composition label
- Summing the elemental compositions over all disconnected components reconstructs the intact metabolite's elemental composition (mass-conservation check)
- Each node's elemental composition yields a monoisotopic mass matching the mass of the corresponding connected component used as a theoretical fragment
- Downstream fragmentation graphs built from this metabolite graph satisfy pruning constraints: node depth ≤ k, at most one 2-cut per root-to-node path restricted to C–N/C–O bonds, and at most one C–C bond fragmentation per path

## Limitations

- 2-cut fragmentations are only feasible for C–N or C–O bonds, so node generation from the labeled metabolite graph is restricted accordingly
- DEREPLICATOR+'s simple shared-peaks scoring against fragmentation-graph nodes results in a high false-negative rate, which propagates back to spectra matched via these labeled graphs
- DEREPLICATOR+ is not applicable to all classes of compounds, so the metabolite-graph representation is likewise limited in scope

## Evidence

- [discussion] Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [readme] Dereplicator+ — a tool for identification of metabolites (both peptidic and non-peptidic) through database search of mass spectra: [evidence span withheld — anchor status: not_found]
- [discussion] 2-cut fragmentations are limited to feasible 2-cuts: "2-cut fragmentations are limited to feasible 2-cuts"
