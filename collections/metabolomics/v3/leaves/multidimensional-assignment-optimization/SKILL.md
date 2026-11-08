---
name: multidimensional-assignment-optimization
description: Use when aligning untargeted LC-MS feature lists (m/z, RT, area CSVs)
  across multiple runs with nonlinear RT drift, where nearest-neighbor or bipartite
  matching yields ambiguous or inaccurate consensus features and a global non-centric
  optimum is needed.
when_to_use_negative:
- When the input is already a consensus feature map or when pairwise correspondence
  is sufficient — G-Aligner's graph construction and MAP solving are designed for
  multi-run, non-centric matching where local pairwise methods underperform.
- 'When computing speed is the binding constraint: the multidimensional assignment
  problem is NP-Hard and G-Aligner''s main limitation is ''lack of computing speed'';
  simple nearest-neighbour or greedy-only pipelines may be preferable for very large
  datasets.'
- When the feature extraction step itself is the bottleneck — this skill addresses
  only the feature alignment (matching) stage, not raw peak detection or quantification.
edam_operation: http://edamontology.org/operation_3809
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Implements the multipartite graph construction, unbalanced MAP formulation
    and three combinatorial optimization solvers used for multi-run feature matching.
  repo: https://github.com/CSi-Studio/G-Aligner
- name: RANSAC
  role: Used together with OBI-Warp for coarse retention-time registration between
    runs prior to graph building.
- name: OBI-Warp
  role: Used together with RANSAC for coarse retention-time registration between runs
    prior to graph building.
- name: Gurobi
  role: One of the combinatorial optimization solvers applied to the MAP in the TripleTOF
    6600 evaluation.
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:6bce59648251d57df7f81190edac598972d0737f5bd9358ec7e1b6d53ddb8cd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - Gurobi
  - MZmine2
  - OBI-Warp
  - RANSAC
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3809
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Multidimensional-assignment-optimization

## Summary

Formulates multi-run LC-MS feature matching as an unbalanced multidimensional assignment problem (MAP) over a multipartite graph of features and potential correspondences, solved with three combinatorial optimization algorithms (Gurobi, Greedy, VLSNS_MSR, VLSNS_MSG variants as surfaced in G-Aligner) to produce optimal consensus feature groups without ambiguous matched groups.

## When to use

Use when aligning untargeted LC-MS feature lists (m/z, RT, area CSVs) across multiple runs with nonlinear RT drift, where nearest-neighbor or bipartite matching yields ambiguous or inaccurate consensus features and a global non-centric optimum is needed.

## When NOT to use

- When the input is already a consensus feature map or when pairwise correspondence is sufficient — G-Aligner's graph construction and MAP solving are designed for multi-run, non-centric matching where local pairwise methods underperform.
- When computing speed is the binding constraint: the multidimensional assignment problem is NP-Hard and G-Aligner's main limitation is 'lack of computing speed'; simple nearest-neighbour or greedy-only pipelines may be preferable for very large datasets.
- When the feature extraction step itself is the bottleneck — this skill addresses only the feature alignment (matching) stage, not raw peak detection or quantification.

## Inputs

