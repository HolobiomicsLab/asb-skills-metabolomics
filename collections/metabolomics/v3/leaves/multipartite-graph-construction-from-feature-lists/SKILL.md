---
name: multipartite-graph-construction-from-feature-lists
description: Use when aligning features across multiple LC-MS runs from CSV feature
  tables (m/z, RT, area columns) where nearest-neighbor matching is insufficient and
  all potential inter-run correspondences must be jointly considered in a graph structure.
when_to_use_negative:
- The input is raw LC-MS data (mzML) rather than already-extracted feature lists —
  feature detection/quantification must be done first by tools like MZmine2, OpenMS
  or XCMS
- A simple reference-run pairwise nearest matching suffices and computational speed
  matters more than accuracy — G-Aligner's main limitation is lack of computing speed
  since the assignment problem is NP-Hard
- Only two runs need pairwise alignment via bipartite matching (e.g. LWBmatch-style);
  the multipartite graph targets non-centric matching across all runs
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Implements the multipartite graph construction, connected-component splitting,
    edge weighting and assignment-problem formulation for feature alignment
  repo: https://github.com/CSi-Studio/G-Aligner
- name: OBI-Warp
  role: Used with RANSAC to perform stable coarse RT registration between runs prior
    to neighborhood search and graph building
- name: MZmine2
  role: One of the feature extraction tools whose CSV feature lists (m/z, RT, area)
    serve as input nodes for the graph
- name: OpenMS
  role: One of the feature extraction tools whose CSV feature lists serve as graph
    input; its alignment output is a comparison baseline
- name: XCMS
  role: One of the feature extraction tools whose CSV feature lists serve as graph
    input; its alignment output is a comparison baseline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:329e0feee02a85c3ec6a23f26ea6668ca545859540bf97a36b2531552fa4f622
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OBI-Warp
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Multipartite graph construction from feature lists

## Summary

Builds an undirected multipartite relation graph in which features from all LC-MS runs are nodes and potential feature correspondences (within user-specified m/z and RT tolerances after coarse RT registration) are edges, enabling non-centric multi-run feature matching. This graph representation is the core data structure G-Aligner uses before solving alignment as an unbalanced multidimensional assignment problem.

## When to use

Use when aligning features across multiple LC-MS runs from CSV feature tables (m/z, RT, area columns) where nearest-neighbor matching is insufficient and all potential inter-run correspondences must be jointly considered in a graph structure.

## When NOT to use

- The input is raw LC-MS data (mzML) rather than already-extracted feature lists — feature detection/quantification must be done first by tools like MZmine2, OpenMS or XCMS
- A simple reference-run pairwise nearest matching suffices and computational speed matters more than accuracy — G-Aligner's main limitation is lack of computing speed since the assignment problem is NP-Hard
- Only two runs need pairwise alignment via bipartite matching (e.g. LWBmatch-style); the multipartite graph targets non-centric matching across all runs

## Inputs

- CSV feature tables (m/z, RT, area columns) extracted per LC-MS run by MZmine2, OpenMS, XCMS, or MetaPro, or manually annotated feature lists for MTBLS562, QE_HF and TripleTOF_6600 benchmark datasets
- User-specified m/z and retention time tolerance parameters (optimized per dataset, reported in supplementary Tables S1-S3)

## Outputs

- Undirected multipartite relation graph whose nodes are features of all runs and whose edges are potential correspondences
- Connected components of the graph, each carrying edge weights (Eqs 1-2) feeding the multidimensional assignment problem

## How to apply

After coarse retention time registration between runs with RANSAC and OBI-Warp, search neighborhoods for each feature using user-specified m/z and RT tolerances across all runs. Build an undirected multipartite relation graph in which features are nodes and potential correspondences are edges — G-Aligner is the first method enabling non-centric comprehensive analysis of all potential feature correspondences among multiple LC-MS runs. Split the graph into connected components to decompose the global matching problem into subproblems, then compute edge weights (per Eqs 1–2 of the paper) so the multi-run matching can be formulated as an unbalanced multidimensional assignment problem solved by combinatorial optimization (Eqs 3–15). Input CSV feature lists must contain m/z, RT and area columns, as produced by MZmine2, OpenMS, XCMS, or manual targeted annotation.

## Related tools

- **G-Aligner** (Implements the multipartite graph construction, connected-component splitting, edge weighting and assignment-problem formulation for feature alignment) — https://github.com/CSi-Studio/G-Aligner
- **OBI-Warp** (Used with RANSAC to perform stable coarse RT registration between runs prior to neighborhood search and graph building)
- **MZmine2** (One of the feature extraction tools whose CSV feature lists (m/z, RT, area) serve as input nodes for the graph)
- **OpenMS** (One of the feature extraction tools whose CSV feature lists serve as graph input; its alignment output is a comparison baseline)
- **XCMS** (One of the feature extraction tools whose CSV feature lists serve as graph input; its alignment output is a comparison baseline)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py  # results saved in experiment/ folder
```

## Evaluation signals

- The resulting consensus feature groups contain no ambiguous matched groups, consistent with G-Aligner achieving the highest alignment accuracy without ambiguous groups
- The pipeline runs end-to-end on all three benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600) via data/evaluate_metapro_galigner.py
- Accurately aligned features/analytes match or exceed reported gains (up to 9.8% and 26.6% increase over MZmine2, OpenMS and XCMS baselines)
- Every node in a connected component belongs to a feature from exactly one run per partite set, and edges only connect features within the user-specified m/z and RT tolerances after RANSAC/OBI-Warp registration
- Feature alignment results are written to the 'experiment' folder in the expected output format

## Limitations

- The main limitation of G-Aligner is the lack of computing speed — the multidimensional assignment problem is NP-Hard and inevitably requires a lot of computing time
- Two acceleration methods keep running time within an acceptable range, but there is still a gap to painless application; GPU acceleration is not yet implemented
- Optimized m/z and RT tolerance parameters are dataset-specific and reported only in supplementary Tables S1-S3, so they must be tuned per dataset

## Evidence

- [intro] graph building from neighborhood search: "G-Aligner is the first method enabling non-centric comprehensive analysis of all potential feature correspondences among multiple LC–MS runs."
- [abstract] graph representation: "G-Aligner treats features and potential correspondences as nodes and edges in a multipartite graph"
- [intro] assignment formulation: "G-Aligner formats the potential correspondences between all features as a multipartite graph, and solves the multi-run feature matching problem as an unbalanced multidimensional assignment problem"
- [intro] RT registration prerequisite: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [readme] input format requirement: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [abstract] performance benchmark: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
