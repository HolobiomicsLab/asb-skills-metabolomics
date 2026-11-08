---
name: molecular-graph-depth-first-traversal
description: Use when scoring query mass spectra against a candidate molecule requires
  decomposing its molecular graph into bonded fragments, and the fragmentation graph
  should only be expanded as far as needed to explain observed peaks.
when_to_use_negative:
- The query spectrum has no explained parent fragment — adaptive expansion never proceeds
  past depth 1, so deeper fragmentation is unnecessary
- The molecule is not a valid bonded structure graph (e.g. fragment ion spectra of
  peptides in Peptide mode may not benefit from generic metabolite graph traversal)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0602
tools:
- name: VInSMoC
  role: Implements the metabolite graph depth-first traversal and adaptive fragmentation
    graph expansion to score query spectra against candidate molecules
  repo: mohimanilab/VInSMoC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:47017d4f370ee68718a5ccbbab99e61929b33207c8c3efc87f9d893b686e7aeb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-graph-depth-first-traversal

## Summary

Single-pass depth-first traversal over a candidate metabolite's molecular graph to construct depth-1 fragmentation graph nodes with Hopcroft–Tarjan cut-vertex finding, in service of building VInSMoC's fragmentation graph on demand.

## When to use

Use when scoring query mass spectra against a candidate molecule requires decomposing its molecular graph into bonded fragments, and the fragmentation graph should only be expanded as far as needed to explain observed peaks.

## When NOT to use

- The query spectrum has no explained parent fragment — adaptive expansion never proceeds past depth 1, so deeper fragmentation is unnecessary
- The molecule is not a valid bonded structure graph (e.g. fragment ion spectra of peptides in Peptide mode may not benefit from generic metabolite graph traversal)

## Inputs

- Candidate molecule structure from a chemical database (e.g. NPAtlas) as a molecular graph
- Query mass spectrum peaks (from MGF, mzXML, or mzML) with precursor m/z
- Adduct/ion-mode configuration (e.g. maximum absolute charge 1, positive or negative ion mode)
- Product ion tolerance for matching theoretical fragments to experimental peaks

## Outputs

- Depth-1 fragmentation graph nodes over the candidate molecule
- Adaptively expanded higher-depth fragmentation graph fragments (cached to avoid recomputation)
- Theoretical fragments for matching against query peaks

## How to apply

For each candidate molecule in the chemical database (e.g. NPAtlas), represent its bonds as a metabolite graph and traverse it with a single-pass depth-first search. During traversal, use Hopcroft–Tarjan cut-vertex finding to identify articulation points and build depth-1 fragmentation graph nodes. Expansion beyond depth 1 is adaptive: a fragment at depth greater than 1 is computed only when its parent fragment has been explained by a query peak (converted from m/z to neutral mass using [M+H]+ or [M-H]- adducts depending on ion mode), and computed higher-depth fragments are cached to avoid duplicate computation. This limits fragmentation-graph work to exactly what is needed to score the query spectrum against that molecule.

## Related tools

- **VInSMoC** (Implements the metabolite graph depth-first traversal and adaptive fragmentation graph expansion to score query spectra against candidate molecules) — mohimanilab/VInSMoC

## Examples

```
Select the VInSMoC method on run.npanalysis.org with your mzML spectrum file and NPAtlas database, set Search mode and Fragmentation mode (Metabolite or Peptide), then submit.
```

## Evaluation signals

- Depth-1 fragmentation graph nodes exist for every candidate molecule processed (Hopcroft–Tarjan cut-vertex finding succeeded)
- No fragment at depth greater than 1 is computed for a parent fragment that was not explained by any query peak
- Repeated queries against the same molecule reuse cached higher-depth fragments rather than recomputing them
- Peak-count score for each candidate falls within the expected range for the matched fragment set, with matches within the product ion tolerance

## Limitations

- Higher-depth fragments are only computed when needed — the traversal does not exhaustively enumerate all fragmentation possibilities upfront
- Peak merging and peak filtering (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) alter the query spectrum before matching, which affects which parent fragments get explained and thus how far fragmentation expands

## Evidence

- [other] Single-pass DFS builds depth-1 fragmentation nodes via cut vertices: [evidence span withheld — anchor status: not_found]
- [methods] Adaptive fragmentation: compute fragment only when parent is explained: [evidence span withheld — anchor status: not_found]
