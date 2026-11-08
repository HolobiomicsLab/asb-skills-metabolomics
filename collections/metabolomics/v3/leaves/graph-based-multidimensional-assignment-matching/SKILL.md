---
name: graph-based-multidimensional-assignment-matching
description: Use when aligning detected LC–MS features (m/z, RT, area) across multiple
  runs with nonlinear retention time drift, and accuracy of consensus feature/analyte
  matching matters more than speed.
when_to_use_negative:
- When computing speed is the priority — G-Aligner's main limitation is lack of computing
  speed due to the NP-Hard multidimensional assignment formulation
- When input is raw MS data rather than an extracted CSV feature table with m/z, RT
  and area columns
- When a quick RT-trend correction suffices, since retention time alignment alone
  describes drifting trends without the feature-matching step
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: G-Aligner
  role: Implements the multipartite-graph formulation and three combinatorial optimization
    solvers for unbalanced multidimensional assignment feature matching
  repo: https://github.com/CSi-Studio/G-Aligner
- name: OpenMS
  role: Baseline clustering-based feature alignment method to compare against
- name: MZmine2
  role: Baseline pairwise nearest matching alignment method (with RANSAC RT alignment)
    to compare against
- name: XCMS
  role: Baseline clustering-based feature alignment method to compare against
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:a3cc757f1ab8ee109fdb8385232574532c621a5f628b2123c69bca7158374280
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# graph-based-multidimensional-assignment-matching

## Summary

G-Aligner's core algorithm: represent features of multiple LC–MS runs and their potential correspondences as a multipartite graph, then solve multi-run feature matching as an unbalanced multidimensional assignment problem using three combinatorial optimization solvers, achieving higher alignment accuracy than nearest-neighbor matching in OpenMS, MZmine2 and XCMS.

## When to use

Use when aligning detected LC–MS features (m/z, RT, area) across multiple runs with nonlinear retention time drift, and accuracy of consensus feature/analyte matching matters more than speed.

## When NOT to use

- When computing speed is the priority — G-Aligner's main limitation is lack of computing speed due to the NP-Hard multidimensional assignment formulation
- When input is raw MS data rather than an extracted CSV feature table with m/z, RT and area columns
- When a quick RT-trend correction suffices, since retention time alignment alone describes drifting trends without the feature-matching step

## Inputs

- CSV-format feature lists per run (m/z, RT, area columns) extracted by MZmine2, OpenMS, XCMS or manually annotated
- mzML files of benchmark metabolomics datasets (e.g., QE_HF, TripleTOF_6600, MTBLS562) from Zenodo deposit 10.5281/zenodo.8313034
- Optimized parameter settings from the paper's supplementary tables (S1–S3)

## Outputs

- Aligned consensus features saved in the 'experiment' folder
- Evaluation metrics: TP, FP, TN, FN, precision, recall, F1 score, feature alignment accuracy (F_ACC), analyte alignment accuracy (A_ACC)

## How to apply

First perform retention time alignment (coarse registration) between runs with RANSAC and OBI-Warp, since naive nearest-neighbor matching after RT alignment cannot accurately correct each feature's drift. Then build a multipartite graph whose nodes are features from each run and whose edges are potential correspondences, and format the multi-run matching as an unbalanced multidimensional assignment problem (NP-Hard). Run G-Aligner's three combinatorial optimization solvers and pick the solver with the highest alignment accuracy on the benchmark at hand, using optimized parameters from the paper's supplementary tables (e.g., Table S2 for QE_HF). Feature extraction input must be CSV format containing m/z, RT and area columns; aligned consensus features are saved in the 'experiment' folder. Evaluate against manually annotated ground truth using TP/FP/TN/FN, precision, recall, F1, feature accuracy (F_ACC) and analyte accuracy (A_ACC).

## Related tools

- **G-Aligner** (Implements the multipartite-graph formulation and three combinatorial optimization solvers for unbalanced multidimensional assignment feature matching) — https://github.com/CSi-Studio/G-Aligner
- **OpenMS** (Baseline clustering-based feature alignment method to compare against)
- **MZmine2** (Baseline pairwise nearest matching alignment method (with RANSAC RT alignment) to compare against)
- **XCMS** (Baseline clustering-based feature alignment method to compare against)

## Examples

```
cd ROOT_PATH; python data/evaluate_metapro_galigner.py  # parameters configured per supplementary Table S2; results saved in experiment folder
```

## Evaluation signals

- On the QE HF dataset's manually annotated features, expected G-Aligner performance is F1 = 0.998, feature accuracy = 0.997 and analyte accuracy = 0.989 (paper Table 2)
- Reproduced F_ACC/A_ACC should be consistent with the paper's reported gains of up to 9.8% in accurately aligned features and 26.6% in accurately aligned analytes over OpenMS, MZmine2 and XCMS
- Aligned consensus feature results exist in the 'experiment' folder after running the alignment script
- Each of the three combinatorial optimization solvers should be run and the one with the highest accuracy identified
- Output evaluation results table contains all required metric columns (TP, FP, TN, FN, precision, recall, F1, F_ACC, A_ACC) populated

## Limitations

- The main limitation of G-Aligner is lack of computing speed; the multidimensional assignment problem is NP-Hard and requires a lot of computing time
- Two acceleration methods are provided but a gap remains before painless application; GPU acceleration is not yet implemented
- Optimized parameters are reported only in supplementary tables (S1–S3), not in the main text, and must be tuned per dataset

## Evidence

- [readme] G-Aligner treats features and potential correspondences as nodes and edges of a multipartite graph: "G-Aligner treats features and potential correspondences as nodes and edges of a multipartite graph"
- [readme] converts the feature matching problem as a multidimensional assignment problem (MAP): "converts the feature matching problem as a multidimensional assignment problem (MAP)"
- [intro] In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [discussion] G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard: "G-Aligner modeled the feature matching problem to a multidimensional assignment problem, which was NP-Hard and inevitably requires a lot of computing time."
- [readme] Feature extraction rsults in csv format, containing m/z, RT and area columns: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
