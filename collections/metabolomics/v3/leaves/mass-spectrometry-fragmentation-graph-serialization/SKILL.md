---
name: mass-spectrometry-fragmentation-graph-serialization
description: Use when building theoretical fragment sets from chemical structure databases
  (e.g., PubChem-derived SDF/MGF inputs) for DEREPLICATOR+ spectral annotation of
  LC-MS/MS natural product data.
when_to_use_negative:
- Input is already a list of observed spectrum peaks needing preprocessing (use the
  six-top-peaks-per-50 Da window filter instead) rather than theoretical fragment
  generation
- The target compounds are peptidic natural products where the original DEREPLICATOR
  amide-only cleavage pipeline suffices
- Compound classes outside DEREPLICATOR+'s scope where the tool 'is not applicable
  to all the classes of compounds'
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DEREPLICATOR+
  role: Constructs metabolite graphs and generates/prunes fragmentation graphs used
    to annotate mass spectra during dereplication
  repo: github.com/ablab/npdtools
- name: NPDtools
  role: Toolkit distributing the dereplicator+.py executable that implements the fragmentation
    graph pipeline
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:ddabb8b5aa11d69ef49dc1de256549b60fdd565b825fab664c5799685f44c4e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR+
  - NPDtools
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectrometry-fragmentation-graph-serialization

## Summary

Constructs DEREPLICATOR+ fragmentation graphs by breadth-first enumeration of 1-cut and 2-cut bond cleavages from a metabolite graph, applies depth and fragmentation-count pruning rules, and serializes the pruned graphs for spectral dereplication.

## When to use

Use when building theoretical fragment sets from chemical structure databases (e.g., PubChem-derived SDF/MGF inputs) for DEREPLICATOR+ spectral annotation of LC-MS/MS natural product data.

## When NOT to use

- Input is already a list of observed spectrum peaks needing preprocessing (use the six-top-peaks-per-50 Da window filter instead) rather than theoretical fragment generation
- The target compounds are peptidic natural products where the original DEREPLICATOR amide-only cleavage pipeline suffices
- Compound classes outside DEREPLICATOR+'s scope where the tool 'is not applicable to all the classes of compounds'

## Inputs

- Chemical structure database of known natural products (e.g., PubChem-derived SDF)
- Metabolite molecular graphs with bonds typed as C–N, C–O, or C–C
- Pruning parameter k (maximum node depth from root)

## Outputs

- Pruned fragmentation graphs per metabolite, serialized (e.g., fragmentation_graphs.pkl)
- Fragment graph nodes labeled by elemental composition / theoretical fragment masses

## How to apply

Parse input chemical structures into molecular graphs with bonds typed as C–N, C–O, or C–C, then build the metabolite graph by disconnecting all bonds so each connected component becomes a node labeled by elemental composition. Generate the fragmentation graph breadth-first from the intact root structure by enumerating 1-cut and 2-cut bond cleavages (bridges and 2-cuts such as amide bonds), then prune it: remove all nodes whose depth from the root exceeds k, retain only nodes with at most one 2-cut per root-to-node path restricted to feasible C–N or C–O bonds, and retain only nodes with at most one C–C bond fragmentation per path. Serialize the pruned graphs (e.g., to fragmentation_graphs.pkl) so that scoring against spectra — defined as the number of unique masses in a spectrum annotating a fragmentation graph node — can proceed. The rationale is that exhaustive fragmentation over chemical databases like PubChem (≈83 million compounds) is infeasible, so these pruning rules keep only chemically plausible fragments while bounding graph size.

## Related tools

- **DEREPLICATOR+** (Constructs metabolite graphs and generates/prunes fragmentation graphs used to annotate mass spectra during dereplication) — github.com/ablab/npdtools
- **NPDtools** (Toolkit distributing the dereplicator+.py executable that implements the fragmentation graph pipeline) — https://github.com/ablab/npdtools

## Examples

```
dereplicator+.py --spectra spectra.mgf --database structures.sdf --output fragmentation_graphs.pkl
```

## Evaluation signals

- Every serialized fragmentation graph satisfies node depth ≤ k from the root
- No root-to-node path contains more than one 2-cut fragmentation, and all 2-cuts are restricted to C–N or C–O bonds
- No root-to-node path contains more than one C–C bond fragmentation
- Each fragment node's mass corresponds to an actual connected component's elemental composition obtained by disconnecting metabolite bonds (bridges and 2-cuts such as amide bonds)
- Serialized output (fragmentation_graphs.pkl) loads without error and contains one fragmentation graph per input structure

## Limitations

- Simple shared-peaks scoring against fragmentation graphs yields a high false-negative rate; improved scoring was not implemented
- P-value recursion is only approximate for non-tree fragmentation graphs; unbiased MCMC probability estimation was described as possible but not implemented
- DEREPLICATOR+ is not applicable to all classes of compounds

## Evidence

- [discussion] Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [discussion] depth pruning: "all nodes with depths exceeding k are removed from the graph"
- [discussion] 2-cut restriction: "only nodes with a path from the root consisting of maximum one 2-cut fragmentation are retained in the graph"
- [discussion] 2-cut feasibility: "2-cut fragmentations are limited to feasible 2-cuts"
- [discussion] C-C restriction: "only nodes with a path from the root consisting of maximum one C–C bond fragmentation are retained in the graph."
- [discussion] scoring definition: "we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M"
