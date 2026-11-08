---
name: lc-ms-feature-alignment-algorithm-implementation
description: Use when multiple LC-MS run feature tables (CSV with m/z, RT, area columns)
  must be aligned into consensus features beyond naive nearest-neighbor matching,
  i.e. all potential cross-run feature correspondences must be considered jointly.
when_to_use_negative:
- Input is raw mzML/MS data rather than already-extracted feature lists — G-Aligner
  only supports feature extraction results in CSV format with m/z, RT and area columns
- A fast, low-latency alignment is required — the multidimensional assignment problem
  is NP-Hard and G-Aligner's main limitation is lack of computing speed
- A pairwise nearest-neighbor alignment against a single reference run is sufficient,
  as in MZmine2's approach
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: 'Core graph-based feature alignment method being implemented: multipartite
    graph construction, graph splitting, and unbalanced multidimensional assignment
    solving'
  repo: https://github.com/CSi-Studio/G-Aligner
- name: OBI-Warp
  role: Used together with RANSAC for stable coarse retention time registration between
    runs prior to graph building
- name: MZmine2
  role: Source of extracted feature lists used as benchmark input for evaluating the
    implemented alignment
- name: OpenMS
  role: Source of extracted feature lists used as benchmark input for evaluating the
    implemented alignment
- name: XCMS
  role: Source of extracted feature lists used as benchmark input for evaluating the
    implemented alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:159be1b11f26f915e4e19f4ab57ca14fec041950e5c7b0aaca2da258db75f181
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lc-ms-feature-alignment-algorithm-implementation

## Summary

Implement the G-Aligner three-step feature alignment pipeline for untargeted LC-MS metabolomics: after coarse RT registration with RANSAC and OBI-Warp, build a multipartite relation graph of features (nodes) and potential correspondences (edges), split it into connected components, and solve multi-run matching as an unbalanced multidimensional assignment problem with three combinatorial optimization algorithms.

## When to use

Use when multiple LC-MS run feature tables (CSV with m/z, RT, area columns) must be aligned into consensus features beyond naive nearest-neighbor matching, i.e. all potential cross-run feature correspondences must be considered jointly.

## When NOT to use

- Input is raw mzML/MS data rather than already-extracted feature lists — G-Aligner only supports feature extraction results in CSV format with m/z, RT and area columns
- A fast, low-latency alignment is required — the multidimensional assignment problem is NP-Hard and G-Aligner's main limitation is lack of computing speed
- A pairwise nearest-neighbor alignment against a single reference run is sufficient, as in MZmine2's approach

## Inputs

- CSV feature tables per LC-MS run containing m/z, RT and area columns (e.g. manual annotations from Zenodo 10.5281/zenodo.8313034 or *_results_* folders of mzmine2, openms, xcms, metapro)
- User-specified m/z and retention time tolerance parameters
- Datasets MTBLS562, QE_HF and TripleTOF_6600

## Outputs

- Consensus feature groups (alignment results) written to the experiment folder
- Grouped features across runs without ambiguous matched groups

## How to apply

First perform coarse retention time registration between runs using RANSAC and OBI-Warp, then for each feature search neighborhoods within user-specified m/z and RT tolerances to build an undirected relation graph over features of all runs, where features are nodes and potential correspondences are edges. Split the multipartite graph into connected components and compute edge weights per Eqs 1-2 of the paper. Formulate the multi-run feature matching as an unbalanced multidimensional assignment problem, solved with three combinatorial optimization algorithms (Eqs 3-9, cost Eqs 10-15). Execute the module on each benchmark feature list via the driver script data/evaluate_metapro_galigner.py (adjusting parameters as needed) and write alignment results to the experiment folder. Rationale: retention time alignment only corrects drifting trends, so modeling all potential correspondences as a graph and solving globally yields higher alignment accuracy than nearest-neighbor matching used by MZmine2, OpenMS and XCMS.

## Related tools

- **G-Aligner** (Core graph-based feature alignment method being implemented: multipartite graph construction, graph splitting, and unbalanced multidimensional assignment solving) — https://github.com/CSi-Studio/G-Aligner
- **OBI-Warp** (Used together with RANSAC for stable coarse retention time registration between runs prior to graph building)
- **MZmine2** (Source of extracted feature lists used as benchmark input for evaluating the implemented alignment)
- **OpenMS** (Source of extracted feature lists used as benchmark input for evaluating the implemented alignment)
- **XCMS** (Source of extracted feature lists used as benchmark input for evaluating the implemented alignment)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Pipeline runs end-to-end on all three benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600)
- Resulting consensus feature groups contain no ambiguous matched groups
- Accurately aligned features/analytes increase relative to MZmine2, OpenMS and XCMS baselines (reported up to 9.8% and 26.6% improvement)
- Output CSV alignment results are written to the experiment folder with consensus feature group structure
- Graph construction reflects all potential correspondences found within the specified m/z and RT tolerance neighborhoods, not only nearest neighbors

## Limitations

- The main limitation of G-Aligner is the lack of computing speed; the multidimensional assignment problem is NP-Hard and inevitably requires a lot of computing time
- Although two acceleration methods keep running time within an acceptable range, there is still a gap between painless application; GPU acceleration is not yet implemented
- Optimized parameters are only reported in supplementary tables (S1-S3), not in the main text

## Evidence

- [other] After coarse RT registration, G-Aligner searches neighborhoods for each feature according to user-specified m/z and retention time tolerances and builds an undirected relation graph containing features of all runs, in which features are nodes and potential correspondences are edges.: [evidence span withheld — anchor status: not_found]
- [intro] In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs.: "G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [abstract] considers the multi-run feature matching problem an unbalanced multidimensional assignment problem: "considers the multi-run feature matching problem an unbalanced multidimensional assignment problem"
- [readme] Change the parameters in data/evaluate_metapro_galigner.py

```python data/evaluate_metapro_galigner.py```

Feature alignment results are saved in ```experiment``` folder.: "Change the parameters in data/evaluate_metapro_galigner.py ... Feature alignment results are saved in ```experiment``` folder."
- [discussion] The main limitation of G-Aligner is the lack of computing speed.: "The main limitation of G-Aligner is the lack of computing speed."
- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
