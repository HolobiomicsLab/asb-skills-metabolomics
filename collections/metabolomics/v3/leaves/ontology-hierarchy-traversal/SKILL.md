---
name: ontology-hierarchy-traversal
description: Use when a biological question names a broad ontology category (e.g.
  'Digestive Tract', 'rodents', 'Bacteria') and must be expanded into the concrete
  organ/species/disease metadata combinations it covers before statistical comparison.
when_to_use_negative:
- The query is already a simple pairwise metadata contrast between two specific combinations,
  with no need for ontological broadening.
- Candidate expansion terms would cross ontology branches (e.g. merging stool, which
  maps to 'fluids and secretions', into a digestive-system expansion) — the expansion
  must not pool across branches.
- The sample metadata cannot be mapped to an established ontology (e.g. cryptic domain-specific
  acronyms), so no 'belongs to' relations can be built.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0089
tools:
- name: NetworkX
  role: Builds and traverses the 'belongs to' ontology graph so broad terms resolve
    to all ontologically related nodes
- name: Python
  role: Custom scripts implementing the ontology loading, expansion, and cross-branch
    exclusion logic
- name: SciPy
  role: Statistics routines applied to the expanded triads (Welch t-tests on log-transformed
    metabolite intensities) after traversal
- name: PostgreSQL
  role: Stores the BinDiscover dataset of averages, medians, and ratios of intensity
    values queried during expansion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:713bb71a62f5ef45db934f3a4accd68dabc1a7c5a8ce518553a62e0334bb8430
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinDiscover
  - NetworkX
  - PostgreSQL
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ontology-hierarchy-traversal

## Summary

Traverses an ontology hierarchy (MeSH for organs/diseases, NCBI taxonomy for species) with a NetworkX 'belongs to' graph so that broad biological query terms like 'Digestive Tract' or 'Bacteria' resolve into their constituent metadata combinations for downstream differential analysis. This is the expansion step that lets BinDiscover's OGDA compare broad biological categories rather than single metadata pairs.

## When to use

Use when a biological question names a broad ontology category (e.g. 'Digestive Tract', 'rodents', 'Bacteria') and must be expanded into the concrete organ/species/disease metadata combinations it covers before statistical comparison.

## When NOT to use

- The query is already a simple pairwise metadata contrast between two specific combinations, with no need for ontological broadening.
- Candidate expansion terms would cross ontology branches (e.g. merging stool, which maps to 'fluids and secretions', into a digestive-system expansion) — the expansion must not pool across branches.
- The sample metadata cannot be mapped to an established ontology (e.g. cryptic domain-specific acronyms), so no 'belongs to' relations can be built.

## Inputs

- Ontology hierarchy (MeSH terms for organs/diseases or NCBI taxonomy for species)
- Curated sample metadata mapping each sample to organ/species/disease combinations (after misspelling correction and mapping to established ontologies)
- Broad query term (e.g. 'Digestive Tract', 'Bacteria', 'rodents')

## Outputs

- NetworkX 'belongs to' graph over ontology nodes
- Expanded set of constituent metadata combinations (organ/species/disease triads) per broad query term, with cross-branch terms excluded
- Per-triad sample groups passing QC thresholds, ready for downstream Welch t-test comparison

## How to apply

Load the relevant ontology hierarchy — MeSH for organ and disease terms or NCBI taxonomy for species — and build a 'belongs to' graph with NetworkX so every node points to its parent category. Given a broad query term, traverse the graph to enumerate all ontologically related descendant nodes, and convert them into constituent metadata combinations (organ/species/disease triads). Exclude candidate terms that belong to a different ontology branch even if semantically adjacent — for example, stool maps to 'fluids and secretions', not to the digestive system branch. Before handing triads to statistical comparison, filter out triads with more than 20% failed FAME samples and triads with fewer than 10 samples total. The rationale, per the paper, is that assigning metadata into ontologies empowers broad comparisons such as phylogenetic organizations and ontologically grouped differential analysis (OGDA), where each expanded category becomes one of the smaller comparisons whose statistics are later aggregated.

## Related tools

- **NetworkX** (Builds and traverses the 'belongs to' ontology graph so broad terms resolve to all ontologically related nodes)
- **Python** (Custom scripts implementing the ontology loading, expansion, and cross-branch exclusion logic)
- **SciPy** (Statistics routines applied to the expanded triads (Welch t-tests on log-transformed metabolite intensities) after traversal)
- **PostgreSQL** (Stores the BinDiscover dataset of averages, medians, and ratios of intensity values queried during expansion)

## Examples

```
import networkx as nx; G = nx.DiGraph(ontology_belongs_to_edges); expanded = nx.descendants(G, 'Digestive Tract') | {'Digestive Tract'}
```

## Evaluation signals

- A broad term such as 'Digestive Tract' or 'Bacteria' resolves in the belongs-to graph to all of its ontologically related descendant nodes.
- Terms from different ontology branches are excluded from the expansion (stool resolves to 'fluids and secretions', not to digestive-system descendants).
- Expansion output is a set of organ/species/disease triads that can be spot-checked against the MeSH or NCBI taxonomy hierarchy.
- Each expanded triad is filtered against the QC rules: dropped if >20% failed FAME samples or fewer than 10 total samples.
- On a synthetic metadata-annotated fixture, ogda_results.csv contains one aggregated p-value and one fold-change per compound with max-p/min-fold-change aggregation applied over all n*m expanded pairs.

## Limitations

- For biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses.
- BinDiscover is built on a static December 2021 snapshot of BinBase, so metadata ontologies may not cover compounds or terms added later.
- Cross-ontology-group comparisons may mix incomparable units of biomass — e.g. comparisons across organs can include biofluids versus tissues.

## Evidence

- [intro] OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs.: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [abstract] we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies.: "we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies."
- [methods] We heavily employed statistics routines from SciPy and the network analysis framework from NetworkX: "We heavily employed statistics routines from SciPy and the network analysis framework from NetworkX"
- [methods] we removed entire biological metadata triads if they showed more than 20% failed FAME samples: "we removed entire biological metadata triads if they showed more than 20% failed FAME samples"
