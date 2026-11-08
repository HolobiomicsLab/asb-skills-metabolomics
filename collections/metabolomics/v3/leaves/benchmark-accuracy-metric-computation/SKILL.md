---
name: benchmark-accuracy-metric-computation
description: Use when evaluating feature alignment quality on a benchmark dataset
  that has manual annotations of library analytes, to produce accuracy/F1 metrics
  comparable to published alignment benchmarks.
when_to_use_negative:
- The dataset has no manually annotated library analytes, so there is no ground truth
  against which to count accurate alignments
- The goal is running or improving the alignment itself (G-Aligner's multipartite
  graph / multidimensional assignment solving) rather than scoring its output
- Input is a quantification or detection benchmark where m/z, RT and area of library
  analytes are not the evaluation target
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Produces the alignment results being evaluated; provides data/evaluate_metapro_galigner.py
    evaluation script and benchmark data
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Comparison alignment software whose extracted feature lists are scored with
    the same metrics
- name: OpenMS
  role: Comparison alignment software whose extracted feature lists are scored with
    the same metrics
- name: XCMS
  role: Comparison alignment software whose extracted feature lists are scored with
    the same metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:ca128c77233de9cc0d7f0c937c1499567823f25b34471c81ac2487865627a7a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# benchmark-accuracy-metric-computation

## Summary

Computes feature-level and analyte-level alignment accuracy (plus F1 score) by matching aligned consensus features against manually annotated library analytes (m/z, RT, area), as done to benchmark G-Aligner against MZmine2, OpenMS, and XCMS on the MTBLS562 dataset.

## When to use

Use when evaluating feature alignment quality on a benchmark dataset that has manual annotations of library analytes, to produce accuracy/F1 metrics comparable to published alignment benchmarks.

## When NOT to use

- The dataset has no manually annotated library analytes, so there is no ground truth against which to count accurate alignments
- The goal is running or improving the alignment itself (G-Aligner's multipartite graph / multidimensional assignment solving) rather than scoring its output
- Input is a quantification or detection benchmark where m/z, RT and area of library analytes are not the evaluation target

## Inputs

- Manually annotated feature dataset (m/z, RT, area of library analytes) from Zenodo 10.5281/zenodo.8313034
- Extracted feature lists per run (csv with m/z, RT and area columns) from data/MTBLS562 of the G-Aligner repo
- Feature alignment results (consensus features) written to the experiment folder

## Outputs

- alignment_accuracy_mtbls562.csv with feature-level accuracy, analyte-level accuracy, F1 score, and computational time
- Counts of accurately aligned features and accurately aligned analytes

## How to apply

Download the manually annotated feature dataset (m/z, RT, area of library analytes) from Zenodo record 10.5281/zenodo.8313034 and the extracted feature lists (e.g. data/MTBLS562) from the G-Aligner repository. Configure the evaluation script data/evaluate_metapro_galigner.py with the parameter set for the dataset (Table S3 for MTBLS562) and point it at the annotated feature lists and the alignment results saved in the 'experiment' folder. Run the alignment, then match aligned features against the manual annotation to count accurately aligned features and accurately aligned analytes. Compute feature-level accuracy, analyte-level accuracy, F1 score, and computational time, saving results as alignment_accuracy_mtbls562.csv for comparison against the paper's Table 3.

## Related tools

- **G-Aligner** (Produces the alignment results being evaluated; provides data/evaluate_metapro_galigner.py evaluation script and benchmark data) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Comparison alignment software whose extracted feature lists are scored with the same metrics)
- **OpenMS** (Comparison alignment software whose extracted feature lists are scored with the same metrics)
- **XCMS** (Comparison alignment software whose extracted feature lists are scored with the same metrics)

## Examples

```
python data/evaluate_metapro_galigner.py  # after pointing it at data/MTBLS562 annotated features; results saved to experiment folder
```

## Evaluation signals

- Computed feature-level and analyte-level accuracy reproduce the published G-Aligner row for MTBLS562 in Table 3 (F1 = 1.000, feature accuracy = 0.999, analyte accuracy = 0.981)
- G-Aligner metrics are the best among OpenMS, MZmine2 and XCMS on the same annotated features
- Per-run feature lists parse as csv with m/z, RT and area columns as required by the evaluation scripts
- Counts of accurately aligned features and analytes are non-decreasing sanity-bounded (<= total annotated analytes) and match manual-annotation matching output
- Metrics CSV is written (alignment_accuracy_mtbls562.csv) with entries for all compared methods

## Limitations

- Ground truth depends on the quality of the manually annotated feature dataset; annotations were only produced for three public benchmark datasets (MTBLS562, QE_HF, TripleTOF_6600)
- Optimized evaluation parameters are reported only in supplementary tables (S1-S3), not in the main text
- G-Aligner's main limitation is computing speed; the multidimensional assignment problem is NP-Hard and requires substantial computing time, so timing metrics may vary by hardware

## Evidence

- [methods] Manually annotated feature lists for evaluation: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [readme] README benchmark scripts and dataset layout: "Change the parameters in data/evaluate_metapro_galigner.py

```python data/evaluate_metapro_galigner.py```

Feature alignment results are saved in ```experiment``` folder."
- [readme] README supported input format: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [abstract] Published accuracy finding for MTBLS562: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
- [methods] Benchmark data availability: "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
