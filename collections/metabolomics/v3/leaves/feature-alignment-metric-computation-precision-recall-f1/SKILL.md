---
name: feature-alignment-metric-computation-precision-recall-f1
description: Use when a feature alignment result (e.g. from G-Aligner, MZmine2, OpenMS,
  XCMS or local bipartite matching) must be quantitatively scored against manually
  annotated library-analyte features on benchmark datasets such as TripleTOF 6600,
  QE HF or MTBLS562.
when_to_use_negative:
- The goal is peak detection or feature quantification accuracy rather than feature
  alignment accuracy - the same Zenodo deposit supports detection and quantification
  evaluations, but the metric set here (precision/recall/F1, F_ACC, A_ACC) is specific
  to alignment.
- No manually annotated reference feature list exists for the dataset; precision/recall/F1
  and F_ACC/A_ACC are only well-defined against such a ground truth.
- The task is to evaluate raw MS data acquisition quality (e.g. instrument calibration)
  rather than the correspondence of consensus features across LC-MS runs.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: G-Aligner
  role: Provides the feature alignment results that are being scored, plus the evaluation
    scripts (data/evaluate_metapro_galigner.py etc.) that compute TP/FP/TN/FN, precision,
    recall, F1, F_ACC and A_ACC against the manually annotated reference.
  repo: https://github.com/CSi-Studio/G-Aligner
- name: pandas
  role: Used to load the csv alignment results and the annotated feature list and
    to compile the per-method metric comparison table.
- name: numpy
  role: Supports the metric computation over m/z and RT matching of aligned features
    to library analytes.
- name: Python
  role: Runtime environment for the evaluation scripts and dependency installation
    described in the README setup.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:4da36b66e82aa7278d1ca04b288b3ac24f01959ecf1908880f2b3763bc8daf4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# feature-alignment-metric-computation-precision-recall-f1

## Summary

Evaluate LC-MS feature alignment quality by computing TP/FP/TN/FN counts and precision, recall, F1, plus G-Aligner-specific accuracy metrics (percentage of accurately aligned features F_ACC and analytes A_ACC) against a manually annotated reference feature list.

## When to use

Use when a feature alignment result (e.g. from G-Aligner, MZmine2, OpenMS, XCMS or local bipartite matching) must be quantitatively scored against manually annotated library-analyte features on benchmark datasets such as TripleTOF 6600, QE HF or MTBLS562.

## When NOT to use

- The goal is peak detection or feature quantification accuracy rather than feature alignment accuracy - the same Zenodo deposit supports detection and quantification evaluations, but the metric set here (precision/recall/F1, F_ACC, A_ACC) is specific to alignment.
- No manually annotated reference feature list exists for the dataset; precision/recall/F1 and F_ACC/A_ACC are only well-defined against such a ground truth.
- The task is to evaluate raw MS data acquisition quality (e.g. instrument calibration) rather than the correspondence of consensus features across LC-MS runs.

## Inputs

- Feature alignment result files (csv with m/z, RT, area columns) saved in the experiment folder for each solver or software configuration
- Manually annotated feature dataset for the benchmark (m/z, RT, area information of library analytes) from Zenodo deposit 10.5281/zenodo.8313034
- Repository evaluation scripts (data/evaluate_metapro_galigner.py, evaluate_metapro_openms.py, evaluate_metapro_xcms.Rmd, evaluate_metapro_mzmine2.java, evaluate_mzmine2_galigner.py, evaluate_openms_galigner.py, evaluate_xcms_galigner.py)

## Outputs

- TP/FP/TN/FN counts per alignment run
- Precision, recall and F1 values per method or solver
- F_ACC: percentage of accurately aligned features
- A_ACC: percentage of analytes with all replicates accurately aligned
- Comparison summary table mirroring Table 1 (TripleTOF 6600), Table 2 (QE HF) and Table 3 (MTBLS562)

## How to apply

Take the alignment result saved in the experiment folder and the manually annotated feature dataset (containing m/z, RT and area information of library analytes for each benchmark dataset). Match aligned consensus features to the annotated analytes to derive TP/FP/TN/FN counts, then compute precision, recall and F1; additionally compute F_ACC, the percentage of accurately aligned features, and A_ACC, the percentage of analytes with all replicates accurately aligned. Run the evaluation per solver configuration (Gurobi, Greedy, VLSNS_MSR, VLSNS_MSG) or per software (MZmine2, OpenMS, XCMS) using the corresponding data/evaluate_metapro_galigner.py, evaluate_metapro_openms.py, evaluate_metapro_xcms.Rmd or evaluate_mzmine2_galigner.py scripts from the G-Aligner repository, and compile the metrics into a comparison table mirroring Table 1-3 of the paper. Use the optimized parameters from supplementary Table S1 (TripleTOF 6600), S2 (QE HF) or S3 (MTBLS562) for the G-Aligner side of the comparison.

## Related tools

- **G-Aligner** (Provides the feature alignment results that are being scored, plus the evaluation scripts (data/evaluate_metapro_galigner.py etc.) that compute TP/FP/TN/FN, precision, recall, F1, F_ACC and A_ACC against the manually annotated reference.) — https://github.com/CSi-Studio/G-Aligner
- **pandas** (Used to load the csv alignment results and the annotated feature list and to compile the per-method metric comparison table.)
- **numpy** (Supports the metric computation over m/z and RT matching of aligned features to library analytes.)
- **Python** (Runtime environment for the evaluation scripts and dependency installation described in the README setup.)

## Examples

```
cd ROOT_PATH && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- Computed precision, recall, F1, F_ACC and A_ACC values match those reported in Table 1 for the TripleTOF 6600 dataset (and Tables 2-3 for QE HF and MTBLS562) within rounding error.
- G-Aligner-based solver configurations (Gurobi, Greedy, VLSNS_MSR, VLSNS_MSG) show the highest values among all compared methods, consistent with the reported up to 9.8% and 26.6% increase in accurately aligned features and analytes.
- Every evaluated alignment run yields non-negative TP/FP/TN/FN counts whose TP + FN total equals the number of annotated library analytes matched to that run's feature set.
- The final summary table has one row per evaluated method (G-Aligner solvers, MZmine2, OpenMS, XCMS, local bipartite matching) with all five metrics populated - no missing cells.
- All metrics lie within the expected 0-100% range, and A_ACC is never higher than F_ACC for the same configuration (since A_ACC requires all replicates of an analyte to be accurate).

## Limitations

- The manually annotated feature dataset used as ground truth covers three public benchmark datasets (TripleTOF 6600, QE HF, MTBLS562); metrics cannot be computed this way on datasets lacking such annotation.
- Optimized parameters are reported only in supplementary tables S1-S3, not in the main text, so reproducing the G-Aligner side requires the supplementary material.
- The evaluation reflects alignment accuracy only; G-Aligner's main limitation of computing speed (the MAP is NP-Hard) is not captured by these accuracy metrics.

## Evidence

- [abstract] G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [abstract] Benchmark datasets, manual annotation results, evaluation methods and results are available at https://doi.org/10.5281/zenodo.8313034: "Benchmark datasets, manual annotation results, evaluation methods and results are available at https://doi.org/10.5281/zenodo.8313034"
- [methods] We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [methods] Feature alignment results are saved in ```experiment``` folder.: "Feature alignment results are saved in ```experiment``` folder."
- [readme] The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034): "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