- CSV feature extraction results containing m/z, RT and area columns for each LC-MS run (from MZmine2, OpenMS, XCMS or G-Aligner's own extraction)
- Manually annotated feature lists (m/z, RT, area of library analytes) for evaluation, downloaded from Zenodo deposit 10.5281/zenodo.8313034
- mzML raw LC-MS files for the benchmark datasets (TripleTOF 6600, QE_HF, MTBLS562)

## Outputs

- Alignment results saved in the experiment folder: consensus feature groups without ambiguous matched groups, with per-feature run assignments across all runs
- Computed evaluation metrics: TP/FP/TN/FN counts, precision, recall, F1, F_ACC (percentage of accurately aligned features) and A_ACC (percentage of analytes with all replicates accurately aligned)
- Computational time per solver run for the MTBLS562/benchmark dataset evaluation

## How to apply

First perform coarse retention-time registration between runs with RANSAC and OBI-Warp, then search neighborhoods for each feature under user-specified m/z and RT tolerances to build an undirected multipartite relation graph in which features are nodes and potential correspondences are edges. Split the graph into connected components, compute edge weights per Eqs 1–2 of G-Aligner, and formulate multi-run matching as an unbalanced multidimensional assignment problem with costs from Eqs 3–15. Solve the MAP with three combinatorial optimization algorithms (Gurobi, Greedy, VLSNS_MSR/VLSNS_MSG per the source cards), writing results to the experiment folder as canonical alignment output. Use the optimized parameters from supplementary Tables S1–S3 for the benchmark dataset being evaluated (TripleTOF 6600, QE_HF or MTBLS562). Validate by comparing precision, recall, F1, F_ACC and A_ACC against manually annotated reference feature lists.

## Related tools

- **G-Aligner** (Implements the multipartite graph construction, unbalanced MAP formulation and three combinatorial optimization solvers used for multi-run feature matching.) — https://github.com/CSi-Studio/G-Aligner
- **RANSAC** (Used together with OBI-Warp for coarse retention-time registration between runs prior to graph building.)
- **OBI-Warp** (Used together with RANSAC for coarse retention-time registration between runs prior to graph building.)
- **Gurobi** (One of the combinatorial optimization solvers applied to the MAP in the TripleTOF 6600 evaluation.)

## Examples

```
python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- The alignment output must contain no ambiguous matched groups — G-Aligner's defining accuracy property on all three benchmark datasets.
- Compare computed precision, recall, F1, F_ACC and A_ACC against the values reported in Table 1 (TripleTOF 6600) and Table 3 (MTBLS562): e.g. MTBLS562 F1 = 1.000, F_ACC = 0.999, A_ACC = 0.981.
- Verify that G-Aligner achieves the highest F_ACC and A_ACC among MZmine2, OpenMS and XCMS, with reported gains up to 9.8% in accurately aligned features and 26.6% in accurately aligned analytes.
- When replacing a native alignment step (e.g. MZmine2 or XCMS), the F_ACC/A_ACC gains should match Tables 4–6: MZmine2 +1.7%/+5.8% (TripleTOF 6600), +9.0%/+26.6% (QE HF), +0.5%/+7.7% (MTBLS562); XCMS +9.8%/+24.5%, +2.6%/+11.4%, +9.4%/+24.7%.
- All four solver variants (Gurobi, Greedy, VLSNS_MSR, VLSNS_MSG) must run end-to-end on the TripleTOF 6600, QE_HF and MTBLS562 feature lists without errors and write results into the experiment folder.

## Limitations

- The main limitation of G-Aligner is the lack of computing speed: the multidimensional assignment problem is NP-Hard and inevitably requires a lot of computing time.
- Although two acceleration methods keep running time within an acceptable range, there is still a gap to painless application; GPU acceleration is not yet implemented.
- Optimized parameters are reported only in supplementary Tables S1–S3 and must be selected per dataset, not from the main text.
- Requires a user-specified m/z and retention-time tolerance; results depend on choosing the dataset-optimized parameter sets from the supplementary tables.

## Evidence

- [intro] G-Aligner formats the potential correspondences between all features as a multipartite graph, and solves the multi-run feature matching problem as an unbalanced multidimensional assignment problem: "G-Aligner formats the potential correspondences between all features as a multipartite graph, and solves the multi-run feature matching problem as an unbalanced multidimensional assignment problem"
- [methods] After coarse RT registration, G-Aligner searches neighborhoods for each feature according to user-specified m/z and retention time tolerances and builds an undirected relation graph: [evidence span withheld — anchor status: not_found]
- [methods] Feature alignment results are saved in experiment folder.: "Feature alignment results are saved in ```experiment``` folder."
- [readme] G-Aligner achieved the best performance in comparison with popular feature alignment methods in MZmine2, OpenMS and XCMS on two public metabolomics benchmark datasets.: "G-Aligner achieved the best performance in comparison with popular feature alignment methods in MZmine2, OpenMS and XCMS on two public metabolomics benchmark datasets."
- [discussion] The main limitation of G-Aligner is the lack of computing speed.: "The main limitation of G-Aligner is the lack of computing speed."
- [discussion] G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time.: "G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time."
- [readme] Feature extraction results in csv format, containing m/z, RT and area columns.: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
